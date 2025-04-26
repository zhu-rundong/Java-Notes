# Nginx 应用实战之请求定位

nginx 有两层指令来匹配请求 URI （http://ip: port/uri）：

- 
  第一个层次是 server 指令，它通过域名、ip 和端口来做第一层级匹配，当找到匹配的 server 后就进入此 server 的 location 匹配。
- 第二个层次是 location 指令，它通过请求 uri 来做第二层匹配。

## server 指令

### 端口绑定

```nginx
server {
    listen       81; # 监听的端口
    server_name  localhost; # 域名或ip
    
    location / {  # 访问路径配置
        return 400;
    }
}

server {
    listen       82; # 监听的端口
    server_name  localhost; # 域名或ip
    
    location / {  # 访问路径配置
        return 401;
    }  
}
```

- 地址栏输入 http://192.168.78.100:81 可以看到 400
- 地址栏输入 http://192.168.78.100:82 可以看到 401

### 域名绑定

一个域名对应一个 ip 地址，一个 ip 地址可以被多个域名绑定。

本地测试可以修改 hosts 文件（C:\Windows\System32\drivers\etc\hosts）, 配置域名和 ip 的映射关系，如果 hosts 文件中配置了域名和 ip 的对应关系，不需要走 DNS 服务器。

```properties
192.168.78.100 www.itzrd.com register.itzrd.com
```

做好域名指向后，修改 nginx 配置文件

```nginx
server {
    listen       80;
    server_name  www.itzrd.com;

    location / {
       return 400;        
    }
}

server {
    listen       80;
    server_name  register.itzrd.com;

    location / {
        return 401;
    }
}
```

- 地址栏输入 http://www.itzrd.com 可以看到 400
- 地址栏输入 http://register.itzrd.com 可以看到 401

## location 指令

每个 server 块中可以包含多个 location 块。

location 块的主要作用是，基于 nginx 服务器接收到的客户端发送过来的请求 uri（例如：`server_name/uri-string`），对除虚拟主机名称（也可以是 IP 别名）之外的字符串（前例中 `/uri-string` 部分）进行匹配，对特定的请求进行处理。

此外，地址定向、数据缓存和应答控制等功能都是在这部分实现，许多第三方模块的配置也是在 location 块中提供功能。

### location 语法

在 nginx 的官方文档中定义的 location 的语法结构为：

```nginx
location [ = | ~ | ~* | ^~ ] /uri { ... }
```

- uri 是待匹配的请求 uri 字符串，它不包含 **查询字符串（query_string）**，如 http://localhost: 8080/test?id = 10，请求 uri 是/test。匹配字符串分为两种：**普通字符串（literal string）和正则表达式（regular expression）**，其中 ~ 和 ~* 用于正则表达式， 其他前缀和无任何前缀都用于普通字符串（为了方便，我们约定，不含正则表达的 uri 称为“标准 uri”，使用正则表达式的 uri 称为“正则 uri”）。
- **方括号里的部分，是可选项**，用来匹配 URI 类型，有四种参数可选，也可以不带参数。

### location 匹配参数解释

|  参数  | 匹配方式     | 匹配模式       | 说明                                                         | 注意事项                                                 |
| ---- | ------------ | -------------- | --------------------------------------------------------- | -------------------------------------------------------- |
| `=`  | 精准匹配     | 普通字符串匹配 | 用于 `标准uri` 前，要求请求字符串与 uri 精准匹配，成功则立即处理，nginx 停止搜索其他匹配。 |                                                          |
| `~`  | 正则匹配     | 正则表达式匹配 | 用于 `正则uri`，表示 uri 包含正则表达式，并且区分大小写。    | 如果 uri 包含正则表达式，就必须要使用“～”或者“～*”标识。 |
| `~*` | 正则匹配     | 正则表达式匹配 | 用于 `正则uri`，表示 uri 包含正则表达式，并且不区分大小写。  | 如果 uri 包含正则表达式，就必须要使用“～”或者“～*”标识。 |
| `^~` | 带参前缀匹配 | 普通字符串匹配 | 用于 `标准uri` 前，并要求一旦匹配到就会立即处理，不再去匹配其他的正则 URI，一般用来匹配目录。 |                                                          |
| `空` | 普通前缀匹配 | 普通字符串匹配 | location 后没有参数直接跟着 `标准uri`，表示前缀匹配，代表跟请求中的 uri 从头开始匹配。 |                                                          |

### location 匹配顺序

location 的匹配并不完全按照其在配置文件中出现的顺序来匹配，请求 URI 会按如下规则进行匹配：

1. 先精准匹配 **`=`** ，精准匹配成功则会立即停止其他类型匹配；
2. 没有精准匹配成功时，进行前缀匹配。先查找带有 **`^~`** 的前缀匹配，带有 **`^~`** 的前缀匹配成功则立即停止其他类型匹配，普通前缀匹配（不带参数 **`^~`** ）成功则会暂存，继续查找正则匹配；
3. **`=`** 和 **`^~`** 均未匹配成功前提下，查找正则匹配 **`~`** 和 **`~*`** 。当同时有多个正则匹配时，按其在配置文件中出现的 `先后顺序优先匹配`，命中则立即停止其他类型匹配；

    **注意：正则匹配会根据匹配顺序，找到第一个匹配的正则表达式后将停止搜索。普通字符串匹配则无视顺序，只会选择最精确的匹配。**
4. 所有正则匹配均未成功时，返回步骤 2 中暂存的普通前缀匹配（不带参数 **`^~`** ）结果。

以上规则简单总结就是优先级从高到低依次为（**`序号越小优先级越高`**）：

```bash
1. location =    # 精准匹配
2. location ^~   # 带参前缀匹配
3. location ~    # 正则匹配（区分大小写）
4. location ~*   # 正则匹配（不区分大小写）
5. location /a   # 普通前缀匹配，优先级低于带参数前缀匹配。
6. location /    # 任何没有匹配成功的，都会匹配这里处理
```

### location 匹配示例

#### 通用匹配

```nginx
 server {
     listen       80;
     server_name  localhost;
     
     # 因为所有的地址都以 /开头，所以这条规则将匹配到所有请求，比如访问 / 和 /doc, 则 / 匹配 /data 也匹配
     location / {
         return 402;
     }
 }
```

#### 普通前缀匹配

##### 普通前缀匹配和通用匹配

```nginx
server {
    listen 80;
    server_name localhost;
    
    # 匹配任何以/doc 开头的地址，匹配符合以后，还要继续往下搜索其它 location ，只有其它 location 后面的正则表达式没有匹配到时，     才会采用这一条
    location /doc {
        return 400;
    }
    
    location / {
        return 402;
    }
}
```

匹配规则：只要请求是以/doc 开头的路径就可命中：

- `curl -I localhost/doc` 返回 `返回 HTTP/1.1 400`

- `curl -I localhost/document` 返回 `返回 HTTP/1.1 400`

- `curl -I localhost/doc/hello` 返回 `返回 HTTP/1.1 400`

- `curl -I localhost/hello` 返回 `返回 HTTP/1.1 402`

##### 多个普通前缀匹配

```nginx
server {
    listen 80;
    server_name localhost;
    
    location /document {
        return 401;
    }
    location /doc {
        return 402;
    }
}
```

- `curl -I localhost/document` 会返回 `HTTP/1.1 401`
- `curl -I localhost/documents` 会返回 `HTTP/1.1 401`
- `curl -I localhost/docu` 会返回 `HTTP/1.1 402`
- `curl -I localhost/documen` 会返回 `HTTP/1.1 402`

**注意：前缀匹配下，返回最长匹配的 location，与 location 所在位置顺序无关**

#### 正则匹配

##### 普通前缀匹配和正则匹配

```nginx
server {
    listen 80;
    server_name localhost;
    
    location /document {
        return 401; 
    }
    location ~* ^/doc {
        return 402; 
    }
}
```

- `curl -I localhost/document` 返回 `返回 HTTP/1.1 402`

**说明：按照上述的规则，正则匹配会有更高的优先级。**

##### 多个区分大小写正则匹配

```nginx
server {
    listen 80;
    server_name localhost;

    location ~ ^/doc[a-z]+ {
        return 401;
    }

    location ~ ^/docu[a-z]+ {
        return 402;
    }
}
```

- `curl -I localhost/document` 返回 `HTTP/1.1 401`

把顺序换一下

```nginx
server {
    listen 80;
    server_name localhost;

    location ~ ^/docu[a-z]+ {
        return 402;
    }
    
    location ~ ^/doc[a-z]+ {
        return 401;
    }
}
```

- `curl -I localhost/document` 返回 `HTTP/1.1 402`

**说明：正则匹配是使用文件中的顺序，先匹配成功的返回。** 

##### 区分和不区分大小写正则匹配

```nginx
server {
    listen 80;
    server_name localhost;

    location ~ /doc{
        return 401;
    }

    location ~* /doc {
        return 402;
    }
}
```

- `curl -I localhost/doc` 返回 `HTTP/1.1 401`

- `curl -I localhost/DOC` 返回 `HTTP/1.1 402`

#### 带参前缀匹配

##### 多个带参前缀匹配

```nginx
server {
    listen 80;
    server_name localhost;
    
    location ^~ /doc {
        return 401;
    }
    
    location ^~ /document {
        return 402;
    }
}
```

- `curl localhost/docu` 返回 `HTTP/1.1 401`
- `curl localhost/documents` 返回 `HTTP/1.1 402`

**说明：同样类型下（当前均为带参前缀匹配），返回最长匹配的 location 配置。** 

##### 带参前缀匹配和正则匹配

```nginx
server {
    listen 80;
    server_name localhost;
    
    location ^~ /doc {
        return 401;
    }
    
    location ~* ^/document$ {
        return 402;
    }
}
```

- `curl localhost/document` 返回 `HTTP/1.1 401`

**说明：第一个带参前缀匹配** **`^~`** **命中以后不会再搜寻正则匹配，所以会第一个命中。** 

#### 精准匹配

##### 多个精准匹配

```nginx
 server {
     listen       80;
     server_name  localhost;
              
     location = / {
         return 400;
     }
     
     location = /document {
         return 401;
     }
 }
```

- `curl -I localhost/` 返回 `HTTP/1.1 400`

- `curl -I localhost/document` 返回 `HTTP/1.1 401`

##### 精准匹配和带参前缀匹配

```nginx
 server {
     listen       80;
     server_name  localhost;
     
     location ^~ /document {
         return 402;
     }
                  
     location = / {
         return 400;
     }
     
     location = /document {
         return 401;
     }
 }
```

- `curl -I localhost/document` 返回 `HTTP/1.1 401`

#### 复杂匹配场景

假设我们有如下几个请求等待匹配：

```nginx
/
/index.html
/documents/document.html
/documents/abc
/images/a.gif
/documents/a.jpg
```

以下是 location 配置及其匹配情况:

```nginx
# A
location  = / {
    # 只精准匹配 / 的查询.
    return 400;
}
# 匹配成功： / 

# B
location / {
    # 匹配任何请求，因为所有请求都是以”/“开始
    # 但是更长字符匹配或者正则表达式匹配会优先匹配
    return 401;
}
#匹配成功：/index.html

# C
location /documents {
    # 匹配任何以 /documents/ 开头的地址，匹配符合以后，还要继续往下搜索/
    # 只有后面的正则表达式没有匹配到时，这一条才会采用这一条/
    return 402;
}
# 匹配成功：/documents/document.html
# 匹配成功：/documents/abc
# 由于下面还有一个 ~ /documents/ 的正则配置，优先级更高，因此，402的配置无用，以上两个匹配成功的请求都会返回404。

# D
location ~ /documents/ {
    # 区分大小写的正则匹配
    # 匹配任何以 /documents/ 开头的地址，匹配符合以后，还要继续往下搜索/
    # 只有后面的正则表达式没有匹配到时，这一条才会采用这一条/
    return 404;
}

# E
location ^~ /images/ {
    # 匹配任何以 /images/ 开头的地址，匹配符合以后，立即停止往下搜索正则，采用这一条。/
    return 405;
}
# 成功匹配：/images/a.gif

# F
location ~* \.(gif|jpg|jpeg)$ {
    # 匹配所有以 .gif、.jpg 或 .jpeg 结尾的请求，不区分大小写
    # 然而，所有请求 /images/ 下的图片会被 [ config E ]  处理，因为 ^~ 到达不了这一条正则/
    return 406; 
}
# 成功匹配：/documents/a.jpg

# G
location /images/ {
    # 字符匹配到 /images/，继续往下，会发现 ^~ 存在/
    return 301;
}

# H
location /images/abc {
    # 最长字符匹配到 /images/abc，继续往下，会发现 ^~ 存在/
    # G与H的放置顺序是没有关系的/
    return 302;
}

# I
location ~ /images/abc/ {
    # 只有去掉 [ config E ] 才有效：先最长匹配 [ config H ] 开头的地址，继续往下搜索，匹配到这一条正则，采用/
    return 304;
}
```

### location 实际网站配置

**至少有三个匹配规则定义**

#### 第一个必选规则

- 直接匹配网站根，通过域名访问网站首页比较频繁，使用这个会加速处理，比如说官网。
- 可以是一个静态首页，也可以直接转发给后端应用服务器

```nginx
location = /index.html {
     root   html
     index  index.html index.htm
 }
 
 location = / {
     root   html
     index  index.html index.htm
 }
```

#### 第二个必选规则

- 处理静态文件请求，这是 nginx 作为 http 服务器的强项
- 有两种配置模式，`目录匹配` 或 `后缀匹配`, 任选其一或搭配使用

```nginx
location ^~ /static/ {
     root /webroot/static/;
 }
 
 location ~* .(gif|jpg|jpeg|png|css|js|ico)$ {
     root /webroot/res/;
 }
```

#### 第三个规则

- 通用规则，比如用来转发带 .php、.jsp 后缀的动态请求到后端应用服务器，非静态文件请求就默认是动态请求。

```nginx
location / {
     proxy_pass http://192/168.78.100:8080;
 }
```

### location 其他配置用法

#### 匹配问号后的参数

请求 URI 中问号后面的参数是不能在 location 中匹配到的，这些参数存储在 **`$query_string`** 变量中，可以用 **`if`** 来判断。

例如，对于参数中带有单引号 `' ; < >` 进行匹配，然后重定向到错误页面。

```http
/plus/list?tid=19&mid=1124‘
```

```nginx
if ( $query_string ~* ”.*[;’<>].*“ ){
  return 404;
}
```

#### URI 结尾带不带 `/`

1. `location` 中的字符有没有 `/` 都没有影响，也就是说 `/user/` 和 `/user` 是一样的。
2. 如果 URI 结构是 `https://domain.com/` 的形式，尾部有没有 `/` 都不会造成重定向。因为浏览器在发起请求的时候，默认加上了 `/` 。虽然很多浏览器在地址栏里也不会显示 `/` 。
3. 如果 URI 的结构是 `https://domain.com/` `some-dir/` 。尾部如果缺少 `/` 将导致重定向。因为根据约定，URL 尾部的 `/` 表示目录，没有 `/` 表示文件。
    1. 如果访问 `/some-dir/` 时，服务器会自动去该目录下找对应的默认文件。
    2. 如果访问 `/some-dir` 时，服务器会先去找 `some-dir` 文件，找不到的话会将 `some-dir` 当成目录，重定向到 `/some-dir/` ，去该目录下找默认文件。

**第一点与 location 配置有关，其他两点无关。**

### rewrite 指令

**rewrite 功能：使用 nginx 提供的全局变量或自己设置的变量，结合正则表达式和标记位实现 URL 重写以及重定向。**

比如：更换域名后需要保持旧的域名能跳转到新的域名上、某网页发生改变需要跳转到新的页面、网站防盗链等等需求。

rewrite 只能放在 `server{},location{},if{}` 中，并且默认只能对 `域名后边的除去传递的参数外的字符串` 起作用。

#### rewrite 跳转场景

- 可以调整用户浏览的 URL，看起来更规范，合乎开发及产品人员的需求

- 为了让搜索引擎搜录网站内容 SEO 及用户体验更好，企业会将动态 URL 地址伪装成静态地址提供服务
- 网址换新域名后，让旧的访问跳转到新的域名上。例如，访问京东的 360buy.com 会跳转到 [jd.com](http://jd.com)
- 根据特殊变量、目录、客户端的信息进行 URL 调整等。

#### rewrite 跳转实现

- nginx 是通过 `ngx_http_rewrite_module` 模块支持 url 重写、支持 if 条件判断，但不支持 else。
- 另外该模块需要 `PCRE` 支持，应在编译 nginx 时指定 PCRE 支持，默认已经安装。
- 根据相关变量重定向和选择不同的配置，从一个 location 跳转到另一个 location，不过这样的循环最多可以执行 `10` 次，超过后 nginx 将返回 500 错误。

#### rewrite 执行顺序

- 执行 server 块里面的 rewrite 指令。
- 执行 location 匹配。
- 执行选定的 location 中的 rewrite 指令。

#### rewrite 语法格式

```nginx
rewrite <regex> <replacement> [flag]
regex ：表示正则匹配规则。
replacement ：表示跳转后的内容。
flag ：表示 rewrite 支持的 flag 标记。
```

**flag 标记说明：**

- `last` ：本条规则匹配完成后，不终止重写后的 url 匹配，一般用在 `server` 和 `if` 中。
- `break` ：本条规则匹配完成即终止，终止重写后的 url 匹配，一般使用在 `location` 中。
- `redirect` ：返回 `302` 临时重定向，浏览器地址会显示跳转后的 URL 地址。
- `permanent` ：返回 `301` 永久重定向，浏览器地址栏会显示跳转后的 URL 地址。

### rewrite 示例

##### 基于域名的跳转

**要求：** 现在公司旧域名 www.itzrd.com 有业务需求变更，需要使用新域名 www.newitzrd.com 代替，但是旧域名不能废除，需要跳转到新域名上，而且后面的参数保持不变。

```nginx
 server {
     listen       80;
     server_name  www.itzrd.com;   # 修改主机名
                  
     location / {
     
         # 判断主机名为www.itzrd.com则重写为www.newitzrd.com
         if ($host = 'www.itzrd.com'){                  
             rewrite ^/(.*)$ http://www.newitzrd.com/$1 permanent;  
         }
         
         root   html;
         index  index.html index.htm;
     }
 }
 
 server {
     listen       80;
     server_name  www.newitzrd.com;   # 修改主机名
                  
     location / {
         return 400;
     }
 }
```

浏览器访问 www.itzrd.com 会显示 400 页面，表示跳转成功。

#### 基于客户端 IP 访问跳转

**要求：** 今天公司业务新版本上线，要求所有 IP 访问任何内容都显示一个固定维护页面，只有公司 IP ：192.168.78.100 访问正常。

```nginx
 
 server {
     listen       80;
     server_name  localhost;     
 
     # 设置变量为$rewrite,变量值为boolean类型的true
     set $rewrite true;        
     # 当客户端IP为192.168.78.100时，将 $rewrite 变量值设为false              
     if ($remote_addr = "192.168.78.100"){       
         set $rewrite false;
     }
     
     # 当变量值为true时，进行rewrite重写，将域名后面的路径重写为/weihu.html
     if ($rewrite = true){                       
         rewrite (.+) /weihu.html;               
     }
     
     # 精确匹配/weihu.html请求
     location = /weihu.html {
         # 看到400页面，就表示重定向成功了
         return 400;                    
     }
     
     location / {
         return 401;
     }
 
 }
```

- `192.168.78.100` 机器访问：`返回 HTTP/1.1 401`
- 非 `192.168.78.100` 机器访问：`返回 HTTP/1.1 400`，表示跳转成功

**注意：** 如果 `rewrite (.+) /weihu.html;` 改成 `rewrite (.+) /weihu.html permanent;` 的话，如果是非 192.168.78.100 的主机访问会使浏览器修改请求访问的 URL 成 www.itzrd.com/weihu.html 再请求访问，这样就会进入一直在 rewrite 的死循环，访问请求会一直被重写成 www.itzrd.com/weihu.html 再请求访问。

#### 基于旧域名跳转到新域名后面加目录

现在访问的是 `register.itzrd.com/post/`，现在需要将这个域名下面的访问都跳转到 www.itzrd.com/bbs/post/。

```nginx
 server {
     listen       80;
     server_name  register.itzrd.com;   
     
     location /post {
         # 这里的$1表示为post
         rewrite ^/(.+) http://www.itzrd.com/bbs$1 permanent;        
     }
 }
 
  server {
     listen       80;
     server_name  www.itzrd.com;   
     
     location = /bbs/post/index.html {
         return 400;
     }
 }
```

浏览器访问 `bbs.itzrd.com/post/index.html` 最终显示 400 页面，表示跳转成功。

#### 基于参数匹配的跳转

访问 www.itzrd.com/100-(100|200)-100.html 跳转到 www.itzrd.com 页面。

```nginx
 server {
     listen       80;
     server_name  www.itzrd.com;    
 
     if ($request_uri ~ ^/100-(100|200)-(\d+).html$) {
         rewrite (.+) http://www.itzrd.com permanent;
     }
     
     location / {
         return 400;
     }
 
 }
```

浏览器访问 www.itzrd.com/100-200-100.html 或 www.itzrd.com/100-100-100.html 最终显示 400 页面，表示跳转成功。

**nginx 变量说明：**

- `$request_uri`：包含请求参数的原始 URI，不包含主机名。

    如：http://www.itzrd.com/abc/bbs/index.html?a = 1&b = 2 中的 /abc/bbs/index.php?a = 1&b = 2 
- `$uri`：这个变量指当前的请求 URI，不包括任何参数

    如：http://www.itzrd.com/abc/bbs/index.html?a = 1&b = 2 中的/abc/bbs/index.html
- `$documenturi`：与 uri 相同，这个变量指当前的请求 URI，不包括任何传递参数

    如：http://www.itzrd.com/abc/bbs/index.html?a = 1&b = 2 中的/abc/bbs/index.html

#### 基于目录下所有 html 结尾的文件跳转

要求：访问 www.itzrd.com/upload/123.html 跳转到首页（upload 目录下）。

```nginx
 server {
     listen       80 ;
     server_name  www.itzrd.com ;
      
     location ~* /upload/.*.html$ {
         rewrite (.+) http://www.itzrd.com permanent
     }
     
     location / {
         return 400;
     }
 }
```

浏览器访问 www.itzrd.com/upload/123.html 最终显示 400 页面，表示跳转成功。

#### 基于普通一条 url 请求的跳转

要求访问一个具体的页面如 www.itzrd.com/abc/123.html 跳转到首页

```nginx
 server {
     listen       80 ;
     server_name  www.itzrd.com;
 
     location ~* ^/abc/123.html {
         rewrite (.+) http://www.itzrd.com permanent
     }
     
     location / {
         return 400;
     }
 }
```

浏览器访问 www.itzrd.com/abc/123.html 最终显示 400 页面，表示跳转成功。
