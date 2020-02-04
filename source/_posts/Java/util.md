---
title: Java工具类整理
date: 2018-04-05 00:00:00
author: vanliuzh
top: true
cover: false
toc: true
mathjax: false
summary: Java工具类整理
categories: Java
tags: [Java, Note]
reprintPolicy: cc_by
---

Java工具类整理，收集和整理的Java相关工具类，大部分来源网络，不过我都测试过了，哈哈


## Byte[]与hex互相转换，用在某些数据传输场景下

```java
import java.util.Arrays;

/**
 * Byte[]与hex的相互转换
 * @explain
 * @author Marydon
 * @creationTime 2018年6月11日下午2:29:11
 * @version 1.0
 * @since
 * @email marydon20170307@163.com
 */
public class ByteUtils {

    // 16进制字符
    private static final char[] HEX_CHAR = { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f' };
}

/**
 * 方法一：将byte类型数组转化成16进制字符串
 * @explain 字符串拼接
 * @param bytes
 * @return
 */
public static String toHexString(byte[] bytes) {
    StringBuilder sb = new StringBuilder();
    int num;
    for (byte b : bytes) {
        num = b < 0 ? 256 + b : b;
        sb.append(HEX_CHAR[num / 16]).append(HEX_CHAR[num % 16]);
    }
    return sb.toString();
}

/**
 * 方法二： byte[] to hex string
 * @explain 使用数组
 * @param bytes
 * @return
 */
public static String toHexString2(byte[] bytes) {
    // 一个byte为8位，可用两个十六进制位表示
    char[] buf = new char[bytes.length * 2];
    int a = 0;
    int index = 0;
    // 使用除与取余进行转换
    for (byte b : bytes) {
        if (b < 0)
            a = 256 + b;
        else
            a = b;

        // 偶数位用商表示
        buf[index++] = HEX_CHAR[a / 16];
        // 奇数位用余数表示
        buf[index++] = HEX_CHAR[a % 16];
    }
    // char[]-->String
    return new String(buf);
}

/**
 * 方法三： byte[]-->hexString
 * @explain 使用位运算
 * @param bytes
 * @return
 */
public static String toHexString3(byte[] bytes) {
    char[] buf = new char[bytes.length * 2];
    int index = 0;
    // 利用位运算进行转换，可以看作方法二的变型
    for (byte b : bytes) {
        buf[index++] = HEX_CHAR[b >>> 4 & 0xf];
        buf[index++] = HEX_CHAR[b & 0xf];
    }

    return new String(buf);
}

/**
 * 方法四：byte[]-->hexString
 * @param bytes
 * @return
 */
public static String toHexString4(byte[] bytes) {
    StringBuilder sb = new StringBuilder(bytes.length * 2);
    // 使用String的format方法进行转换
    for (byte b : bytes) {
        sb.append(String.format("%02x", new Integer(b & 0xff)));
    }

    return sb.toString();
}  

/**
 * 将16进制字符串转换为byte[]
 * @explain 16进制字符串不区分大小写，返回的数组相同
 * @param hexString
 *            16进制字符串
 * @return byte[]
 */
public static byte[] fromHexString(String hexString) {
    if (null == hexString || "".equals(hexString.trim())) {
        return new byte[0];
    }

    byte[] bytes = new byte[hexString.length() / 2];
    // 16进制字符串
    String hex;
    for (int i = 0; i < hexString.length() / 2; i++) {
        // 每次截取2位
        hex = hexString.substring(i * 2, i * 2 + 2);
        // 16进制-->十进制
        bytes[i] = (byte) Integer.parseInt(hex, 16);
    }

    return bytes;
}
```

## 通过反射操作对象私有方法和私有变量

```java
public class ReflectionUtils {
 
    /**
     * 获取私有成员变量的值
     * @param instance
     * @param filedName
     * @return
     */
    public static Object getPrivateField(Object instance, String filedName) throws NoSuchFieldException, IllegalAccessException {
        Field field = instance.getClass().getDeclaredField(filedName);
        field.setAccessible(true);
        return field.get(instance);
    }
 
    /**
     * 设置私有成员的值
     * @param instance
     * @param fieldName
     * @param value
     * @throws NoSuchFieldException
     * @throws IllegalAccessException
     */
    public static void setPrivateField(Object instance, String fieldName, Object value) throws NoSuchFieldException, IllegalAccessException {
        Field field = instance.getClass().getDeclaredField(fieldName);
        field.setAccessible(true);
        field.set(instance, value);
    }
 
    /**
     * 访问私有方法
     * @param instance
     * @param methodName
     * @param classes
     * @param objects
     * @return
     * @throws NoSuchMethodException
     * @throws InvocationTargetException
     * @throws IllegalAccessException
     */
    public static Object invokePrivateMethod(Object instance, String methodName, Class[] classes, String objects) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        Method method = instance.getClass().getDeclaredMethod(methodName, classes);
        method.setAccessible(true);
        return method.invoke(instance, objects);
    }
}
```