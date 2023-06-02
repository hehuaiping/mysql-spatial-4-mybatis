### 根据mysql spatial data 存储格式解析数据和写入数据
以Mysql空间数据类型 Geometry 来实操一波
Geometry 实际存储格式为：长度为25个字节

4个字节用于整数SRID（0）
1个字节（整数字节顺序）（1 =小字节序）
4个字节用于整数类型信息
8字节的双精度X坐标
8字节的双精度Y坐标

例如，POINT(1 -1)由以下25个字节的序列组成，每个序列由两个十六进制数字表示：
sql复制代码mysql> SET @g = ST_GeomFromText('POINT(1 -1)');
mysql> SELECT LENGTH(@g);
+------------+
| LENGTH(@g) |
+------------+
|         25 |
+------------+
mysql> SELECT HEX(@g);
+----------------------------------------------------+
| HEX(@g)                                            |
+----------------------------------------------------+
| 000000000101000000000000000000F03F000000000000F0BF |
+----------------------------------------------------+




组成大小值SRID4个字节00000000字节顺序1个字节01WKB类型4字节01000000X坐标8字节000000000000F03FY坐标8字节000000000000F0BF
读POINT
```java
// 模拟 POINT(1 -1)
        String pointstr = "000000000101000000000000000000F03F000000000000F0BF";
        byte[] bytes = HexUtil.decodeHex(pointstr);
        // 小端点排序（Java默认是大端点排序）
        ByteBuffer wrap = ByteBuffer.wrap(bytes)
                .order(ByteOrder.LITTLE_ENDIAN);
        int SRID = wrap.getInt();
        byte endian = wrap.get();
        int wkbType = wrap.getInt();
        double x = wrap.getDouble();
        double y = wrap.getDouble();
        System.out.println("SRID: " + SRID);
        System.out.println("endian: " + endian);
        System.out.println("wkbType: " + wkbType);
        System.out.println("x: " + x);
        System.out.println("y: " + y);
```
SRID: 0
endian: 1
wkbType: 1
x: 1.0
y: -1.0

写POINT
```java  
        // 小端点排序（Java默认是大端点排序）
        ByteBuffer wrap = ByteBuffer.allocate(25)
                .order(ByteOrder.LITTLE_ENDIAN);
        // SRID: 0        
        wrap.putInt(0);
        // endian: 1
        wrap.put((byte) 1);
        // wkbType: 1
        wrap.putInt(1);
        // x: 1.0
        wrap.putDouble(1);
        // y: -1.0
        wrap.putDouble(-1);
        byte[] array = wrap.array();
        String encodeHexStr = HexUtil.encodeHexStr(array, false);
        System.out.println(encodeHexStr);
```
000000000101000000000000000000F03F000000000000F0BF
