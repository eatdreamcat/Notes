# Linux时间轮定时器

核心思想 ： 只关心目前即将到期的任务

参考：

linux:      https://github.com/torvalds/linux

skynet:   https://github.com/cloudwu/skynet

* HZ 1000 steps

 * Level   Offset    Granularity                    Range

 * 0            0            1 ms                           0 ms -         63 ms

 * 1           64           8 ms                           64 ms -        511 ms

 * 2          128          64 ms                         512 ms -       4095 ms (512ms - ~4s)

 * 3          192         512 ms                        4096 ms -      32767 ms (~4s - ~32s)

 * 4          256        4096 ms (~4s)             32768 ms -     262143 ms (~32s - ~4m)

 * 5          320     32768 ms (~32s)            262144 ms -    2097151 ms (~4m - ~34m)

 * 6          384    262144 ms (~4m)            2097152 ms -   16777215 ms (~34m - ~4h)

 * 7          448    2097152 ms (~34m)       16777216 ms -  134217727 ms (~4h - ~1d)

 * 8          512    16777216 ms (~4h)         134217728 ms - 1073741823 ms (~1d - ~12d)

   

​       将大量的任务，按照到期时间expire划分进不同时间颗粒度的列表中，每一种列表称为一个时间轮，不同颗粒度的时间轮之间，存在一定比例的周期关系。

![](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/均匀分布1.png)

## 算法实现步骤

1. 设定一个固定的频率HZ，即一秒钟需要Tick多少次。

   比如1000HZ代表一秒钟Tick1000次，则每一次为1ms，每一次Tick称之为一个Jiff。

2. 计算Task的Index，按照时间轮的分级

3. 每次Tick只关注最低等级的时间轮

4. 当Tick次数是各级颗粒度整数倍时，则将高颗粒度的轮对应的当前格内的Task列表重新散列到上一级轮内

   

   ![](C:\Users\吃吃\Desktop\Notes\TimeWheel\20200508142827872.png)

   

   ## 压测

   ### 十万数据压测

​       ![](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/十万压测.png)

![](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/十万压测2.png)

	### 百万数据压测

![](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/百万压测.png)



## 精准度测试

添加262144个延迟任务，每个任务间隔一个Tick执行

![](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/精准度测试1.png)

![](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/精准度测试2.png)

![](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/精准度测试3.png)

## 优化TODO

1. 考虑多线程情况 (例如Tick当前是在主线程的Update中驱动，但是用户如果在子线程添加Task，可能会导致读写竞争)
2. Task堆积在一个Slot的情况，会导致遍历的当前Slot内的任务数过多
3. Task排序，当最小颗粒度比较大时，比如33ms，一个延迟30ms的任务可能比一个延迟20ms的任务先执行（默认会按照添加顺序执行）
4. 高耗时的同步Task





# TODO

1.linux 链表快速索引的实现