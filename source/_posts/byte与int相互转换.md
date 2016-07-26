---
title: byte与int相互转换
date: 2016-04-18 15:18:24
tags: java
---
# byte转int
- 64->100
```
    public static int byteToInt(byte src) {
        int value;
        value = src & 0xFF;
        return value;
    }
```
<!-- more -->
# byte[]转int(低位在前，高位在后)
- E0 07 00 00->2016
```
     /**
     * byte数组中取int数值，本方法适用于(低位在前，高位在后)的顺序. 
     * @param src    byte数组
     * @param offset 从数组的第offset位开始
     * @return int数值
     */
    public static int bytesToInt(byte[] src, int offset) {
        int value;
        value = ((src[offset] & 0xFF)
                | ((src[offset + 1] & 0xFF) << 8)
                | ((src[offset + 2] & 0xFF) << 16)
                | ((src[offset + 3] & 0xFF) << 24));
        return value;
    }
```
# byte[]转int(低位在后，高位在前)
- 00 00 07 E0->2016
```
    /**
     * byte数组中取int数值，本方法适用于(低位在后，高位在前)的顺序。
     */
    public static int bytesToInt2(byte[] src, int offset) {
        int value;
        value = ((src[offset] & 0xFF) << 24)
                | ((src[offset + 1] & 0xFF) << 16)
                | ((src[offset + 2] & 0xFF) << 8)
                | (src[offset + 3] & 0xFF);
        return value;
    }
```
# int转byte[] (低位在前，高位在后)
- 2016->E0 07 00 00
```
    /**
     * int 转 byte[]
     * @param i 2016
     * @return E0 07 00 00
     */
    public static byte[] intToByte(int i) {
        byte[] result = new byte[4];
        result[0] = (byte) ((i >> 24) & 0xFF);
        result[1] = (byte) ((i >> 16) & 0xFF);
        result[2] = (byte) ((i >> 8) & 0xFF);
        result[3] = (byte) (i & 0xFF);
        return result;
    }
```
# int转byte[] (高位在前，低位在后)
- 2016->00 00 07 E0
```
    /**
     * 将int数值转换为占四个字节的byte数组高位在前，低位在后)的顺序
     */
    public static byte[] intToBytes2(int value) {
        byte[] src = new byte[4];
        src[0] = (byte) ((value >> 24) & 0xFF);
        src[1] = (byte) ((value >> 16) & 0xFF);
        src[2] = (byte) ((value >> 8) & 0xFF);
        src[3] = (byte) (value & 0xFF);
        return src;
    } 
```