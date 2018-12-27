---
layout:     post
title:      "译:Java局部变量类型推断（Var类型）的26条细则"
subtitle:   "static-class-variable-cause-concurrent-problem"
date:       2018-12-20 12:00:00
author:     "沈歌"
header-img: "img/home-bg-o.jpg"
tags:
        - var
        - java
---




原文链接：https://dzone.com/articles/var-work-in-progress

作者：Anghel Leonard

译者：沈歌

![image](https://shenpengyan.github.io/img/in-post/var-work-in-progress/using-var-in-java-10.png)

Java局部变量类型推断（LVTI），简称`var`类型（标识符`var`不是一个关键字，是一个预留类型名），Java 10中通过[JEP 286: Local-Variable Type Inference](http://openjdk.java.net/jeps/286) 添加进来。作为100%编译特征，它不会影响字节码，运行时或者性能。在编译时，编译器会检查赋值语句右侧代码，从而推断出具体类型。它查看声明的右侧，如果这是一个初始化语句，它会用那个类型取代`var`。另外，它非常有助于减少冗余代码和样板代码。它还只在旨在编写代码时所涉及的仪式。例如，使用`var evenAndOdd=...` 代替`Map<Boolean, List<Integer>> evenAndOdd...` 非常方便。根据用例，它有一个代码可读性的权衡，会在下面第一条中提到。

此外，这里有26条细则，覆盖了`var`类型的用例，包括它的限制。

## 1. 争取起有意义的局部变量名

通常我们在起全局变量名的时候会注意这一点，但是选择局部变量名的时候不太注意。尤其是当方法很短，方法名和实现都不错的时候，我们趋向于简化我们的变量名。但是当我们使用`var`替代显式类型的时候，具体的类型是通过编译器推断出来的。所以，对于人来说阅读或者理解代码非常困难。在这一点上`var`削弱了代码可读性。这种事情之所以会发生，是因为大多数情况下，我们会把变量类型当成是第一信息，而把变量名当成第二信息。但是使用`var`的时候，恰恰相反。

### 示例1

即使到这里，一些朋友仍然坚持局部变量名短点好。我们看一下：

``` Java

// HAVING
public boolean callDocumentationTask() {
    DocumentationTool dtl = ToolProvider.getSystemDocumentationTool();
    DocumentationTask dtt = dtl.getTask(...);
    return dtt.call();
}

```

我们换成`var`时，避免：

``` Java
// AVOID
public boolean callDocumentationTask() {
    var dtl = ToolProvider.getSystemDocumentationTool();
    var dtt = dtl.getTask(...);
    return dtt.call();
}
```

更好：

``` Java
// PREFER
public boolean callDocumentationTask() {
    var documentationTool = ToolProvider.getSystemDocumentationTool();
    var documentationTask = documentationTool.getTask(...);
  return documentationTask.call();
}
```

### 示例2：


避免：
``` Java
// AVOID
public List<Product> fetchProducts(long userId) {
    var u = userRepository.findById(userId);
    var p = u.getCart();
    return p;
}
```

更好：

``` Java
// PREFER
public List<Product> fetchProducts(long userId) {
    var user = userRepository.findById(userId);
    var productList = user.getCart();
    return productList;
}
```

### 示例3：

争取为局部变量起有意义的名字并不意味着要掉入过度命名的坑，避免在短方法中使用单一类型的数据流：

``` Java

// AVOID
var byteArrayOutputStream = new ByteArrayOutputStream();
```

用如下代码代替更加清晰：

``` Java
// PREFER
var outputStream = new ByteArrayOutputStream();
// or
var outputStreamOfFoo = new ByteArrayOutputStream();
```

另外，你知道吗，Java内部使用了一个类名字叫：`InternalFrameInternalFrameTitlePaneInternalFrameTitlePaneMaximizeButtonWindowNotFocusedState` 

额。。。命名这个类型的变量是个挑战。

## 2. 使用*数据类型标志*来帮助var去推断出预期的基本数据类型(int, long, float, double)

如果在基本数据类型中不使用有效的*数据类型标志*，我们可能会发现预期的类型和推测出的类型不一致。这是由于`var`的隐式类型转换导致的。

例如，下面两行代码的表现是符合预期的，首先，我们声明一个`boolean` 和一个`char`使用显式类型：

``` Java
boolean flag = true; // 这是一个boolean类型
char a = 'a';        // 这是一个char类型
```

现在，我们使用`var`代替显式基本类型：

``` Java
var flag = true; // 被推断为boolean类型
var a = 'a';     // 被推断为char类型
```

到目前为止，一切都很完美。接下来，我们看一下相同逻辑下的`int`, `long`, `double` 和 `float`:

``` Java
int intNumber = 20;       // 这是int类型
long longNumber = 20;     // 这是long类型
float floatNumber = 20;   // 这是float类型, 20.0
double doubleNumber = 20; // 这是double类型, 20.0
```

以上代码是很常见而且清晰的，现在我们使用`var`:

避免：

``` Java
// AVOID
var intNumber = 20;    // 推断为int
var longNumber = 20;   // 推断为int
var floatNumber = 20;  // 推断为int
var doubleNumber = 20; // 推断为int
```

四个变量都被推断成了`int`。为了修正这个行为，我们需要依赖Java中的数据类型标志。

更好实现:
``` Java
// PREFER
var intNumber = 20;     // 推断为int
var longNumber = 20L;   // 推断为long
var floatNumber = 20F;  // 推断为float, 20.0
var doubleNumber = 20D; // 推断为double, 20.0
```

但是如果我们使用小数声明一个数字，会发生什么呢？当你认为你的数字是一个`float`的时候，避免这样做：

``` Java
// 避免，如果这是一个float
var floatNumber = 20.5; // 推断为double
```

你应该用对应的数据类型标志来避免这样的问题：

```
// 更好, 如果这是一个float
var floatNumber = 20.5F; // 推断为float
```

## 3. 在某些情况下，*Var*和*隐式类型转换*可以维持可维护性

在某些情况下，*Var*和*隐式类型转换*可以维持可维护性。例如，假设我们的代码包含两个方法：第一个方法接收一个包含不同条目的购物卡，比较市场中不同的价格，计算出最好的价格，并汇总返回`float`类型的总价。另一个方法简单的把这个`float`价格从卡中扣除。

首先，我们看一下计算最好价格的方法：

``` Java
public float computeBestPrice(String[] items) {
   ...
   float price = ...;
   return price;
}
```

然后，我们看一下扣款的方法：

``` Java
public boolean debitCard(float amount, ...) {
    ...
}
```

现在，我们把这两个方法汇总，提供一个服务方法。顾客选择要买的商品，计算最优价格，然后扣款：

``` Java
// AVOID
public void purchaseCart(long customerId) {
    ...
    float price = computeBestPrice(...);
    debitCard(price, ...);
}
```

一段时间后，公司想要去除价格中的小数部分作为打折策略，使用`int`代替了`float`, 我们需要修改代码。

``` Java
public int computeBestPrice(String[] items) {
   ...
   float realprice = ...;
   ...
   int price = (int) realprice;
   return price;
}
public boolean debitCard(int amount, ...) {
    ...
}
```

问题在于我们使用了显示类型`float`，这样的更改不能被兼容。代码会报编译时错误。但是如果我们预判到这种情况，使用`var`代替`float`, 我们的代码会因为隐式类型转换而变得没有兼容性问题。

``` Java
// PREFER
public void purchaseCart(long customerId) {
    ...
    var price = computeBestPrice(...);
    debitCard(price, ...);
}
```

## 4. 当数据类型标志解决不了问题的时候，依赖显式向下转换或者避免*var*

一些Java基础数据类型不支持数据类型标志。例如`byte`和`short`。使用显式基础数据类型时没有任何问题。使用`var`代替的时候：

``` Java
// 这样更好，而不是使用var
byte byteNumber = 45;     // 这是byte类型
short shortNumber = 4533; // 这是short类型
```

为什么在这种情况下显式类型比`var`好呢？我们切换到`var`.注意示例中都会被推断为`int`, 而不是我们预期的类型。

避免使用以下代码：

``` Java
// AVOID
var byteNumber = 45;    // 推断为int
var shortNumber = 4533; // 推断为int
```

这里没有基础数据类型帮助我们，因此我们需要依赖显示强制类型转换。从个人角度来讲，我会避免这么用，因为没啥好处，但是可以这么用。

如果你真的想用`var`，这么用:

``` Java
// 如果你真的想用var，这么写
var byteNumber = (byte) 45;     // 推断为byte
var shortNumber = (short) 4533; // 推断为short
```

## 5. 如果变量名没有对人来说足够的类型信息，避免使用*var*

使用`var`有助于提供更加简练的代码。例如, 在使用构造方法时（这是使用局部变量的常见示例），我们可以简单地避免重复类名的必要性，从而消除冗余。

避免：

```
// AVOID
MemoryCacheImageInputStream inputStream = new MemoryCacheImageInputStream(...);
```

更好：

```
// PREFER
var inputStream = new MemoryCacheImageInputStream(...);
```

在下面的结构中，`var`也是一个简化代码而不丢失信息的好方法。

避免：

``` Java
// AVOID
JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
StandardJavaFileManager fm = compiler.getStandardFileManager(...);
```

更好：

``` Java
// PREFER
var compiler = ToolProvider.getSystemJavaCompiler();
var fileManager = compiler.getStandardFileManager(...);
```

为什么这样基于`var`的例子我们感觉比较舒服呢？因为需要的信息已经在变量名中了。但是如果使用`var` 加上变量名，还是会丢失信息，那么最好避免使用`var`。

避免:

``` Java
// AVOID
public File fetchCartContent() {
    return new File(...);
}
// As a human, is hard to infer the "cart" type without 
// inspecting the fetchCartContent() method
var cart = fetchCartContent();
```

使用以下代码代替：

``` Java
// PREFER
public File fetchCartContent() {
    return new File(...);
}
File cart = fetchCartContent();
```

思考一个基于`java.nio.channels.Selector`的例子。这个类有一个静态方法叫做`open()`，返回一个新的`Selector`实例并且执行open动作。但是`Selector.open()`很容易被认为返回一个`boolean`标识打开当前选择器是否成功，或者返回`void`。使用`var`导致丢失信息会引发这样的困扰。

## 6. *var*类型确保编译时安全

`var`类型是编译时安全的。这意味着如果我们试图实现一个错的赋值，会导致编译时报错。例如，以下代码编译不会通过。

``` Java
// 编译通不过
var items = 10;
items = "10 items"; // 不兼容类型: String不能转为int
```

以下代码编译会通过

``` Java
var items = 10;
items = 20;
```

这个代码也会编译通过：

``` Java
var items = "10";
items = "10 items" ;
```

所以，一旦编译器已经推断出了`var`对应的类型，我们只能赋值对应类型的值给它。

## 7. *var* 不能被用于将真实类型的实例赋值给接口类型变量。

在Java中，我们使用“面向接口编程”的技术。

例如，我们创建一个`ArrayList`的实例，如下(绑定代码到抽象)：

``` Java
List<String> products = new ArrayList<>();
```

我们避免这样的事情(绑定代码到实现):

``` Java
ArrayList<String> products = new ArrayList<>();
```

所以，通过第一个例子创建一个`ArrayList`实例更好，但是我们也需要声明一个`List`类型的变量。因为`List`是一个接口，我们可以很容易的切换到`List`的其他实现类，而无需额外的修改。

这就是“面向接口编程”，但是`var`不能这么用。这意味着当我们使用`var`时，推断出的类型是实现类的类型。例如，下面这行代码，推测出的类型是`ArrayList<String>`:

``` Java
var productList = new ArrayList<String>(); // 推断为ArrayList<String>
```

以下几个论点支持这一行为：

- 首先，`var`是局部变量，大多数情况下，“面向接口编程”在方法参数和返回类型的时候更有用。
- 局部变量的作用域比较小，切换实现引起的发现和修复成本比较低。
- var将其右侧的代码视为用于对端实际类型的初始化程序，如果将来修改初始化程序，则推断类型会改变，从而导致后续依赖此变量的代码产生问题。

## 8. 意外推断类型的可能性

如果不存在推断类型所需的信息，则与菱形运算符组合的`var`类型可能导致意外推断类型。

在Java 7之前的Coin项目中，我们写了这样的代码:

``` Java
//显式指定泛型类的实例化参数类型
List<String> products = new ArrayList<String>();
```

从Java 7开始，我们有了菱形运算符，它能够推断泛型类实例化参数类型：

``` Java
// inferring generic class's instantiation parameter type 
List<String> products = new ArrayList<>();
```

那么，以下代码推断出什么类型呢？

首先应该避免这么用：

``` Java
// AVOID
var productList = new ArrayList<>(); // 推断为ArrayList<Object>
```

推断出的类型是Object的ArrayList。之所以会这样是因为没有找到能够推测到预期类型为String的信息，这会导致返回一个最广泛可用类型，Object。

所以为了避免这样的情形，我们必须提供能够推断到预测类型的信息。这个可以直接给也可以间接给。

更好的实现(直接)：

``` Java

// PREFER
var productList = new ArrayList<String>(); // 推断为ArrayList<String>
```

更好的实现（间接）：

``` Java
var productStack = new ArrayDeque<String>(); 
var productList = new ArrayList<>(productStack); // 推断为ArrayList<String>
```

更好的实现（间接）：

``` Java
Product p1 = new Product();
Product p2 = new Product();
var listOfProduct = List.of(p1, p2); // 推断为List<Product>
// 不要这么干
var listofProduct = List.of(); // 推断为List<Object>
listofProduct.add(p1);
listofProduct.add(p2);
```

## 9. 赋值数组到*var*不需要中括号[]

我们都知道Java中如何声明一个数组：

``` Java
int[] numbers = new int[5];
// 或者，这样写不太好
int numbers[] = new int[5];
```

那么怎么用var呢？左边不需要使用括号。

避免这么写（编译不通过）：

``` Java
// 编译通不过
var[] numbers = new int[5];
// 或者
var numbers[] = new int[5];
```

应该这么用：

``` Java
// PREFER
var numbers = new int[5]; // 推断为int数组
numbers[0] = 2;   // 对
numbers[0] = 2.2; // 错
numbers[0] = "2"; // 错
```

另外，这么用也不能编译，这是因为右边没有自己的类型。

```
// 显式类型表现符合预期
int[] numbers = {1, 2, 3};
// 编译通不过
var numbers = {1, 2, 3};
var numbers[] = {1, 2, 3};
var[] numbers = {1, 2, 3};
```

## 10. *var*类型不能被用于复合声明（一行声明多个变量）

如果你是复合声明的粉丝，你一定要知道`var`不支持这种声明。下面的代码不能编译：

``` Java

// 编译通不过
// error: 'var' 不允许复合声明
var hello = "hello", bye = "bye", welcome = "welcome";
```

用下面的代码代替：

``` Java
// PREFER
String hello = "hello", bye = "bye", welcome = "welcome";
```

或者这么用：

``` Java
// PREFER
var hello = "hello";
var bye = "bye";
var welcome = "welcome";
```

## 11. 局部变量应力求最小化其范围。*var*类型强化了这一论点。

局部变量应该保持小作用域，我确定你在`var`出现之前就听过这个，这样可以增强代码可读性，也方便更快的修复bug。

例如我们定义一个java栈：

避免：

``` Java
// AVOID
...
var stack = new Stack<String>();
stack.push("George");
stack.push("Tyllen");
stack.push("Martin");
stack.push("Kelly");
...
// 50行不用stack的代码
// George, Tyllen, Martin, Kelly  
stack.forEach(...);
...
```

注意我们调用`forEach` 方法，该方法继承自`java.util.Vector`.这个方法将以Vector的方式遍历栈。现在我们准备切换`Stack`到`ArrayDeque`，切换之后`forEach()`方法将变成`ArrayDeque`的，将以stack(LIFO)的方式遍历stack。

``` Java
// AVOID
...
var stack = new ArrayDeque<String>();
stack.push("George");
stack.push("Tyllen");
stack.push("Martin");
stack.push("Kelly");
...
// 50行不用stack的代码
// Kelly, Martin, Tyllen, George
stack.forEach(...);
...
```

这不是我们想要的，我们很难看出引入了一个错误，因为包含`forEach()`部分的代码不在研发完成修改的代码附近。为了快速修复这个错误，并避免上下滚动来了解发生了什么，最好缩小stack变量的作用域范围。

最好这么写：

``` Java
// PREFER
...
var stack = new Stack<String>();
stack.push("George");
stack.push("Tyllen");
stack.push("Martin");
stack.push("Kelly");
...
// George, Tyllen, Martin, Kelly  
stack.forEach(...);
...
// 50行不用stack的代码
```

现在，当开发人员从`Stack`切换到`ArrayQueue`的时候，他们能够很快的注意到bug，并修复它。

## 12. *var*类型便于三元运算符右侧的不同类型的操作数

我们可以在三元运算符的右侧使用不同类型的操作数。

使用具体类型的时候，以下代码无法编译：

``` Java
// 编译通不过
List code = containsDuplicates ? List.of(12, 1, 12) : Set.of(12, 1, 10);
// or
Set code = containsDuplicates ? List.of(12, 1, 12) : Set.of(12, 1, 10);
```

虽然我们可以这么写：

``` Java
Collection code = containsDuplicates ? List.of(12, 1, 12) : Set.of(12, 1, 10);
Object code = containsDuplicates ? List.of(12, 1, 12) : Set.of(12, 1, 10);
```

这样也编译不过：

``` Java
// 编译通不过：
int code = intOrString ? 12112 : "12112";
String code = intOrString ? 12112 : "12112";
```

但是我们可以这么写：

``` Java
Serializable code = intOrString ? 12112 : "12112";
Object code = intOrString ? 12112 : "12112";
```

在这种情况下，使用`var`更好：

```
// PREFER
// inferred as Collection<Integer>
var code = containsDuplicates ? List.of(12, 1, 12) : Set.of(12, 1, 10);
// inferred as Serializable
var code = intOrString ? 12112 : "12112";
```

千万不要从这些例子中得出var类型是在运行时做类型推断的，它不是！！！

当然，我们使用相同的类型作为操作数时`var`是支持的。

``` Java
// 推断为float
var code = oneOrTwoDigits ? 1211.2f : 1211.25f;
```

## 13. *var*类型能够用在循环体中

我们能非常简单的在for循环中用`var`类型取代具体类型。这是两个例子。

var替换int：

``` Java
// 显式类型
for (int i = 0; i < 5; i++) {
     ...
}
// 使用 var
for (var i = 0; i < 5; i++) { // i 推断为 int类型
     ...
}
```

var替换Order：

``` Java
List<Order> orderList = ...;
// 显式类型
for (Order order : orderList) {
    ...
}
// 使用 var
for (var order : orderList) { // order 推断成Order类型
    ...
}
```

## 14. var类型能够和Java 8中的Stream一起用

将Java10中的var与Java 8中的Stream结合起来非常简单。

你需要使用var取代显式类型Stream:

例1：

``` Java
// 显式类型
Stream<Integer> numbers = Stream.of(1, 2, 3, 4, 5);                
numbers.filter(t -> t % 2 == 0).forEach(System.out::println);
// 使用 var
var numbers = Stream.of(1, 2, 3, 4, 5); // 推断为 Stream<Integer>               
numbers.filter(t -> t % 2 == 0).forEach(System.out::println);
```

例2:

``` Java
// 显式类型
Stream<String> paths = Files.lines(Path.of("..."));
List<File> files = paths.map(p -> new File(p)).collect(toList());
// 使用 var
var paths = Files.lines(Path.of("...")); // 推断为 Stream<String>
var files = paths.map(p -> new File(p)).collect(toList()); // 推断为 List<File>
```

## 15. *var*类型可用于声明局部变量，可用于分解表达式嵌套/长链

`var`类型可用于声明局部变量，可用于分解表达式嵌套/长链.

大的或者嵌套的表达看起来令人印象深刻，通常它们被认为是聪明的代码。有时候我们会故意这么写，有时候我们从一个小表达式开始写，慢慢越来越大。为了提高代码可读性，建议用局部变量来破坏大型/嵌套表达式。但有时候，添加这些局部变量是我们想要避免的体力活。如下：

避免：

``` Java
List<Integer> intList = List.of(1, 1, 2, 3, 4, 4, 6, 2, 1, 5, 4, 5);
// AVOID
int result = intList.stream()
    .collect(Collectors.partitioningBy(i -> i % 2 == 0))
    .values()
    .stream()
    .max(Comparator.comparing(List::size))
    .orElse(Collections.emptyList())
    .stream()
    .mapToInt(Integer::intValue)
    .sum();
```

更好：

``` Java

List<Integer> intList = List.of(1, 1, 2, 3, 4, 4, 6, 2, 1, 5, 4, 5);
// PREFER
Map<Boolean, List<Integer>> evenAndOdd = intList.stream()
    .collect(Collectors.partitioningBy(i -> i % 2 == 0));
Optional<List<Integer>> evenOrOdd = evenAndOdd.values()
    .stream()
    .max(Comparator.comparing(List::size));
int sumEvenOrOdd = evenOrOdd.orElse(Collections.emptyList())
    .stream()
    .mapToInt(Integer::intValue)
    .sum();
```

第二段代码可读性更强，更简洁，但是第一段代码也是对的。我们的思维会适应这样的大表达式并且更喜欢它们而不是局部变量。然而，使用var类型对于使用局部变量的方式来说是一个优化，因为它节省了获取显式类型的时间。

更好

``` Java
var intList = List.of(1, 1, 2, 3, 4, 4, 6, 2, 1, 5, 4, 5);
// PREFER
var evenAndOdd = intList.stream()
    .collect(Collectors.partitioningBy(i -> i % 2 == 0));
var evenOrOdd = evenAndOdd.values()
    .stream()
    .max(Comparator.comparing(List::size));
var sumEvenOrOdd = evenOrOdd.orElse(Collections.emptyList())
    .stream()
    .mapToInt(Integer::intValue)
    .sum();
```

## 16. *var*类型不能被用于方法返回类型或者方法参数类型。

试着写下面的两段代码，编译通不过。

使用var作为方法返回类型：

``` Java
// 编译通不过
public var countItems(Order order, long timestamp) {
    ...        
}
```

使用var作为方法参数类型：

``` Java
// 编译通不过
public int countItems(var order, var timestamp) {
    ...  
}
```

## 17. *var*类型的局部变量可以用来传入到方法参数，也可以用来存放方法返回值

`var`类型的局部变量可以用来传入到方法参数，也可以用来存放方法返回值。下面这两段代码能够编译而且运行。

``` Java
public int countItems(Order order, long timestamp) {
    ...
}
public boolean checkOrder() {
    var order = ...;     // Order实例
    var timestamp = ...; // long类型的 timestamp
    var itemsNr = countItems(order, timestamp); // 推断为int类型
    ...
}

```

它也适用于泛型。下面的代码片段也是对的。

``` Java
public <A, B> B contains(A container, B tocontain) {
    ...
}
var order = ...;   // Order实例
var product = ...; // Product实例
var resultProduct = contains(order, product); // inferred as Product type
```

## 18. *var*类型能和匿名类一起使用。

避免：

```
public interface Weighter {
    int getWeight(Product product);
}
// AVOID
Weighter weighter = new Weighter() {
    @Override
    public int getWeight(Product product) {
        ...
    }
};
Product product = ...; // Product实例
int weight = weighter.getWeight(product);
```

更好的代码：

``` Java
public interface Weighter {
    int getWeight(Product product);
}
// PREFER
var weighter = new Weighter() {
    @Override
    public int getWeight(Product product) {
        ...
    }
};
var product = ...; // Product实例
var weight = weighter.getWeight(product);
```

## 19. *var*类型可以是Effectively Final

从Java SE 8开始，局部类可以访问封闭块内final或者effectively final的参数。变量初始化后不再改变的参数为effectively final。

所以，var类型的变量可以是effectively final的。我们可以从以下代码中看到。

避免：

``` Java
public interface Weighter {
    int getWeight(Product product);
}
// AVOID
int ratio = 5; // 这是effectively final
Weighter weighter = new Weighter() {
    @Override
    public int getWeight(Product product) {
        return ratio * ...;
    }
};
ratio = 3; // 这行赋值语句会报错
```

更好：

``` Java
public interface Weighter {
    int getWeight(Product product);
}
// PREFER
var ratio = 5; // 这是effectively final
var weighter = new Weighter() {
    @Override
    public int getWeight(Product product) {
        return ratio * ...;
    }
};
ratio = 3; // 这行赋值语句会报错
```

## 20. *var*类型可以用final修饰

默认情况下，var类型的局部变量可以被重新赋值（除非它是effectively final的）。但是我们可以声明它为final类型，如下：

避免：

``` Java

// AVOID
// IT DOESN'T COMPILE
public void discount(int price) {
    final int limit = 2000;
    final int discount = 5;
    if (price > limit) {
        discount++; // 这行会报错
    }
}
```

更好：

``` Java
// PREFER
// IT DOESN'T COMPILE
public void discount(int price) {
    final var limit = 2000;
    final var discount = 5;
    if (price > limit) {
        discount++; // 这行会报错
    }
}

```

## 21. Lambda表达式和方法引用需要显示对象类型

当对应的类型推断不出来时不能使用var类型。所以，lambda表达式和方法引用初始化不被允许。这是var类型限制的一部分。

下面的代码无法编译：

``` Java
// 编译不通过
// lambda表达式需要显式目标类型
var f = x -> x + 1;
// 方法引用需要显式目标类型
var exception = IllegalArgumentException::new;
```

用以下代码代替：

```

// PREFER
Function<Integer, Integer> f = x -> x + 1;
Supplier<IllegalArgumentException> exception = IllegalArgumentException::new;
```

但是在lambda的内容中，Java 11允许我们去使用var作为lambda参数。例如，以下代码在Java 11中可以很好的工作(详见[JEP 323](https://openjdk.java.net/jeps/323)(lambda参数中的局部变量))

``` Java
// Java 11
(var x, var y) -> x + y
// or 
(@Nonnull var x, @Nonnull var y) -> x + y
```

## 22. 为*var*类型赋值为null是不被允许的。

此外，也不允许缺少初始化程序。这是var类型的另一个限制。

以下代码不会编译通过(赋值null)：

``` Java
// 编译通不过
var message = null; // 类型错误: 变量初始化为'null'
```

这个代码也不会编译通过（缺少初始化）：

``` Java
// IT DOESN'T COMPILE
var message; // 使用var不能不做初始化
...
message = "hello";
```

更好：

``` Java
// PREFER
String message = null;
// or
String message;
...
message = "hello";
```

## 23. *var*类型不能作为对象的域（Field）

*var*类型可以用来做局部变量，但是不能用来做对象的域/全局变量。

这个限制会导致这里的编译错误：

``` Java
// 编译通不过
public class Product {
    private var price; // 'var' 不被允许
    private var name;  // 'var' 不被允许
    ...
}
```

用以下代码代替：

``` Java
// PREFER
public class Product {
    private int price; 
    private String name;
    ...
}
```

## 24. *var*不被允许在catch块中使用

但是它被允许在try-with-resources中。

### catch块

当代码抛出异常时，我们必须通过显式类型catch它，因为var类型不被允许。这个限制会导致以下代码的编译时错误：

``` Java
// 编译通不过
try {
    TimeUnit.NANOSECONDS.sleep(5000);
} catch (var ex) {
    ...
}
```

用这个取代：

``` Java
// PREFER
try {
    TimeUnit.NANOSECONDS.sleep(5000);
} catch (InterruptedException ex) {
    ...
}
```
### try-with-resources

另一方面，var类型可以用在try-with-resource中，例如：

``` Java
// 显式类型
try (PrintWriter writer = new PrintWriter(new File("welcome.txt"))) {
    writer.println("Welcome message");
}
```

可以用var重写：

``` Java
// 使用 var
try (var writer = new PrintWriter(new File("welcome.txt"))) {
    writer.println("Welcome message");
}
```


## 25. *var*类型不能和泛型T一起使用

假定我们有下面的代码：

``` Java
public <T extends Number> T add(T t) {
     T temp = t;
     ...
     return temp;   
}
```

这种情况下，使用var的运行结果是符合预期的，我们可以用var替换T，如下：

``` Java
public <T extends Number> T add(T t) {
     var temp = t;
     ...
     return temp;   
}
```

我们看一下另一个var能够成功使用的例子，如下：

``` Java
public <T extends Number> T add(T t) {
     List<T> numbers = new ArrayList<>();
     numbers.add((T) Integer.valueOf(3));
     numbers.add((T) Double.valueOf(3.9));
     numbers.add(t);
     numbers.add("5"); // 错误：类型不兼容，string不能转为T
     ...     
}
```

可以用var取代List<T>, 如下：

``` Java
public <T extends Number> T add(T t) {
     var numbers = new ArrayList<T>();
     // DON'T DO THIS, DON'T FORGET THE, T
     var numbers = new ArrayList<>();
     numbers.add((T) Integer.valueOf(3));
     numbers.add((T) Double.valueOf(3.9));
     numbers.add(t);
     numbers.add("5"); // // 错误：类型不兼容，string不能转为T
     ...     
}
```


## 26. 使用带有var类型的通配符(?)，协方差和反对变量时要特别注意

### 使用？通配符

这么做是安全的：

``` Java
// 显式类型
Class<?> clazz = Integer.class;
// 使用var
var clazz = Integer.class;
```

但是，不要因为代码中有错误，而var可以让它们魔法般的消失，就使用var取代Foo<?>。看下一个例子，不是非常明显，但是我想让它指出核心。考虑一下当你编写这一段代码的过程，也许，你尝试定义一个String的ArrayList，并最终定义成了Collection<?>。


``` Java
// 显式类型
Collection<?> stuff = new ArrayList<>();
stuff.add("hello"); // 编译错误
stuff.add("world"); // 编译错误
// 使用var，错误会消失，但是我不确定你是你想要的结果
var stuff = new ArrayList<>();
strings.add("hello"); // 错误消失
strings.add("world"); // 错误消失
```

### Java协变（Foo<? extends T>）和逆变(Foo<? super T>)

我们知道可以这么写：

``` Java
// 显式类型
Class<? extends Number> intNumber = Integer.class;
Class<? super FilterReader> fileReader = Reader.class;
```

而且如果我们错误赋值了错误的类型，接收到一个编译时错误，这就是我们想要的：

``` Java
// 编译通不过
// 错误: Class<Reader> 不能转换到 Class<? extends Number>
Class<? extends Number> intNumber = Reader.class;
// 错误: Class<Integer> 不能转化到Class<? super FilterReader>
Class<? super FilterReader> fileReader = Integer.class;
```

但是如果我们使用var:

``` Java
// using var
var intNumber = Integer.class;
var fileReader = Reader.class;
```

然后我们可以为这些变量赋值任何类，因此我们的边界/约束消失了，这并不是我们想要的。

``` Java
// 编译通过
var intNumber = Reader.class;
var fileReader = Integer.class;
```


## 结论

var类型非常酷，而且将来会继续升级。留心[JEP 323](https://openjdk.java.net/jeps/323) 和 [JEP 301](http://openjdk.java.net/jeps/301)了解更多。祝您愉快。

