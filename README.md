ceph-dencoder是用于帮助码农进行ceph debug的小工具，用户简单的dump出ceph的数据及相应的个字段的值。官网介绍是这样的：

> ceph-dencoder is a utility to encode, decode, and dump ceph data structures. It is used for debugging and for testing inter-version compatibility.


下面介绍一下我使用它解决一个疑问的过程。

之前就知道，ceph使用了扩展属性来存储object的元数据，但对于它如何使用它，却没有更直观的认识，我注意到，每次ceph写一个object之后，观察user.ceph_这个扩展属性，都会发生变化，但这是一个很长的字符串（如下），有没有办法把它取出来看一眼呢？
> [root@haha 1.0_head]# getfattr -d * 
> 
> file: test__head_40E8AAB5__1
> user.ceph._=0sDgjhAAAABAMlAAAAAAAAAAQAAAB0ZXN0/v////////+1quhAAAAAAAABAAAAAAAAAAYDHAAAAAEAAAAAAAAA/////wAAAAAAAAAA//////////8AAAAAmVEAAAAAAABnAAAAAAAAAAAAAAAAAAAAAgIVAAAACNETAAAAAAAAAQAAAAAAAAAAAAAAAABAAAAAAABrpcVUELluNgICFQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAJlRAAAAAAAAAAAAAAAAAAAABAAAAGulxVQILOA2
> user.ceph.snapset=0sAgIZAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAAA==
> user.cephos.spill_out=0sMAA=


首先我们可以猜测，里边肯定有一部分存储的是size，mtime，object name之类的类似于inode里边存储的数据，但是具体是什么呢，这就要借助于ceph-dencoder来看一看了，

> [root@haha 1.0_head]# attr test__head_40E8AAB5__1 -g ceph._ -q /tmp/a    
> [root@haha 1.0_head]#  ceph-dencoder type object_info_t import /tmp/a decode dump_json
> { "oid": { "oid": "test",
> "key": "",
> "snapid": -2,
> "hash": 1088989877,
> "max": 0,
> "pool": 1,
> "namespace": ""},
> "category": "",
> "version": "103'20889",
> "prior_version": "0'0",
> "last_reqid": "client.5073.0:1",
> "user_version": 20889,
> "size": 4194304,
> "mtime": "2015-01-26 10:24:43.913226",
> "local_mtime": "2015-01-26 10:24:43.920661",
> "lost": 0,
> "flags": 4,
> "wrlock_by": "unknown.0.0:0",
> "snaps": [],
> "truncate_seq": 0,
> "truncate_size": 0,
> "watchers": {}}


于是，我们能够看到，这个扩展属性的全部内容了，其中hash字段存储的是object的哈希值，而且这个字段是用于构造文件系统中存储的文件“test__head_40E8AAB5__1 ”的一部分，1088989877转化为16进制即为40E8AAB5。
另外，存储的还有object的size，mtime等等。
