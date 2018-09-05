Section 6.2 page 87

实际上还可以把它归结为过度依赖嵌套循环或过度弱化 Query 语句的功能造成性能消耗过多。



Section 6.3 page 89

当MySQL Server 的连接线程接收到 Client 端发送过来的 Query 请求时，会经过一系列的分解 Parse，并进行相应的分析。然后，MySQL 会通过查询优化器模块 （Optimizer）根据该 Query 所涉及的数据表相关统计信息进行计算分析，再得出一个MySQL认为最合理最优化的数据访问方式，也就是常说的 “执行计划”，然后根据所得到的执行计划通过调用存储引擎接口来获取相应数据。最后将存储引擎返回的数据进行相关处理，并以Client端所要求的格式作为结果集返回给Client端的应用程序。



Section 8.2 page 123

一般来说，Query 语句的优化思路和原则主要体现在以下几个方面：

- 优化更需要优化的 Query
- 定位优化对象的性能瓶颈
- 明确优化目标
- 从 Explain 入手
- 多使用 Profile
- 永远用小结果集驱动大的结果集
- 尽可能在索引中完成排序
- 只取自己需要的 column
- 仅仅使用最有效的过滤条件
- 尽可能避免复杂的 join 和子查询。



选择合适的索引的建议：

- 对于单键索引，尽量选择针对当前 Query 过滤性更好的索引
- 在选择组合索引时，当前 Query 中过滤性最好的字段在索引字段顺序中排列越靠前越好
- 在选择组合索引时，尽量选择可以包含当前 Query 的 Where 子句中更多字段的索引
- 尽可能通过分析统计信息和调整 Query 的写法来达到选择合适索引的目的，减少通过使用 Hint 人为控制索引的选择，因为这会使用后期的维护成本增加，同时增加维护所带来的潜在风险。



Section 8.5.2 Join 语句的优化

1 尽可能减少 Join 语句中 Nested Loop 的循环总次数

2 优先优化 Nested Loop 的内层循环

3 保证 Join 语句中被驱动表的 Join 条件字段已经被索引

4 当无法保证被驱动表的 Join 条件字段被索引且内存资源充足时，不要太吝啬 Join Buffer 的设置。



Section 8.6 page 155

在 MySQL中， Order By 的实现有如下两种类型：

- 一种是通过有序索引直接取得有序的数据，这样不用进行任何排序操作即可得到满足客户端要求的有序数据并返回给客户端
- 另外一种须通过MySQL的排序算法将存储引擎中返回的数据进行排序后，再将排序后的数据返回给客户端。



涉及到的优化项

- 加大 max_length_for_sort_data 参数的设置

  决定使用老式排序算法还是改进版的排序算法是通过参数 max_length_for_sort_data 来决定的

- 去掉不必要的返回字段

- 增大 sort_buffer_size 参数设置

  增大 sort_buffer_size 并不是为了让 MySQL 选择改进版的排序算法，而是为了让 MySQL尽量减少在排序过程中对需要排序的数据进行分段，因为分段会造成 MySQL 不得不使用临时表进行交换排序。




Section 8.6 page 165

Group By 的优化方式：

- 尽可能让 MySQL 利用索引来完成 GROUP BY 操作
- 当无法使用索引完成 GROUP BY 时，由于要使用临时表且需要 filesort，所以必须要有足够的 sort_buffer_size 供 MySQL 排序时使用，而且尽量不要进行大结果集的 GROUP BY 操作，因为如果超过系统设置的临时表大小就会出现将临时表数据复制到磁盘上面再进行操作的情况。



Section 9.1 page 170

Schema 优化方法：

- 适度冗余 - 让 Query 尽量减少 Join
- 大字段垂直拆分
- 大表水平分拆

Section 9.2 page 176

合适数据类型

	优化数据类型提高性能的主要原理是：

- 通过选用更小的数据类型减小存储空间，使查询相同数据需要的IO资源降低
- 通过合适的数据类型加速数据的比较。

Section 10.3 page 193

Query Cache 真的是 “葵花宝典” 吗？

MySQL  的 Query Cache 的实现原理实际上并不是特别复杂，简单来说就是将客户端请求的 Query 语句 （仅限于 SELECT 类型的 Query）通过一定的 Hash 算法进行一个计算，得到一个 Hash 值，存放在一个 Hash 桶中。同时将该 Query 的结果集 (Result Set) 也存放在一个内存 Cache 中。存放在 Query hash 值的链表中每一个 hash 值所在节点的同时，还存放了该 Query 所对应的 Result Set 的 Cache 所在的内存地址，以及该 Query 涉及的所有 Table 标识等一些其他信息，系统接收到一个 Select 类型的 Query 时，首先计算出其 hash 值，然后通过该 hash 值到 Query Cache 中去匹配，如果找到了完全相同的 Query，则直接将之前所缓存的 Result Set 返回给客户端，完全不需要进行后面的任何步骤即可完成这次请求。而后端的任何一个表的任何一条数据发生变化之后，也会通知 Query Cache，需要将所有跟该 Table 相关的 Query的 Cache 全部失效。

