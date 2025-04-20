# Nginx 配置解析

## nginx 配置系统概述

Nginx 的配置系统由一个主配置文件 nginx.conf 和其他一些辅助配置文件构成。这些配置文件均是纯文本文件，全部位于 Nginx 安装目录下的 conf 目录下。

## nginx 主配置文件

### nginx 主配置文件结构

```nginx
...              #nginx全局块

events {         #events块
   ...    #events块
}

http {     #http块

    ...   #http全局块
    
    server {       #server块
        ...       #server全局块
        location [PATTERN] {  #location块
            ...  #location块
        }
        
        location [PATTERN] {
            ...
        }
    }
    
    server {
      ...
    }
    ...     #http全局块
}
```

### nginx 主配置文件默认配置

简化之后的 nginx.conf：

```nginx

worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
    }

}
```

## nginx include 指令

include 可以用在任何地方，前提是 include 的文件自身语法正确。

主配置中 include 的使用方式如下：

```nginx
# 绝对路径
include /etc/conf/nginx.conf;

# 相对路径，相对路径以nginx.conf为基准
include port/80.conf;

# 通配符
include *.conf;
```

## nginx 配置指令详解

### 全局块配置

全局块是默认配置文件 **从开始到 events 块之间的一部分内容**，主要设置一些影响 Nginx 服务器整体运行的配置指令，因此，这些指令的作用域是 Nginx 服务器全局。

通常包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数、Nginx 进程 PID 存放路径、日志的存放路径和类型以及配置文件引入等。

#### main.conf(主配置)

```nginx
# 定义nginx worker进程运行的用户和用户组，默认为nobody nobody。
# user指令在Windows上不生效，如果你制定具体用户和用户组会报小面警告
# user [user] [group]
user itxh itxh;

# 指定工作线程数，可以制定具体的进程数，也可使用自动模式，这个指令只能在全局块配置
# worker_processes number | auto；
# 如果指定进程数的话，建议设置为等于CPU总核心数。默认为1
# 列子：指定4个工作线程，这种情况下会生成一个master进程和4个worker进程
worker_processes 4;

# 指定错误日志的路径和日志级别，此指令可以在全局块、http块、server块以及location块中配置。
# 其中debug级别的日志需要编译时使用--with-debug开启debug开关
# 日志级别依次为：debug | info | notice | warn | error | crit | alert | emerg

# error_log [path] [debug | info | notice | warn | error | crit | alert | emerg] 
# error_log  logs/error.log  notice;
# error_log  logs/error.log  info;
error_log /var/log/nginx/error.log info;

# 进程文件，记录master process的pid号
# 在Linux/Unix下，很多程序比如nginx会启动多个进程，而发信号的时候需要知道要向哪个进程发信号。
# 不同的进程有不同的pid（process id）。将pid写进文件可以使得在发信号时比较简单。
# 比如要重新加载nginx的配置可以这样写：kill -HUP `cat /path/to/nginx.pid`
pid /var/run/nginx.pid;

#一个worker进程打开的最多文件描述符数目,理论值应该是最多打开文件数（系统的值ulimit -n）与nginx进程数相除
# 但是nginx分配请求并不均匀,所以建议与ulimit -n的值保持一致.
worker_rlimit_nofile 65535;
```

### events 块配置

events 块涉及的指令 **主要影响 Nginx 服务器与用户的网络连接**。常用到的设置包括是否开启对多 worker process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型处理连接请求，每个 worker process 可以同时支持的最大连接数等。

这一部分的指令对 Nginx 服务器的性能影响较大，在实际配置中应该根据实际情况灵活调整。

#### events_core.conf(核心配置)

```nginx

# 当某一时刻只有一个网络连接到来时，多个睡眠进程会被同时叫醒，但只有一个进程可获得连接。如果每次唤醒的进程数目太多，会影响一部分系统   性能。在Nginx服务器的多进程下，就有可能出现这样的问题。
# 开启的时候，将会对多个Nginx进程接收连接进行序列化，防止多个进程对连接的争抢
# 默认是开启状态，只能在events块中进行配置
accept_mutex on | off; #设置网路连接序列化锁，防止惊群现象发生，默认为on。

# 如果multi_accept被禁止了，nginx一个工作进程只能同时接受一个新的连接。否则，一个工作进程可以同时接受所有的新连接。 
# 如果nginx使用kqueue连接方法，那么这条指令会被忽略，因为这个方法会报告在等待被接受的新连接的数量。
# 默认是off状态，只能在event块配置
multi_accept on | off;  #设置一个进程是否同时接受多个网络连接，默认为off

# 指定使用哪种网络IO模型
# method可选择的内容有：select、poll、kqueue、epoll、rtsig、/dev/poll以及eventport，一般操作系统不是支持上面所有模型的。
# epoll模型是Linux 2.6以上版本内核中的高性能网络I/O模型,如果跑在FreeBSD上面,就用kqueue模型.
# 只能在events块中进行配置
# use [method]
use epoll;

# 设置允许每一个worker process同时开启的最大连接数，当每个工作进程接受的连接数超过这个值时将不再接收连接
# 当所有的工作进程都接收满时，连接进入logback，logback满后连接被拒绝
# 只能在events块中进行配置
# 注意：这个值不能超过超过系统支持打开的最大文件数，也不能超过单个进程支持打开的最大文件数
worker_connections  1024; #单个进程最大连接数，默认为512（最大连接数=连接数*进程数）
```

#### events_other.conf(其他配置)

```nginx
# 客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求头的大小不会超过1k，不过由于一般系统分页都要大于1k，所   以这里设置为分页大小。
# 分页大小可以用命令getconf PAGESIZE 取得。
# [root@web001 ~]# getconf PAGESIZE
# 4096
# 但也有client_header_buffer_size超过4k的情况，但是client_header_buffer_size该值必须设置为“系统分页大小”的整倍数。
client_header_buffer_size 4k;

# 打开文件指定缓存，默认是没有启用的
# max：指定缓存数量，建议和打开文件数一致
# inactive：是指经过多长时间文件没被请求后删除缓存。
open_file_cache max=65535 inactive=60s;

# 多长时间检查一次open_file_cache缓存的有效信息。
# 语法:open_file_cache_valid time 
# 默认值:open_file_cache_valid 60 
# 使用字段:http, server, location
open_file_cache_valid 80s;

# open_file_cache指令中的inactive参数时间内文件的最少使用次数，如果超过这个数字，文件描述符一直是在缓存中打开的，如上例，如果有   一个文件在inactive时间内一次没被使用，它将被移除。
# 语法:open_file_cache_min_uses number 默认值:open_file_cache_min_uses 1 使用字段:http, server, location  这个指令   指定了在open_file_cache指令无效的参数中一定的时间范围内可以使用的最小文件数,如果使用更大的值,文件描述符在cache中总是打开状态.
open_file_cache_min_uses 1;

# 指定是否在搜索一个文件时记录cache错误.
# 语法:open_file_cache_errors on | off 
# 默认值:open_file_cache_errors off 
# 使用字段:http, server, location 
open_file_cache_errors on;
```

### http 全局块配置

http 块是 Nginx 服务器配置中的重要部分，**代理、缓存和日志定义等绝大多数的功能和第三方模块的配置** 都可以放在这个模块中。

http 块中可以包含自己的全局块，也可以包含 server 块，server 块中又可以进一步包含 location 块。在这里我们使用“http 全局块”来表示 http 中自己的全局块，即 http 块中不包含在 server 块中的部分。

http 全局块中配置的指令包括文件引入、MIME-Type 定义、日志自定义、是否使用 sendfile 传输文件、连接超时时间、单连接请求数上限等。

#### http_common.conf(公共配置)

```nginx
# 常用的浏览器中，可以显示的内容有HTML、XML、GIF及Flash等种类繁多的文本、媒体等资源，浏览器为区分这些资源，需要使用MIME Type。
# 换言之，MIME Type是网络资源的媒体类型。Nginx服务器作为Web服务器，必须能够识别前端请求的资源类型。
# include指令，用于包含其他的配置文件，可以放在配置文件的任何地方，但是要注意你包含进来的配置文件一定符合配置规范
# 比如说你include进来的配置是worker_processes指令的配置，而你将这个指令包含到了http块中，这肯定是不行的，上面已经介绍过         worker_processes指令只能在全局块中。
# 下面的指令将mime.types包含进来，mime.types和ngin.conf同级目录，不同级的话需要指定具体路径
include mime.types; #文件扩展名与文件类型映射表

# 配置默认类型，如果不加此指令，默认值为text/plain。
# 此指令还可以在http块、server块或者location块中进行配置。
default_type application/octet-stream; 
#charset utf-8; #默认编码

# 配置连接超时时间,此指令可以在[http块、server块或location块]中配置。
# 与用户建立会话连接后，Nginx服务器可以保持这些连接打开一段时间
# timeout，服务器端对连接的保持时间。默认值为75s;
# header_timeout，可选项，在应答报文头部的Keep-Alive域设置超时时间：“Keep-Alive:timeout= header_timeout”。报文中的这个指   令可以被Mozilla或者Konqueror识别。
# keepalive_timeout timeout [header_timeout]
# 下面配置的含义是，在服务器端保持连接的时间设置为120 s，发给用户端的应答报文头部中Keep-Alive域的超时时间设置为100 s。
keepalive_timeout 120s 100s ;

# 配置单连接请求数上限，此指令可以在http块、server块或location块中配置。
# Nginx服务器端和用户端建立会话连接后，用户端通过此连接发送请求。
# 指令keepalive_requests用于限制用户通过某一连接向Nginx服务器发送请求的次数。默认是100
# keepalive_requests number;
keepalive_requests 100;

# 保存服务器名字的hash表是由指令server_names_hash_max_size 和server_names_hash_bucket_size所控制的。
# 参数hash bucket size总是等于hash表的大小，并且是一路处理器缓存大小的倍数。在减少了在内存中的存取次数后，使在处理器中加速查找     hash表键值成为可能。如果hash bucket size等于一路处理器缓存的大小，那么在查找键的时候，最坏的情况下在内存中查找的次数为2。第一   次是确定存储单元的地址，第二次是在存储单元中查找键值。因此，如果Nginx给出需要增大hash max size 或 hash bucket size的提示，   那么首要的是增大前一个参数的大小.
server_names_hash_bucket_size 128; #服务器名字的hash表大小

error_page 404 https://www.baidu.com; #错误页
```

#### access_log.conf(访问日志)

```nginx
# access_log配置，此指令可以在[http块、server块或者location块]中进行设置
# 在全局块中，我们介绍过errer_log指令，其用于配置Nginx进程运行时的日志存放和级别
# 此处所指的日志与常规的不同，它是指记录Nginx服务器提供服务过程应答前端请求的日志
# access_log path [format [buffer=size]]
# 如果你要关闭access_log,你可以使用下面的命令
access_log off;

# log_format指令，用于定义日志格式，此指令只能在http块中进行配置
# 日志格式设定
# $remote_addr与$http_x_forwarded_for用以记录客户端的ip地址；
# $remote_user：用来记录客户端用户名称；
# $time_local： 用来记录访问时间与时区；
# $request： 用来记录请求的url与http协议；
# $status： 用来记录请求状态；成功是200，
# $body_bytes_sent ：记录发送给客户端文件主体内容大小；
# $http_referer：用来记录从那个页面链接访问过来的；
# $http_user_agent：记录客户浏览器的相关信息；
# 通常web服务器放在反向代理的后面，这样就不能获取到客户的IP地址了，通过$remote_add拿到的IP地址是反向代理服务器的iP地址。反向代理   服务器在转发请求的http头信息中，可以增加x_forwarded_for信息，用以记录原有客户端的IP地址和原来客户端的请求的服务器地址。
log_format  main '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';


# 定义了上面的日志格式后，可以以下面的形式使用日志
#access_log  logs/access.log  main;
access_log  /usr/local/nginx/logs/host.access.log  main;
access_log  /usr/local/nginx/logs/host.access.404.log  log404;   
```

#### sendfile.conf(文件传输)

```nginx
# 开启关闭sendfile方式（zero copy 方式）传输文件，可以在[http块、server块或者location块]中进行配置
# sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为     off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
sendfile on;

# 设置sendfile最大数据量,此指令可以在[http块、server块或location块]中配置
# 每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
# sendfile_max_chunk size;
# 其中，size值如果大于0，Nginx进程的每个worker process每次调用sendfile()传输的数据量最大不能超过这个值(这里是128k，所以每次不   能超过128k)；如果设置为0，则无限制。默认值为0。
sendfile_max_chunk 128k;

# 此选项仅在使用sendfile的时候使用
# 可以通过设置套接字的 TCP_NODELAY = on 选项来完成，这样就禁用了Nagle 算法。（不需要等待0.2s）
tcp_nodelay on; # 防止网络阻塞
# 在 nginx 中，tcp_nopush 配置和 tcp_nodelay "互斥"。它可以配置一次发送数据的包大小。也就是说，它不是按时间累计  0.2 秒后发送   包，而是当包累计到一定大小后就发送。在 nginx 中，tcp_nopush 必须和 sendfile 搭配使用。
tcp_nopush on; # 防止网络阻塞
```

#### gzip.conf(压缩)

```nginx
# gzip模块设置
# 开启gzip压缩输出
gzip on; 
# 最小压缩文件大小 ，允许压缩的页面的最小字节数,页面字节数从header偷得content-length中获取.默认是0,不管页面多大都进行压缩.建议设置成大于1k的字节数,小于1k可能会越压越大
gzip_min_length 1k;
# 压缩缓冲区，表示申请4个单位为16k的内存作为压缩结果流缓存,默认值是申请与原始数据大小相同的内存空间来存储gzip压缩结果
gzip_buffers 4 16k; 
# 压缩版本（默认1.1,目前大部分浏览器已经支持gzip解压.前端如果是squid2.5请使用1.0）
gzip_http_version 1.1; 
# 压缩等级.1压缩比最小,处理速度快.9压缩比最大,比较消耗cpu资源,处理速度最慢,但是因为压缩比最大,所以包最小,传输速度快
gzip_comp_level 2; 
# 压缩类型,默认就已经包含text/html,所以下面就不用再写了,写上去也不会有问题,但是会有一个warn.
gzip_types text/plain application/x-javascript text/css application/xml;
# 选项可以让前端的缓存服务器缓存经过gzip压缩的页面.例如:用squid缓存经过nginx压缩的数据
gzip_vary on;
```

#### send_large_file.conf(大文件上传)

用 nginx 作代理服务器，上传大文件时（测试上传 50m 的文件），提示上传超时或文件过大。原因可能是：

1. nginx 对上传文件大小有限制，而且默认是 **1M**。
2. 上传文件很大导致上传超时超时，还要适当调整上传超时时间。

```nginx
# 限制请求体的大小，若超过所设定的大小，返回413错误。
client_max_body_size 50m; #文件大小限制，默认1m

# 读取请求头的超时时间（单位分钟），若超过所设定的大小，返回408错误。
client_header_timeout 1m;

# 读取请求实体的超时时间，若超过所设定的大小，返回413错误。
client_body_timeout 1m;

# http请求无法立即被容器(tomcat, netty等)处理，被放在nginx的待处理池中等待被处理。
# 此参数为等待的最长时间，默认为60秒，官方推荐最长不要超过75秒。
proxy_connect_timeout 60s;

# http请求被容器(tomcat, netty等)处理后，nginx会等待处理结果，也就是容器返回的response。
# 此参数即为服务器响应时间，默认60秒。
proxy_read_timeout 1m;

# http请求被服务器处理完后，把数据传返回给Nginx的用时，默认60秒。
proxy_send_timeout 1m;
```

#### header_buffer.conf(请求头部)

```nginx
# nginx默认会用client_header_buffer_size这个buffer来读取header值
# 如果(请求行+请求头)的大小如果没超过client_header_buffer_size，放行请求。
# 如果(请求行+请求头)的大小如果超过client_header_buffer_size，则以large_client_header_buffers配置为准

# 请求行+请求头的标准大小为1k
client_header_buffer_size 1k;

# 请求行+请求头的最大大小为4 * 8k，4个buffer，每个buffer大小 8k
large_client_header_buffers 4 8k;
```

#### autoindex.conf(目录访问)

```nginx
# 开启目录列表访问,默认关闭.
autoindex on; # 显示目录
# 显示文件大小 默认为on,显示出文件的确切大小,单位是bytes 改为off后,显示出文件的大概大小,单位是kB或者MB或者GB
autoindex_exact_size on;
# 显示文件时间 默认为off,显示的文件时间为GMT时间 改为on后,显示的文件时间为文件的服务器时间
autoindex_localtime on; 
```

#### upstream.conf(负载均衡)

```nginx
# 负载均衡配置
upstream www.itxh.com {

# upstream的负载均衡，weight是权重，可以根据机器配置定义权重。weigth参数表示权值，权值越高被分配到的几率越大。
server 192.168.80.121:80 weight=3;
server 192.168.80.122:80 weight=2;
server 192.168.80.123:80 weight=3;

# nginx的upstream目前支持4种方式的分配
# 1、轮询（默认）
# 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
# 2、weight
# 指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
# 例如：
# upstream bakend {
#     server 192.168.0.14 weight=10;
#     server 192.168.0.15 weight=10;
# }
# 2、ip_hash
# 每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
# 例如：
# upstream bakend {
#     ip_hash;
#     server 192.168.0.14:88;
#     server 192.168.0.15:80;
# }
# 3、fair（第三方）
# 按后端服务器的响应时间来分配请求，响应时间短的优先分配。
# upstream backend {
#     server server1;
#     server server2;
#     fair;
# }
# 4、url_hash（第三方）
# 按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
# 例：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法
# upstream backend {
#     server squid1:3128;
#     server squid2:3128;
#     hash $request_uri;
#     hash_method crc32;
# }

# tips:
# upstream bakend{#定义负载均衡设备的Ip及设备状态}{
#    ip_hash;
#    server 127.0.0.1:9090 down;
#    server 127.0.0.1:8080 weight=2;
#    server 127.0.0.1:6060;
#    server 127.0.0.1:7070 backup;
# }
# 在需要使用负载均衡的server中增加 proxy_pass http:

# 每个设备的状态设置为:
# 1.down表示单前的server暂时不参与负载
# 2.weight为weight越大，负载的权重就越大。
# 3.max_fails：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream模块定义的错误
# 4.fail_timeout:max_fails次失败后，暂停的时间。
# 5.backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

# nginx支持同时设置多组的负载均衡，用来给不用的server来使用。
# client_body_in_file_only设置为On 可以讲client post过来的数据记录到文件中用来做debug
# client_body_temp_path设置记录文件的目录 可以设置最多3层目录
# location对URL进行匹配.可以进行重定向或者进行新的代理 负载均衡
}
```

### server 块配置

server 块和“虚拟主机”的概念有密切联系。

虚拟主机，又称虚拟服务器、主机空间或是网页空间，它是一种技术。该技术是为了节省互联网服务器硬件成本而出现的。这里的“主机”或“空间”是由实体的服务器延伸而来，硬件系统可以基于服务器群，或者单个服务器等。

虚拟主机技术主要应用于 HTTP、FTP 及 EMAIL 等多项服务，将一台服务器的某项或者全部服务内容逻辑划分为多个服务单位，对外表现为多个服务器，从而充分利用服务器硬件资源。从用户角度来看，一台虚拟主机和一台独立的硬件主机是完全一样的。

在使用 Nginx 服务器提供 Web 服务时，利用虚拟主机的技术就可以避免为每一个要运行的网站提供单独的 Nginx 服务器，也无需为每个网站对应运行一组 Nginx 进程。虚拟主机技术使得 Nginx 服务器可以在同一台服务器上只运行一组 Nginx 进程，就可以运行多个网站。

每一个 http 块都可以包含多个 server 块，而每个 server 块就相当于一台虚拟主机，它内部可有多台主机联合提供服务，一起对外提供在逻辑上关系密切的一组服务（或网站）。

和 http 块相同，server 块也可以包含自己的全局块，同时可以包含多个 location 块。在 server 全局块中，最常见的两个配置项是本虚拟主机的监听配置和本虚拟主机的名称或 IP 配置。

#### server_global.conf

```nginx
# server块中最重要的指令就是listen指令，这个指令有三种配置语法。
# 这个指令默认的配置值是：listen *:80 | *:8000；只能在server块种配置这个指令。
# 监听端口
listen 80;
# listen 127.0.0.1:8000;  #只监听来自127.0.0.1这个IP，请求8000端口的请求
# listen 127.0.0.1; #只监听来自127.0.0.1这个IP，请求80端口的请求（不指定端口，默认80）
# listen 8000; #监听来自所有IP，请求8000端口的请求
# listen *:8000; #和上面效果一样
# listen localhost:8000; #和第一种效果一致


#域名可以有多个，用空格隔开
server_name www.itxh.com;

# 如果匹配不到location中的文件，则显示index指定的文件
index index.html index.htm index.php;

# 当匹配到location时，该location定位到的磁盘根路径（和URL中的根/对应）
root /data/www/itxh;
```

#### static_locations.conf

```nginx
# 图片缓存时间设置
location ~ .*.(gif|jpg|jpeg|png|bmp|swf)${
    # 有效期 10天
    expires 10d;
}

#JS和CSS缓存时间设置
location ~ .*.(js|css)?$ {
    # 有效期 1小时
    expires 1h;
}

#所有静态文件由nginx直接读取不经过tomcat或resin
location ~ .*.(htm|html|gif|jpg|jpeg|png|bmp|swf|ioc|rar|zip|txt|flv|mid|doc|ppt|pdf|xls|mp3|wma)$ {
   # 有效期 15天
   expires 15d; 
}
```

#### nginx_status_location.conf(Nginx 状态地址)

```nginx
# 设定查看Nginx状态的地址.StubStatus模块能够获取Nginx自上次启动以来的工作状态，此模块非核心模块，需要在Nginx编译安装时手工指定才   能使用
# 设定查看Nginx状态的地址
location /NginxStatus {
    stub_status on;
    access_log on;
    auth_basic "NginxStatus";
    auth_basic_user_file confpasswd;
    # htpasswd文件的内容可以用apache提供的htpasswd工具来产生。
}   
```

#### https_server.conf

```nginx
server {
    # 监听端口 HTTPS
    listen 443 ssl;
    server_name https://www.baidu.com;
    root /data/www/;
    index index.html index.htm index.php;

    # 配置域名证书
    ssl_certificate      C:\WebServer\Certs\certificate.crt;
    ssl_certificate_key  C:\WebServer\Certs\private.key;
    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;
    ssl_protocols SSLv2 SSLv3 TLSv1;
    ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
    ssl_prefer_server_ciphers  on;



    # 配置地址拦截转发，解决跨域验证问题
    location /oauth/ {
        proxy_pass https://localhost:13580/oauth/;
        proxy_set_header HOST $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### location 块配置

**每个 server 块中可以包含多个 location 块。** 在整个 Nginx 配置文档中起着重要的作用，而且 Nginx 服务器在许多功能上的灵活性往往在 location 指令的配置中体现出来。

**location 块的主要作用是：** 基于 Nginx 服务器接收到的请求字符串（例如， `server_name/uri-string`），对除虚拟主机名称（也可以是 IP 别名，后文有详细阐述）之外的字符串（前例中 `/uri-string` 部分）进行匹配，对特定的请求进行处理。

在 Nginx 的官方文档中定义的 location 的语法结构为：

```nginx
location [ = | ~ | ~* | ^~ ] uri { ... }
```

- uri 变量 **：待匹配的请求字符串**，可以是不含正则表达的字符串，如/myserver 等；也可以是包含有正则表达的字符串，如 .jpg$（表示以.jpg 结尾的 URL）等。为了方便，我们约定，不含正则表达的 uri 称为“标准 uri”，使用正则表达式的 uri 称为“正则 uri”。

- 方括号里的部分：**可选项，用来改变请求字符串与 uri 的匹配方式**。

在不添加此选项时，Nginx 服务器首先在 server 块的多个 location 块中搜索是否有标准 uri 和请求字符串匹配，如果有多个可以匹配，就记录匹配度最高的一个。然后，服务器再用 location 块中的正则 uri 和请求字符串匹配，当第一个正则 uri 匹配成功，结束搜索，并使用这个 location 块处理此请求；如果正则匹配全部失败，就使用刚才记录的匹配度最高的 location 块处理此请求。

**可选项中各个标识的含义：**

- `=`：用于标准 uri 前，要求请求字符串与 uri 严格匹配。如果已经匹配成功，就停止继续向下搜索并立即处理此请求。
- `^～`：用于标准 uri 前，要求 Nginx 服务器找到标识 uri 和请求字符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location 块中的正则 uri 和请求字符串做匹配。
- `～`：用于表示 uri 包含正则表达式，并且区分大小写。
- `～*`：用于表示 uri 包含正则表达式，并且不区分大小写。注意如果 uri 包含正则表达式，就必须要使用 `～` 或者 `～*` 标识。

在浏览器传送 URI 时对一部分字符进行 URL 编码，比如空格被编码为“%20”，问号被编码为“%3f”等。“～”有一个特点是，它对 uri 中的这些符号将会进行编码处理。比如，如果 location 块收到的 URI 为“/html/%20/data”，则当 Nginx 服务器搜索到配置为“～ /html/ /data”的 location 时，可以匹配成功。

#### proxy.conf

```nginx
proxy_redirect off;

proxy_set_header X-Real-IP $remote_addr;      
# 后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
# 以下是一些反向代理的配置，可选。
proxy_set_header Host $host;

# 允许客户端请求的最大单文件字节数
client_max_body_size 10m;

# 缓冲区代理缓冲用户端请求的最大字节数，
# 如果把它设置为比较大的数值，例如256k，那么，无论使用firefox还是IE浏览器，来提交任意小于256k的图片，都很正常。如果注释该指令，使   用默认的client_body_buffer_size设置，也就是操作系统页面大小的两倍，8k或者16k，问题就出现了。
# 无论使用firefox4.0还是IE8.0，提交一个比较大，200k左右的图片，都返回500 Internal Server Error错误
client_body_buffer_size 128k;

# 表示使nginx阻止HTTP应答代码为400或者更高的应答。
proxy_intercept_errors on;

# 后端服务器连接的超时时间_发起握手等候响应超时时间
# nginx跟后端服务器连接超时时间(代理连接超时)
proxy_connect_timeout 90;

# 后端服务器数据回传时间(代理发送超时)
# 后端服务器数据回传时间_就是在规定时间之内后端服务器必须传完所有的数据
proxy_send_timeout 90;

# 连接成功后，后端服务器响应时间(代理接收超时)
# 连接成功后_等候后端服务器响应时间_其实已经进入后端的排队之中等候处理（也可以说是后端服务器处理请求的时间）
proxy_read_timeout 90;

# 设置代理服务器（nginx）保存用户头信息的缓冲区大小
# 设置从被代理服务器读取的第一部分应答的缓冲区大小，通常情况下这部分应答中包含一个小的应答头，默认情况下这个值的大小为指令           proxy_buffers中指定的一个缓冲区的大小，不过可以将其设置为更小
proxy_buffer_size 4k;

# proxy_buffers缓冲区，网页平均在32k以下的设置
# 设置用于读取应答（来自被代理服务器）的缓冲区数目和大小，默认情况也为分页大小，根据操作系统的不同可能是4k或者8k
proxy_buffers 4 32k;

# 高负荷下缓冲大小（proxy_buffers*2）
proxy_busy_buffers_size 64k;

# 设置在写入proxy_temp_path时数据的大小，预防一个工作进程在传递文件时阻塞太长
# 设定缓存文件夹大小，大于这个值，将从upstream服务器传
proxy_temp_file_write_size 64k;
```



## nginx 变量

### 自定义变量

由于 Nginx 配置文件是 perl 脚本，所以其是可以使用如下方式自定义变量的。

- 定义变量（在server块内定义变量）

  ```nginx
   # 变量值为字符串或者数字
  set $变量名 变量值; 
  ```

- 使用变量

  ```nginx
  location / {
      root $变量名;
  }
  ```

### 内置变量

- `$args` 请求中的参数;
- `$binary_remote_addr` 远程地址的二进制表示
- `$body_bytes_sent `记录发送给客户端文件主体内容大小，也就是已发送的消息体字节数；
- `$content_length` HTTP 请求信息里的"Content-Length"
- `$content_type` HTTP请求信息里的"Content-Type"
- `$document_root `针对当前请求的根路径设置值
- `$document_uri` 与uri 相同
- `$host` 请求信息中的"Host"，如果请求中没有 Host 行，则等于设置的服务器名;
- `$http_cookie cookie` 信息
- `$http_referer`用来记录从那个页面链接访问过来的，也就是来源地址；
- `$http_user_agent` 记录客户端浏览器的相关信息；用户所使用的代理，一般为浏览器。
- `$http_via` 最后一个访问服务器的 Ip 地址
- `$http_x_forwarded_for`获取客户端浏览器的 IP。若当前 Nginx 是反代服务器，则此变量获取到的值为杠(-)。若当前 Nginx 是静态代理服务器，则此变量获取到的是客户端的IP 地址。
- `$limit_rate` 对连接速率的限制
- `$remote_addr` 客户端地址，获取访问者的 IP 地址。若当前 Nginx 是反代服务器，则此变量获取到的就是客户端的 IP 地址；若当前 Nginx 是静态代理服务器，则此变量获取到的是反代服务器的 IP 地址。
- `$remote_port `客户端端口号
- `$remote_user` 获取访问者的用户名。
- `$request`  获取请求的相关信息，包含请求方式、请求的 URI，及访问协议。
- `$request_body` 用户请求主体
- `$request_body_file` 发往后端的本地文件名称
- `$request_filename` 当前请求的文件路径名
- `$request_method` 请求的方法，比如"GET"、"POST"等
- `$request_uri` 请求的 URI，带参数
- `$server_addr` 服务器地址，如果没有用 listen 指明服务器地址，使用这个变量将发起一次系统调用以取得地址(造成资源浪费)
- `$server_name` 请求到达的服务器名
- `$server_port` 请求到达的服务器端口号
- `$server_protocol` 请求的协议版本，"HTTP/1.0"或"HTTP/1.1"
- `$uri` 请求的 URI，可能和最初的值有不同，比如经过重定向之类的
- `$status`  用来记录请求状态；后端服务器向其返回的状态码，例如 200。
- $`time_local`  用来记录访问时间与时区；

