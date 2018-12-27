---
layout:     post
title:      "Java二维码登录流程实现(包含短地址生成，含部分代码)"
subtitle:   "java qrcode login flow"
date:       2016-11-28 12:00:00
author:     "沈歌"
header-img: "img/home-bg-o.jpg"
tags:
        - qrcode
---


> 近年来，二维码的使用越来越风生水起，笔者最近手头也遇到了一个需要使用二维码扫码登录网站的活，所以研究了一下这一套机制，并用代码实现了整个流程，接下来就和大家聊聊二维码登录及的那些事儿。


#### 二维码原理

二维码是微信搞起来的，当年微信扫码二维码登录网页微信的时候，感觉很神奇，然而，我们了解了它的原理，也就没那么神奇了。二维码实际上就是通过黑白的点阵包含了一个url请求信息。端上扫码，请求url，做对应的操作。

#### 一般性扫码操作的原理

微信登录、支付宝扫码支付都是这个原理：

![二维码原理图](https://shenpengyan.github.io/img/in-post/java-qrcode-login-flow/qrcode.png)

如图所示：

**1. 请求二维码**

桌面端向服务器发起请求一个二维码的。

**2. 生成包含唯一id的二维码**

桌面端会随机生成一个id，id唯一标识这个二维码，以便后续操作。

**3. 端上扫码**

移动端扫码二维码，解chu出二维码中的url请求。 

**4. 移动端发送请求到服务器**

移动端向服务器发送url请求，请求中包含两个信息，唯一id标识扫的是哪个码，端上浏览器中特定的cookie或者header参数等会标识由哪个用户来进行扫码的。

**5. 服务器端通知扫码成功**

服务器端收到二维码中信息的url请求时，通知端上已经扫码成功，并添加必要的登录Cookie等信息。这里的通知方式一般有几种：websocket、轮训hold住请求直到超时、隔几秒轮训。

#### 二维码中url的艺术

##### 如何实现自有客户端和其他客户端扫码（如微信）表现的不同

比如，在业务中，你可能想要这样的操作，如果是你公司的二维码被其他app（如微信）所扫描，想要跳转一个提示页，提示页上可以有一个app的下载链接；而当被你自己的app所扫描时，直接进行对应的请求。

这种情况下，可以这样来做，所有二维码中的链接都进行一层加密，然后统一用另一个链接来处理。

如：`www.test.com/qr?p=xxxxxx`,p参数中包含服务器与客户端约定的加解密算法（可以是对称的也可以是非对称的),端上扫码到这种特定路径的时候，直接用解密算法解p参数，得到`www.testqr.com/qrcode?key=s1arV`,这样就可以向服务器发起请求了，而其他客户端因为不知道这个规则，只能直接去请求`www.test.com/qr?p=xxxxxx`，这个请求返回提示页。

##### 如何让二维码更简单

很多时候，又要马儿跑，又要马儿不吃草。想要二维码中带有很多参数，但是又不想要二维码太复杂，难以被扫码出来。这时候，就需要考虑如何在不影响业务的情况下让二维码变的简单。

* 与端上约定规则：比如定义编码信息中i参数为`1,2,3`表示不同的uri，端上来匹配遇到不同的i参数时请求哪个接口

* 简化一切能简化的地方：简化uri，简化参数中的key、value。如`www.a.com/q?k=s1arV`就比`www.abc.def.adfg.edu.com.cn/qrcode/scan?k=77179574e98a7c860007df62a5dbd98b` 要简化很多，生成的二维码要好扫很多。

* 简化唯一id参数：上一条中前一个请求中参数值只有5位，后一个请求中参数值为生成的32位md5值。生成一个端的key至关重要。

#### 示例代码

##### 生成二维码（去掉白边，增加中间的logo）

需要导入jar包：`zxing`的 `core-2.0.jar`

```
import java.awt.BasicStroke;
import java.awt.Color;
import java.awt.Graphics;
import java.awt.Graphics2D;
import java.awt.Image;
import java.awt.Shape;
import java.awt.geom.RoundRectangle2D;
import java.awt.image.BufferedImage;
import java.io.ByteArrayOutputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

import javax.imageio.ImageIO;
import com.google.zxing.BarcodeFormat;
import com.google.zxing.EncodeHintType;
import com.google.zxing.MultiFormatWriter;
import com.google.zxing.common.BitMatrix;
import com.google.zxing.qrcode.decoder.ErrorCorrectionLevel;

public class QrCodeUtil {
    private static final int BLACK = Color.black.getRGB();
    private static final int WHITE = Color.WHITE.getRGB();
    private static final int DEFAULT_QR_SIZE = 183;
    private static final String DEFAULT_QR_FORMAT = "png";
    private static final byte[] EMPTY_BYTES = new byte[0];
    
    public static byte[] createQrCode(String content, int size, String extension) {
        return createQrCode(content, size, extension, null);
    }

    /**
     * 生成带图片的二维码
     * @param content  二维码中要包含的信息
     * @param size  大小
     * @param extension  文件格式扩展
     * @param insertImg  中间的logo图片
     * @return
     */
    public static byte[] createQrCode(String content, int size, String extension, Image insertImg) {
        if (size <= 0) {
            throw new IllegalArgumentException("size (" + size + ")  cannot be <= 0");
        }
        ByteArrayOutputStream baos = null;
        try {
            Map<EncodeHintType, Object> hints = new HashMap<EncodeHintType, Object>();
            hints.put(EncodeHintType.CHARACTER_SET, "utf-8");
            hints.put(EncodeHintType.ERROR_CORRECTION, ErrorCorrectionLevel.M);

            //使用信息生成指定大小的点阵
            BitMatrix m = new MultiFormatWriter().encode(content, BarcodeFormat.QR_CODE, size, size, hints);
            
            //去掉白边
            m = updateBit(m, 0);
            
            int width = m.getWidth();
            int height = m.getHeight();
            
            //将BitMatrix中的信息设置到BufferdImage中，形成黑白图片
            BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
            for (int i = 0; i < width; i++) {
                for (int j = 0; j < height; j++) {
                    image.setRGB(i, j, m.get(i, j) ? BLACK : WHITE);
                }
            }
            if (insertImg != null) {
                // 插入中间的logo图片
                insertImage(image, insertImg, m.getWidth());
            }
            //将因为去白边而变小的图片再放大
            image = zoomInImage(image, size, size);
            baos = new ByteArrayOutputStream();
            ImageIO.write(image, extension, baos);
            
            return baos.toByteArray();
        } catch (Exception e) {
        } finally {
            if(baos != null)
                try {
                    baos.close();
                } catch (IOException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
        }
        return EMPTY_BYTES;
    }
    
    /**
     * 自定义二维码白边宽度
     * @param matrix
     * @param margin
     * @return
     */
    private static BitMatrix updateBit(BitMatrix matrix, int margin) {
        int tempM = margin * 2;
        int[] rec = matrix.getEnclosingRectangle(); // 获取二维码图案的属性
        int resWidth = rec[2] + tempM;
        int resHeight = rec[3] + tempM;
        BitMatrix resMatrix = new BitMatrix(resWidth, resHeight); // 按照自定义边框生成新的BitMatrix
        resMatrix.clear();
        for (int i = margin; i < resWidth - margin; i++) { // 循环，将二维码图案绘制到新的bitMatrix中
            for (int j = margin; j < resHeight - margin; j++) {
                if (matrix.get(i - margin + rec[0], j - margin + rec[1])) {
                    resMatrix.set(i, j);
                }
            }
        }
        return resMatrix;
    }
    
    // 图片放大缩小
    public static BufferedImage zoomInImage(BufferedImage originalImage, int width, int height) {
        BufferedImage newImage = new BufferedImage(width, height, originalImage.getType());
        Graphics g = newImage.getGraphics();
        g.drawImage(originalImage, 0, 0, width, height, null);
        g.dispose();
        return newImage;
    }
    
    private static void insertImage(BufferedImage source, Image insertImg, int size) {
        try {
            int width = insertImg.getWidth(null);
            int height = insertImg.getHeight(null);
            width = width > size / 6 ? size / 6 : width;  // logo设为二维码的六分之一大小
            height = height > size / 6 ? size / 6 : height;
            Graphics2D graph = source.createGraphics();
            int x = (size - width) / 2;
            int y = (size - height) / 2;
            graph.drawImage(insertImg, x, y, width, height, null);
            Shape shape = new RoundRectangle2D.Float(x, y, width, width, 6, 6);
            graph.setStroke(new BasicStroke(3f));
            graph.draw(shape);
            graph.dispose();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static byte[] createQrCode(String content) {
        return createQrCode(content, DEFAULT_QR_SIZE, DEFAULT_QR_FORMAT);
    }

    public static void main(String[] args){
        try {
            FileOutputStream fos = new FileOutputStream("ab.png");
            fos.write(createQrCode("test"));
            fos.close();
        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        
    }
    
}

```

##### 生成短链接

基本思路：

短网址映射算法的理论：

1. 将长网址加随机数用用md5算法生成32位签名串，分为4段,每段8个字符

2. 对这4段循环处理，取每段的8个字符, 将他看成16进制字符串与0x3fffffff(30位1)的位与操作，超过30位的忽略处理

3. 将每段得到的这30位又分成6段，每5位的数字作为字母表的索引取得特定字符，依次进行获得6位字符串；

4. 这样一个md5字符串可以获得4个6位串，取里面的任意一个就可作为这个长url的短url地址。 

5. 最好是用一个key-value数据库存储，万一发生碰撞换一个，如果四个都发生碰撞，重新生成md5（因为有随机数，会生成不一样的md5）


```
public class ShortUrlUtil {

    /**
     * 传入32位md5值
     * @param md5
     * @return
     */
    public static String[] shortUrl(String md5) {
        // 要使用生成 URL 的字符
        String[] chars = new String[] { "a", "b", "c", "d", "e", "f", "g", "h",
                "i", "j", "k", "l", "m", "n", "o", "p", "q", "r", "s", "t",
                "u", "v", "w", "x", "y", "z", "0", "1", "2", "3", "4", "5",
                "6", "7", "8", "9", "A", "B", "C", "D", "E", "F", "G", "H",
                "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T",
                "U", "V", "W", "X", "Y", "Z"

        };
        
        String[] resUrl = new String[4];  
        
        for (int i = 0; i < 4; i++) {

            // 把加密字符按照 8 位一组 16 进制与 0x3FFFFFFF 进行位与运算，超过30位的忽略
            String sTempSubString = md5.substring(i * 8, i * 8 + 8);

            // 这里需要使用 long 型来转换，因为 Inteper .parseInt() 只能处理 31 位 , 首位为符号位 , 如果不用 long ，则会越界
            long lHexLong = 0x3FFFFFFF & Long.parseLong(sTempSubString, 16);
            String outChars = "";
            for (int j = 0; j < 6; j++) {
                // 把得到的值与 0x0000003D 进行位与运算，取得字符数组 chars 索引
                long index = 0x0000003D & lHexLong;
                // 把取得的字符相加
                outChars += chars[(int) index];
                // 每次循环按位右移 5 位
                lHexLong = lHexLong >> 5;
            }
            // 把字符串存入对应索引的输出数组
            resUrl[i] = outChars;
        }
        return resUrl;
    }
    
    public static void main(String [] args){
        String[] test = shortUrl("fdf8d941f23680be79af83f921b107ac");
        for (String string : test) {
            System.out.println(string);
        }
    }
    
}

```
