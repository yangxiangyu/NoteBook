#mysqlt tune

##mysql体系机构
![mysql 体系结构](http://images.cnblogs.com/cnblogs_com/yjf512/105018221.png  "mysql 体系结构")

- connectors 不同语言与sql的交互
- Management Services & Utillties系统管理和控制工具
- Connection Pool 链接池
	- 强制限制
您可以在 mysqld 中强制一些限制来确保系统负载不会导致资源耗尽的情况出现。清单 3 给出了 my.cnf 中与资源有关的一些重要设置。
清单 3. MySQL 资源设置             
set-variable=max_connections=500
set-variable=wait_timeout=10
max_connect_errors = 100
 连接最大个数是在第一行中进行管理的。与 Apache 中的 MaxClients 类似，其想法是确保只建立服务允许数目的连接。要确定服务器上目前建立过的最大连接数，请执行 SHOW STATUS LIKE 'max_used_connections'。
第 2 行告诉 mysqld 终止所有空闲时间超过 10 秒的连接。在 LAMP 应用程序中，连接数据库的时间通常就是 Web 服务器处理请求所花费的时间。有时候，如果负载过重，连接会挂起，并且会占用连接表空间。如果有多个交互用户或使用了到数据库的持久连接，那么将这个值设低一点并不可取！
最后一行是一个安全的方法。如果一个主机在连接到服务器时有问题，并重试很多次后放弃，那么这个主机就会被锁定，直到 FLUSH HOSTS 之后才能运行。默认情况下，10 次失败就足以导致锁定了。将这个值修改为 100 会给服务器足够的时间来从问题中恢复。如果重试 100 次都无法建立连接，那么使用再高的值也不会有太多帮助，可能它根本就无法连接。
- SQL Interface SQL接口
- Parser 解释器 将sql语句分解为数据结构 ，当出错时候说明这个sql是不可用的
- Optimlzer 查询优化器，按照“选取-投影-链接”进行优化
	- 临时表可以在更高级的查询中使用，其中数据在进一步进行处理（例如 GROUP BY 字句）之前，都必须先保存到临时表中；理想情况下，在内存中创建临时表。但是如果临时表变得太大，就需要写入磁盘中。清单 7 给出了与临时表创建有关的统计信息。
清单 7. 确定临时表的使用       
mysql> SHOW STATUS LIKE 'created_tmp%';
+-------------------------+-------+
| Variable_name           | Value |
+-------------------------+-------+
| Created_tmp_disk_tables | 30660 |
| Created_tmp_files       | 2     |
| Created_tmp_tables      | 32912 |
+-------------------------+-------+
3 rows in set (0.00 sec)
每次使用临时表都会增大 Created_tmp_tables；基于磁盘的表也会增大 Created_tmp_disk_tables。对于这个比率，并没有什么严格的规则，因为这依赖于所涉及的查询。长时间观察 Created_tmp_disk_tables 会显示所创建的磁盘表的比率，您可以确定设置的效率。 tmp_table_size 和 max_heap_table_size 都可以控制临时表的最大大小，因此请确保在 my.cnf 中对这两个值都进行了设置。
- Caches & Buffers 
    - 对查询进行缓存 query_cache_size 
很多 LAMP 应用程序都严重依赖于数据库，但却会反复执行相同的查询。每次执行查询时，数据库都必须要执行相同的工作 —— 对查询进行分析，确定如何执行查询，从磁盘中加载信息，然后将结果返回给客户机。MySQL 有一个特性称为查询缓存，它将（后面会用到的）查询结果保存在内存中。在很多情况下，这会极大地提高性能。不过，问题是查询缓存在默认情况下是禁用的。
将 query_cache_size = 32M 添加到 /etc/my.conf 中可以启用 32MB 的查询缓存。
监视查询缓存
在启用查询缓存之后，重要的是要理解它是否得到了有效的使用。MySQL 有几个可以查看的变量，可以用来了解缓存中的情况。清单 2 给出了缓存的状态。
清单 2. 显示查询缓存的统计信息             
mysql> SHOW STATUS LIKE 'qcache%';
+-------------------------+------------+
| Variable_name           | Value      |
+-------------------------+------------+
| Qcache_free_blocks      | 5216       |
| Qcache_free_memory      | 14640664   |
| Qcache_hits             | 2581646882 |
| Qcache_inserts          | 360210964  |
| Qcache_lowmem_prunes    | 281680433  |
| Qcache_not_cached       | 79740667   |
| Qcache_queries_in_cache | 16927      |
| Qcache_total_blocks     | 47042      |
+-------------------------+------------+
8 rows in set (0.00 sec)
这些项的解释如表 1 所示。
表 1. MySQL 查询缓存变量
变量名	说明
Qcache_free_blocks	缓存中相邻内存块的个数。数目大说明可能有碎片。FLUSH QUERY CACHE 会对缓存中的碎片进行整理，从而得到一个空闲块。
Qcache_free_memory	缓存中的空闲内存。
Qcache_hits	每次查询在缓存中命中时就增大。
Qcache_inserts	每次插入一个查询时就增大。命中次数除以插入次数就是不中比率；用 1 减去这个值就是命中率。在上面这个例子中，大约有 87% 的查询都在缓存中命中。
Qcache_lowmem_prunes	缓存出现内存不足并且必须要进行清理以便为更多查询提供空间的次数。这个数字最好长时间来看；如果这个数字在不断增长，就表示可能碎片非常严重，或者内存很少。（上面的 free_blocks 和 free_memory 可以告诉您属于哪种情况）。
Qcache_not_cached	不适合进行缓存的查询的数量，通常是由于这些查询不是 SELECT 语句。
Qcache_queries_in_cache	当前缓存的查询（和响应）的数量。
Qcache_total_blocks	缓存中块的数量。
通常，间隔几秒显示这些变量就可以看出区别，这可以帮助确定缓存是否正在有效地使用。运行 FLUSH STATUS 可以重置一些计数器，如果服务器已经运行了一段时间，这会非常有帮助。
使用非常大的查询缓存，期望可以缓存所有东西，这种想法非常诱人。由于 mysqld 必须要对缓存进行维护，例如当内存变得很低时执行剪除，因此服务器可能会在试图管理缓存时而陷入困境。作为一条规则，如果 FLUSH QUERY CACHE 占用了很长时间，那就说明缓存太大了。
	- 表缓存  table_cache
每个表都可以表示为磁盘上的一个文件，必须先打开，后读取。为了加快从文件中读取数据的过程，mysqld 对这些打开文件进行了缓存，其最大数目由 /etc/mysqld.conf 中的 table_cache 指定。清单 4 给出了显示与打开表有关的活动的方式。
清单 4. 显示打开表的活动
 mysql> SHOW STATUS LIKE 'open%tables';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Open_tables   | 5000  |
| Opened_tables | 195   |
+---------------+-------+
2 rows in set (0.00 sec)
清单 4 说明目前有 5,000 个表是打开的，有 195 个表需要打开，因为现在缓存中已经没有可用文件描述符了（由于统计信息在前面已经清除了，因此可能会存在 5,000 个打开表中只有 195 个打开记录的情况）。如果 Opened_tables 随着重新运行 SHOW STATUS 命令快速增加，就说明缓存命中率不够。如果 Open_tables 比 table_cache 设置小很多，就说明该值太大了（不过有空间可以增长总不是什么坏事）。例如，使用 table_cache = 5000 可以调整表的缓存。
	- 线程缓存 thread_cache
线程来说也有一个缓存。 mysqld 在接收连接时会根据需要生成线程。在一个连接变化很快的繁忙服务器上，对线程进行缓存便于以后使用可以加快最初的连接。
清单 5 显示如何确定是否缓存了足够的线程。
清单 5. 显示线程使用统计信息             
mysql> SHOW STATUS LIKE 'threads%';
+-------------------+--------+
| Variable_name     | Value  |
+-------------------+--------+
| Threads_cached    | 27     |
| Threads_connected | 15     |
| Threads_created   | 838610 |
| Threads_running   | 3      |
+-------------------+--------+
4 rows in set (0.00 sec)
此处重要的值是 Threads_created，每次 mysqld 需要创建一个新线程时，这个值都会增加。如果这个数字在连续执行 SHOW STATUS 命令时快速增加，就应该尝试增大线程缓存。例如，可以在 my.cnf 中使用 thread_cache = 40 来实现此目的
	-  key_buffer
关键字缓冲区保存了 MyISAM 表的索引块。理想情况下，对于这些块的请求应该来自于内存，而不是来自于磁盘。清单 6 显示了如何确定有多少块是从磁盘中读取的，以及有多少块是从内存中读取的。
清单 6. 确定关键字效率         
mysql> show status like '%key_read%';
+-------------------+-----------+
| Variable_name     | Value     |
+-------------------+-----------+
| Key_read_requests | 163554268 |
| Key_reads         | 98247     |
+-------------------+-----------+
2 rows in set (0.00 sec)
 Key_reads 代表命中磁盘的请求个数， Key_read_requests 是总数。命中磁盘的读请求数除以读请求总数就是不中比率 —— 在本例中每 1,000 个请求，大约有 0.6 个没有命中内存。如果每 1,000 个请求中命中磁盘的数目超过 1 个，就应该考虑增大关键字缓冲区了。例如，key_buffer = 384M 会将缓冲区设置为 384MB。
	- 排序缓存 sort_buffer_size
当 MySQL 必须要进行排序时，就会在从磁盘上读取数据时分配一个排序缓冲区来存放这些数据行。如果要排序的数据太大，那么数据就必须保存到磁盘上的临时文件中，并再次进行排序。如果 sort_merge_passes 状态变量很大，这就指示了磁盘的活动情况。清单 8 给出了一些与排序相关的状态计数器信息。
清单 8. 显示排序统计信息    
mysql> SHOW STATUS LIKE "sort%";
+-------------------+---------+
| Variable_name     | Value   |
+-------------------+---------+
| Sort_merge_passes | 1       |
| Sort_range        | 79192   |
| Sort_rows         | 2066532 |
| Sort_scan         | 44006   |
+-------------------+---------+
4 rows in set (0.00 sec)
如果 sort_merge_passes 很大，就表示需要注意 sort_buffer_size。例如， sort_buffer_size = 4M 将排序缓冲区设置为 4MB。
	- 读取表缓存 read_buffer_size
MySQL 也会分配一些内存来读取表。理想情况下，索引提供了足够多的信息，可以只读入所需要的行，但是有时候查询（设计不佳或数据本性使然）需要读取表中大量数据。要理解这种行为，需要知道运行了多少个 SELECT 语句，以及需要读取表中的下一行数据的次数（而不是通过索引直接访问）。实现这种功能的命令如清单 9 所示。
清单 9. 确定表扫描比率          
mysql> SHOW STATUS LIKE "com_select";
+---------------+--------+
| Variable_name | Value  |
+---------------+--------+
| Com_select    | 318243 |
+---------------+--------+
1 row in set (0.00 sec)
mysql> SHOW STATUS LIKE "handler_read_rnd_next";
+-----------------------+-----------+
| Variable_name         | Value     |
+-----------------------+-----------+
| Handler_read_rnd_next | 165959471 |
+-----------------------+-----------+
1 row in set (0.00 sec)
 Handler_read_rnd_next / Com_select 得出了表扫描比率 —— 在本例中是 521:1。如果该值超过 4000，就应该查看 read_buffer_size，例如 read_buffer_size = 4M。如果这个数字超过了 8M，就应该与开发人员讨论一下对这些查询进行调优了！
	
- Engine 引擎 ，innodb，myisam等
- 文件系统和

