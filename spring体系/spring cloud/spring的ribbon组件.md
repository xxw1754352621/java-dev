# **ribbon组件**
- 调度（资源分配）
  - 算法：轮询调度算法(Round-Robin Scheduling) 
    - 缺点：(非阻塞)不关心每台服务器的当前连接数和响应速度，不利于后面的请求及时得到响应
    -     j = i;
          do {
              j = (j + 1) mod n;
              i = j;
              return Si;
          } while (j != i);
          return NULL;
  - 改进算法：权重轮询算法(Weighted Round-Robin)
    - 避免低性能的服务器负载过重
    -     3台服务器，当前权重（2:3:1），计算位置（2,5,6），二次位置（11,101,110），第5次请求，负载在第2台服务器
          方法一：
          int n=3;
          int weight（i）{
            return i^(2+3+1)
          }
          
          i=10，位置=4，二次位置=100,4<5,所以第二台
          
          方法二：
          记住上一次调用的服务器
          int[] services={2,3,1};
          volatile int n=3;
          volatile int lastService=0；
          volatile int lastWeight=services[lastService];
          int getService(){
                 //synchronized(object){}
                  if(lastWeight==0) 
                  {
                      lastService=(lastService++) mod n ;
                      lastWeight=services[lastService]
                      return lastService;
                  }
                  else{
                      lastWeight--;
                      return lastService;
                  }
                  
              
          }
          
      
- 源码：
