---
layout:     post
title:      "解决线上问题：static共享变量引起的并发问题"
subtitle:   "static-class-variable-cause-concurrent-problem"
date:       2018-12-20 12:00:00
author:     "沈歌"
header-img: "img/home-bg-o.jpg"
tags:
        - real-problem
        - java
---


代码简要记录如下：

LogIntegerceptor: 用于在请求返回前做日志记录

``` Java
public class LogInterceptor implements HandlerInterceptor {

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
            throws Exception {
        List<OperationLogDetail> details = LogObjectHolder.instance().getDetails();

        details.forEach(t -> {
                    t.setLogId(log.getId());
                    operationLogDetailDao.insert(t);
                }
        );

    }

}

```

LogObjectHolder: 对象作用域为REQUEST，即每个请求都创建一个新的LogObjectHolder实例。

``` Java
@Component
@Scope(scopeName = WebApplicationContext.SCOPE_REQUEST)
public class LogObjectHolder implements Serializable {

    private static List<OperationLogDetail> details;

    public LogObjectHolder(){
        details = Lists.newArrayList();
    }

    public void mod(Integer id, String fieldName){
        try {
            OperationLogDetail detail = new OperationLogDetail();
            detail.setOpType("mod");
            detail.setContentId(id);
            detail.setDescription(fieldName);
            details.add(detail);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static List<OperationLogDetail> getDetails() {
        return details;
    }

    public static LogObjectHolder instance() {
        LogObjectHolder bean = SpringContextHolder.getBean(LogObjectHolder.class);
        return bean;
    }
}
```

代码逻辑如下：

每个请求结束后，通过调用LogObjectHolder的mod方法，将log放到details中，然后请求返回前会调用LogIntegerceptor的postHandle，将操作日志记录到数据库中。

但是以上代码在线上，偶现报错，报错如下：

```
### Error updating database.  Cause: com.mysql.jdbc.exceptions.jdbc4.MySQLIntegrityConstraintViolationException: Duplicate entry '498843' for key 'PRIMARY'
### The error may involve defaultParameterMap
### The error occurred while setting parameters
### SQL: INSERT INTO operation_log_detail (                    id ,     log_id ,     class_name ,     op_type ,     content_id ,     description ,     content                ) VALUES (                    ? ,     ? ,     ? ,     ? ,     ? ,     ? ,     ?                )

```

是一个插入数据时key重复的报错。那么一定有其他请求插入了这条数据，查找相关的log，发现是一个并发的另一个http请求的逻辑插入进去的。


那么为什么是另一个请求插入进去呢？难道是spring的scope有问题？？？

然后看了一下spring的request scope的实现，是通过ThreadLocal实现的，但是查看日志发现，插入数据的是另一个线程操作的，那么可以否定这种可能了。

然后再研究一下老日志，发现有大量这样的报错：


```
java.util.ConcurrentModificationException: null
    at java.util.ArrayList.forEach(ArrayList.java:1252) ~[?:1.8.0_144]
```

因为ArrayList不是线程安全的，forEach的过程中，list的size变化，就会抛出这个异常。源码如下：

```
    @Override
    public void forEach(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        final int expectedModCount = modCount;
        @SuppressWarnings("unchecked")
        final E[] elementData = (E[]) this.elementData;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            action.accept(elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```

所以，一定是有并发请求的。

然后，仔细看了一下实现代码，终于找到了问题。

```
    private static List<OperationLogDetail> details;
```

这里，details是静态类变量，我们知道，静态类变量jdk1.8之前存在方法区，jdk1.8之后存在元空间（Metaspace），全局只有一份数据。

那么threadA put， threadB insert，在并发情况下是完全有可能发生的。threadB已经insert过了，id已经生成并放到了OperationLogDetail对象中，然后threadA再insert，就会报Duplicate entry的错了。
