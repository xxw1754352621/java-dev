# JVM[堆的配置比例修改](https://www.cnblogs.com/SaraMoring/p/5713732.html)

1.Java heap的大小（新生代+老年代）

　　-Xms堆的最小值

　　-Xmx堆空间的最大值

2.新生代堆空间大小调整

　　-XX:NewSize新生代的最小值

　　-XX:MaxNewSize新生代的最大值

​        -XX:G1NewSizePercent=40 和 -XX:G1MaxNewSizePercent=90

　　-XX:NewRatio设置新生代与老年代在堆空间的大小

　　-XX:SurvivorRatio新生代中Eden所占区域的大小

3.永久代大小调整

　　-XX:MaxPermSize

4.其他

　  -XX:MaxTenuringThreshold,设置将新生代对象转到老年代时需要经过多少次垃圾回收，但是仍然没有被回收

​     -XX:G1MixedGCLiveThresholdPercent=100。