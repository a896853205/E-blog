# **mysql排序的优化**

## ***Mysql server逻辑框架***

 

![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/C5I6tuQmWty2LHkfPkGvPmqWv*xz98hWAhEFJoiMy0Qvn03gJojqq6fTgihwf4..yPvoBaEWfLcPXonxco1K4w!!/b&bo=ygIAAgAAAAARB*o!&rf=viewer_4) 

第一层：客户端和连接服务

第二层：主要完成大多数的核心服务功能(如select语句、过程、函数等等)

第三层：存储引擎层，负责MySQL中数据的存储和提取

第四层：数据存储层，将数据存储在运行于裸设备的文件系统之上

 

## **sql排序的执行**

首先使用sql语句建表:

 ```sql
CREATE TABLE user (

 id int(11) NOT NULL,

 city varchar(16) NOT NULL,

 name varchar(16) NOT NULL,

 age int(11) NOT NULL,

 PRIMARY KEY (id),

 KEY city (city)

) ENGINE=InnoDB;
 ```

 

`PRIMARY KEY`是主键,表中有唯一主键`id`

`KEY`是索引,表中有索引`city`

`InnoDB`是两大存储引擎之一,它的特点是支持事务和行级锁。对比 `Myisam` 的存储引擎，`InnoDB` 的缺点是写的处理效率差一些并且会占用更多的磁盘空间以保留数据和索引。

`MyISAM` : `data`存的是数据地址。索引是索引，数据是数据。

`InnoDB` : `data`存的是数据本身。索引也是数据。



## **全字段排序**

使用explain来分析的这条查询语句的执行情况，结果如下图：

![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4NbnuL4d71fWuJi8AWpp8ng3WXxjXFO.LsnxJRNL1.3UaylWZ33m2T2ylZDBws3uRjBmUm1iTvM.iGUYb8NKNcYA!/b&bo=tAJ5AAAAAAARF.8!&rf=viewer_4) 

`Extra`这个字段中的`Using filesort`表示的就是需要排序，`MySQL` 会给每个线程分配一块内存用于排序，称为`sort_buffer`。

运行过程：首先初始化`sort_buffer`，把需要查询的字段放入，然后按照索引`city`查询到`city='beijing'`的记录，取出整行，再取出三个字段的值，放入`sort_buffer`，重复至找不到`city=’beijing’`的记录，此时的`id`为`IDX`，`sort_buffer`的`name`是无序的，最后按`name`做快速排序（如果为排序开辟的内存不够，则使用外部排序），最终返回结果集。

 

缺点：就是如果查询要返回的字段很多的话，那么sort_buffer里面要放的字段数太多，这样内存里能够同时放下的行数很少，要分成很多个临时文件，排序的性能会很差。

![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4Ndscu5zHfgRIkf.bKceHRbSoN775pRevuLkLsykUA7sOCRa3wWj7pdB3IuS5LgNOThuFJ43QS3bhvtU8EWj.gUA!/b&bo=tAJxAQAAAAARF.Y!&rf=viewer_4) 

## **rowid 排序**

\* `oracle`中的`rowid`和`rownum`

`rowid`

`rowid`是物理结构上的，每插入一行数据，都会生成一条唯一的编号。可以说默认排序是根据`rownum`升序的，但是本质上还是根据`rowid`升序排列的。

`rownum`

`rownum`可以说是伪列，并不存在，是`Oracle`根据查询结果给每一行确定一个数字，从1开始，每一行递增1。

 

设置`rowid`排序方法：

```SET max_length_for_sort_data = 16;sql
SET max_length_for_sort_data = 16;
```



由于`city`、`name`、`age` 这三个字段的定义总长度超过了16，`max_length_for_sort_data`是 `MySQL` 中专门控制用于排序的行数据的长度的一个参数。它的意思是，如果单行的长度超过这个值，`MySQL `就认为单行太大，要换一个算法，也就是`rowid`排序。

 

运行过程：首先初始化`sort_buffer`，把`id`和`name`字段放入，然后按照索引`city`查询到`city=’beijing’`的记录，取出整行，再取出`id`和`name`字段的值，放入`sort_buffer`，重复至找不到`city=’beijing’`的记录，此时的`id`为`IDX`，`sort_buffer`的`name`是无序的，按`name`做快速排序，遍历排序结果，取前 1000 行，并按照 `id `的值回到原表中取出 `city`、`name` 和 `age `三个字段返回给客户端，最终返回结果集。

 

![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4NRUbo2Bd37YsAPurpROsuqbFJCWI1Mx.SemZBBEGD7*zmcYr3PyEgWes.VcwW6x105Afq9XUSnfSghp1JRIIu3o!/b&bo=ogR1AgAAAAADJ9M!&rf=viewer_4) 

 

`mysql`的原则是如果内存够，就要多利用内存，尽量减少磁盘访问。

对于` InnoDB` 表来说，`rowid` 排序会要求回表多造成磁盘读，因此不会被优先选择。

 

## **联合索引**

由于每次通过`city`取出的`name`都是无序的，如果能够保证从`city`这个索引上取出来的行，直接按照`name`排序即可降低内存的使用？因此想到了联合索引，创建`(city,name)`联合索引，`sql `语句如下：

 ```sql
alter table user add index city_user(city, name);
 ```

 

![img](http://m.qpic.cn/psc?/V119yzzJ3eTVp6/eDPUD.WZD3qAEkJwtXE4NSp8FpDx006L.7yrOrv6EcxFwWFwxqbdBQoomJMNeu3YrVcTPNovs3XXXWvXxMLdoNumwuMz2dVGpY9Jr*OfhNI!/b&bo=ogRhAQAAAAADF*Q!&rf=viewer_4) 

 

运行过程：

（1）从索引`(city,name)`找到第一个满足 `city='beijing’`条件的主键 `id`。

（2）到主键 `id` 索引取出整行，取 `name`、`city`、`age` 三个字段的值，作为结果集的一部分直接返回。

（3）从索引`(city,name)`取下一个记录主键` id`

（4）重复（2）（3）直到查到第1000条记录

 

这个查询过程不需要临时表，也不需要排序。可以使用`explain`语句查询，发现已经不需要排序，当然还可以再进行优化，就是直接创建`(city,name,age)`联合索引，可以省去主键索引这一步。

 

参考：https://www.cnblogs.com/y-rong/p/8110596.html

https://juejin.im/post/5e945b9651882573b7537c2a