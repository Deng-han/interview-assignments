# 短域名服务设计

##题目解析
短域名服务可以拆分为两个接口:
1. 接口请求带上长域名 服务端存储按照key-value形式存储短域名-长域名之间的映射关系,并且返回短域名。
2. 接口请求带上短域名 服务端根据长短域名之间的映射关系,查询到长域名信息 返回。

##设计思路
此问题的本质在于存储长域名和短域名的映射关系，而最简单的映射关系就是“id->url”的这种映射关系，而且id也是唯一的，也就保证了这种映射关系的唯一性。
####设计需考虑的问题
1. 调用方肯定不能直接传一个id过来，因为我们的调用方有一部分直接是用户，可能有人通过id自增的特性，恶意刷我们的接口。这里我们可以对id进行编码，因为url中只直接支持a-zA-Z0-9这总共62个字符，其他的字符则需要编码，我们可以将id通过base62的方式进行编码，从而得到一个字符串，而且base62编码有一个好处就是，十进制的1亿，编码之后也只有5个字符，因而base62编码能够支持大量的短码，而不用过于担心短码不够的问题；

2. 如果直接采用按顺序a-zA-Z0-9的base62编码，可能用户根据这个顺序顺推，而恶意刷我们的短码，造成我们的服务器宕机，因而这里我们在使用base62的时候，需要对原始字符进行乱序，然后依据这个乱序来编码和解码操作；

3. 如何高效的生成全局唯一id，需要考虑到服务可能是多节点集群部署，因此可以增加一个t_sequece的数据库表 存储步长 并且有version作为版本号控制 批量读取id区间到内存。


##思考思路

经过上述的设计方案之后归纳总结几点思路:

1. 短域名的传输需要经过加密传输 防止用户恶意刷内存数据

2. 需要考虑后期的扩展,因此生成id的方式可以采用策略模式来做衍生，分为单节点和多节点模式来读取yml文件进行加载



## 压测思路
1. 因为没有数据库和缓存作为存储,只是使用map作为存储 那么要考虑到多个请求同时对map读写的时候造成的并发问题 例如线程a 请求map 访问 baidu.com  此时没有 而另外一个线程同时在请求map放入此域名
那么 此种case要考虑到
   
2. 多个线程同时在请求同一个资源的时候是否能做到读共享锁,写的时候排他锁 能做到资源最大化的利用 所以需要多个线程同时读取长域名 例如alibaba.com




## 注意

1. swagger采用的是3.0版本 因此访问网页和之前的不同 url:http://localhost:8080/doc.html 或者 http://localhost:8080/swagger-ui.html