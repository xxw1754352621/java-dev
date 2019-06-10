# Timsort排序
前提：
- 涉及算法
  - 插入排序（直接插入，二分查找插入，希尔排序）
    - 这里采用了二分查找插入
    - 平均      最坏	   最好	     稳定性	空间复杂度
    -  O(n^2)   O(n^2)	O(nlogn)	稳定	  O(1)
  - 归并排序
    - 平均	     最坏	     最好	     稳定性	空间复杂度
    -  O(nlogn)	O(nlogn)	O(nlogn)	稳定	  O(n+logn)
- 性能
  - Timsort排序
    - 平均	     最坏	     最好	 稳定性	空间复杂度
    - O(nlogn)	O(nlogn)	O(n)	稳定	  O(n)

### 数组 < 32
- 初始化数组 a .length = n     <   32  
- 计算临时空间个数tempBase，根据个数
- 比较出最小 运行起始位 ，a[3] 
- a[3]开始往前面二分查找插入（binarySort排序），以此类推

### 数组 >= 32
- 初始化数组 a .length = n     >=   32  
- 计算临时空间个数tempBase，根据个数
- 计算最小运行长度minRun（n>>1,直到n<32,
     然后n+1/0（最小16位），后面合并的规则使用）
- 执行binarySort排序第一个run，同n<32的操作
- 排序后,记住当前run信息（pushRun）
- 判断是否需要归并折叠（mergeCollapse）
- 继续找下一个run，相应参数修改

具体详情见：[Timsort排序.vsdx](https://github.com/xxw1754352621/java-dev/blob/master/java%E5%9F%BA%E7%A1%80/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/Timsort%E7%AE%97%E6%B3%95.vsdx)

