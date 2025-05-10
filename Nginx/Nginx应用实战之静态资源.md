# Nginx 应用实战之静态资源



nginx 静态代理是指，将所有的静态资源，例如，css、js、html、jpg 等资源存放到 nginx 服务器，而不存放在应用服务器 Tomcat 中。当客户端发出的请求是对这些静态资源的请求时，nginx 直接将这些静态资源响应给客户端，而无需提交给应用服务器处理，这样就减轻了应用服务器的压力。

## location 常用配置指令

### root

**根路径配置（请求 URI 根路径/映射配置）**，用于访问文件系统，在匹配到 location 配置的 URL 路径后，指向 root 配置的路径，并把请求路径附加到其后，如：

```nginx
# http://ip:port/test/1.jpg
# /test/1.jpg
location /test/  {
    root  /usr/local/;
    #index index.html;
}
```

请求 `http://ip:port/test/1.jpg`，将会返回文件/usr/local/test/1.jpg。

### alias

**别名配置**，用于访问文件系统，在匹配到 location 配置的 URL 路径后，指向 alias 配置的路径，如：

```nginx
# http://ip:port/test/1.jpg
location /test/  {
    alias  /usr/local/;
}
```

请求 `http://ip:port/test/1.jpg`，将会返回文件/usr/local/1.jpg。

如果 alias 配置在正则匹配的 location 内，则 **正则表达式中必须包含捕获语句（也就是括号()），而且 alias 配置中也要引用这些捕获值**。如：

```nginx
location ~* /img/(.+\\.(gif|png|jpeg)){
    alias  /usr/local/images/$1;
}
```

请求中只要能匹配到正则，比如/img/flower.png  或者  /resource/img/flower.png，都会转换为请求/usr/local/images/flower.png。

### proxy_pass

**反向代理配置**，用于代理请求，适用于前后端负载分离或多台机器、服务器负载分离的场景，在匹配到 location 配置的 URL 路径后，转发请求到 proxy_pass 配置的 URL，是否会附加 location 配置路径与 proxy_pass 配置的路径后是否有 "/" 有关，有 "/" 则不附加，如：

```nginx
location /test/  {        
    proxy_pass  http://127.0.0.1:8080/;
}
```

请求/test/1.jpg，将会被 nginx 转发请求到 http://127.0.0.1:8080/1.jpg（未附加/test/路径）。

## 扩展名拦截

### 修改配置文件

```nginx
server {
    listen 80;
    server_name localhost;
    
    location / {
        # 相对路径是相对于nginx的安装目录，比如/user/local/nginx，那么完整的目录就是：/user/local/nginx/html
        root html;
        index index.html;
    }
    
    location ~* .*\.(css|js|html|jpg|png)$ {
        root /opt/statics;
    }
}
```

### 创建目录

在/opt 目录中创建 statics 目录；在/opt/statics 目录中创建 css、js、images 目录。

![image-20250510144606941](assets/image-20250510144606941.png)

### 上传图片

向/opt/statics/images 目录中上传一个图片 car_su7.png。

### 重启 Nginx

```nginx
nginx -s reload
```

### 浏览器访问

http://192.168.78.100/images/car_su7.png

![image-20250510145710150](assets/image-20250510145710150.png)

## 静态资源优化配置

### sendfile

默认情况下，nginx 会自行处理文件传输，并在发送之前将文件复制到缓冲区中。

启用 sendfile 指令跳过了将数据复制到缓冲区的步骤，并允许将数据从一个文件描述符直接复制到另一个文件描述符。同时，为了防止一个快速连接完全占用工作进程，也可以使用 `sendfile_max_chunk` 指令限制单个 sendfile()调用中传输的数据量。

#### 语法

```nginx
# 默认 off;
sendfile on | off;
```

#### 可配置段

```nginx
http，server，location，if in location
```

#### 配置示例

```nginx
location /mp3 {
    #...
    sendfile    on;
    sendfile_max_chunk  1m;
    #...
}
```

### tcp_nopush

将 `tcp_nopush` 指令与 `sendfile on` 指令一起使用，可以使 nginx 在 sendfile()获取数据块之后立即在一个数据包中发送 HTTP 响应头。**即在 sendfile 开启情况下，提高网络包的 "传输效率"。**

#### 语法

```nginx
# 默认 off;
tcp_nopush on | off;
```

#### 可配置段

```nginx
http, server, location
```

#### 配置示例

```nginx
location /mp3 {
    #...
    sendfile   on;
    tcp_nopush on;
    #...
}
```

## 页面压缩协议

### 常见的压缩协议

浏览器中最常见的压缩算法有：

- `deflate`：是一种过时的压缩算法，是 huffman 编码的一种加强。
- `gzip`：是目前大多数浏览器都支持的一种压缩算法，是对 deflate 的改进（常用）。
- `sdch`：谷歌开发的一种压缩算法，一种全新的压缩思路。deflate 与 gzip 的的压缩思想是，修改传输数据的编码格式以达到减少体量的目的，其最终传输的数据并没有减少。而 sdch 压缩算法的思想是，让冗余的数据仅出现一次，其最终传输的数据减少了。
- `Zopfli`：谷歌开发的一种压缩算法，Deflate 压缩算法的改进。比标准的 gzip -9 要小 3%-8%，但压缩用时是 gzip -9 的 80 多倍。
- `br`：即 Brotli，谷歌开发的一种压缩算法，是一种全新的数据格式。与 Zopfli 相比，压缩率能够降低 20%-26%。Brotli -1 有着与 Gzip -9 相近的压缩比和更快的压缩解压速度。

### gzip 压缩常用配置

```nginx
# 开启 gzip 压缩，默认为 off。
gzip on;
# 当返回内容大于此值（默认gzip_min_length 20;单位为字节）时才会使用gzip进行压缩，当值为0时，所有页面都进行压缩。
gzip_min_length 5k;
# 指定压缩级别，取值为 1-9，数字越大，压缩比越高，但压缩所用时间会越长。默认为1，建议使用 4
gzip_comp_level 4;
# “4” 表示的是缓存颗粒数量，而“16k”表示的是缓存颗粒大小。
gzip_buffers 4 16k;
# 开启动态压缩。默认值 off。
gzip_vary on;
# 压缩协议版本配置（1.0 | 1.1），也可不设置，目前主流几乎都是v1.1版本协议
gzip_http_version 1.1;
# 指定要压缩的文件类型。默认值 text/html。
gzip_types text/plain text/javascript image/jpeg image/gif image/png;
```

### gzip 压缩其他配置

#### gzip 预压缩配置

Nginx 的动态压缩是对每个请求先压缩再输出，会造成服务端一定程度的 CPU 消耗，因此可以利用 nginx 模块 Gzip Precompression 模块进行预压缩。

同时 nginx 默认安装 `ngx_http_gzip_module`，采用的是 chunked 方式的动态压缩，静态压缩需要使用 `http_gzip_static_module` 模块，进行 pre-compress。

对需要压缩的文件，直接读取已经压缩好的文件(文件名为加.gz)，而不是动态压缩，对于不支持 gzip 的请求则读取原文件，即预压缩。

##### 语法

```nginx
# 默认 off
gzip_static on | off | always;
```

##### 可配置段

```nginx
http, server, location
```

##### 配置示例

```nginx
location /mp3  {
    #...
    gzip_static  on;
    gzip_proxied expired no-cache no-store private auth;
    #..
}
```

##### 注意

- **文件可以使用 gzip 命令来进行压缩，或任何其他兼容的命令，建议压缩文件和原始文件的修改日期和时间保持一致。** 

- **gzip_static 配置优先级高于 gzip。** 

- **开启 nginx_static 后，对于任何文件都会先查找是否有对应的 gz 文件。** 

- **gzip_types 设置对 gzip_static 无效。** 

- **gzip static 默认适用 HTTP 1.1。** 

#### gzip_buffers 压缩缓冲配置

**设置系统获取几个单位的缓存用于存储 gzip 的压缩结果数据流。** 如果没有设置，默认值是申请跟原始数据相同大小的内存空间去存储 gzip 压缩结果。

##### 语法

```nginx
# 默认值 gzip_buffers 32 4k|16 8k;
gzip_buffers number size;
```

##### 可配置段

```nginx
http, server, location
```

##### 配置示例

```nginx
# 释义：关闭IE6及以下的浏览器压缩。
location /mp3  {
    #...
    gzip on;
    gzip_buffers 4 16k;
    gzip_comp_level 2;
    gzip_disable "MSIE [1-6]\.";
    #...
}
```

#### gzip_proxied 反向代理压缩配置

**Nginx 作为反向代理的时候启用，开启或者关闭后端服务器返回的结果，匹配的前提是后端服务器必须要返回包含 "Via" 的 header 头。**

##### 语法

```nginx
# 默认值 off
gzip_proxied off | expired | no-cache | no-store | private | no_last_modified | no_etag | auth | any ...;
```

##### 可配置段

```nginx
http, server, location
```

##### 参数释义

- off：关闭所有的代理结果数据的压缩
- expired：如果 header 头中包含 "Expires" 头信息，启用压缩；
- no-cache：如果 header 头中包含 "Cache-Control: no-cache" 头信息，启用压缩；
- no-store：如果 header 头中包含 "Cache-Control: no-store" 头信息，启用压缩；
- private：如果 header 头中包含 "Cache-Control: private" 头信息，启用压缩；
- no_last_modified：如果 header 头中不包含 "Last-Modified" 头信息，启用压缩；
- no_etag：如果 header 头中不包含 "ETag" 头信息，启用压缩；
- auth：如果 header 头中包含 "Authorization" 头信息，启用压缩；
- any：无条件启用压缩。

