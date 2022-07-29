[toc]
# MyDB
My MySQL 

本项目借鉴于[GuoZiyang](https://github.com/CN-GuoZiyang/MYDB) 和[@qw4990](https://github.com/qw4990/NYADB2) 两位大佬的开源项目

## TM(TransactionManager)
事务管理
1. 提供了其他模块来查询、修改某个事务的状态的接口。
2. 还提供了begin、commit、abort接口来管理事务桩体

TM通过维护一个**.tid文件**来维护事务的状态，文件头部前8个字节
记录了管理的事务的个数。

**超级事务**：TID为0，永远为committed状态，当一些操作想在没有申请事务的情况下进行，那么可以将操作的 TID 设置为 0。

**普通事务**：每个事务的占用长度为1 byte，故事务在文件中的位置为
8 + tid - 1; 普通事务tid从1开始。

**TID文件合法性检查**：每次启动数据库时需要对TID文件进行检查。
判断TID文件的长度 == 8(首部) + (事务的数量[首部8个字节记录] - 1)*事务大小。不等就说明文件错误，需要手动检查。

## DM(DataManager)

DM 直接管理数据库 DB 文件和日志文件。DM 的主要职责有：
1) 分页管理 DB 文件，并进行缓存；
2) 管理日志文件，保证在发生错误时可以根据日志进行恢复；
3) 抽象 DB 文件为 DataItem 供上层模块使用，并提供缓存。.

db文件以.db文件结尾

### 分页管理DB文件

AbstractCache: 实现了一个引用计数策略的缓存的抽象类。子类可以通过实现getForCache()和releaseForCache这两个方法来加载数据到缓存中和从缓存中释放数据

PageCacheImpl ： 页面缓存的具体实现

**PageOne：** 第一页，用于启动数据库时的检查。在每次数据库启动时，会生成一串随机字节，存储在 100 ~ 107 字节。在数据库正常关闭时，会将这串字节，拷贝到第一页的 108 ~ 115 字节。数据库在每次启动时，就会检查第一页两处的字节是否相同
，来判断上一次是否正常关闭。

**PageX：** 普通页面，默认每一页的大小为8KB，页面的页号从1开始，前两个字节记录了当前页的空闲空间的页内偏移位置（一页的大小为8KB， 2个字节完全可以表示）。

### 日志文件管理
