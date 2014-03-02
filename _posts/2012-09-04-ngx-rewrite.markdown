---
layout: post
title: "ngx-rewrite"
date: 2012-09-04 20:19
comments: true
categories: 
---

建议直接看官方文档^-^:[ngx\_http\_rewrite\_module](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)

nginx的rewrite模块包括:break,if,return,rewrite,rewrite\_log,set,uninitialized\_variable\_warn这个几个指令.
nginx的rewrite模块使得你可以通过正则表达式匹配、return redirect、条件选择配置来改变URIs.

### 指令处理的顺序:

从写在server block的指令开始处理(大部分指令都是从server block开始写)

搜索将要跳转的location

选择一个location进行处理,如果改变uri,一个新的location将会被搜索,如果循环超过10次,将会自动报500错误

<!-- more -->
### return指令(server,location,if)

return code [text] | URL

return URL

停止继续处理并返回一个code给client.

### rewrite指令(server,location,if)

rewrite regex replacement [flag]->[last|break|redirect|permanent]

如果URI符合了regex正则匹配,URI将会被改变为replacement string,因为rewrite指令会按照配置文件里面的出现顺序的执行下去,
flag能够终止指令的进一步处理.

如果replacement是以"http://""https://"开头的,一个redirect(302)请求会发往客户端.

last : 找到一个新的location matching the changed URI后就终止rewrite模块的处理

break : 停止处理rewrite模块的当前设置

redirect : 返回一个临时的跳转(http code 302),这个是当replacement string不是以"http://""https://"开头时用的

permanent : 返回一个永久跳转(http code 301)


### break指令(server,location,if)

stops processing the current set of ngx\_http\_rewrite\_module directives.
停止处理当前rewrite模块指令的设置

### if指令(server,location)

if(condition){
    ....
}

condition:
```
variable name false: empty string, any string start with "0"
use = !=
match a variable with a string using "~" "~*"(忽略大小写)
check if a file exist with "-f" "!-f"
check directory "-d" "!-d"
"-e" "!-e" check a file,directory,or a symbolic
"-x" "!-x" check for executable file
```

### The end

At last I will give a config of rewrite all requests to php entry file which is alwasys used 
in php project.

```
server {
    listen 80;
    server_name localhost;
    root /path/to/your/project;
    location / {
        rewrite ^/(.*)$ /index.php/$1 last;
    }
    location ~ \.php {
        fastcgi_pass    127.0.0.1:9000;
        fastcgi_split_path_info  ^(.+\.php)(.*)$;
        fastcgi_param PATH_INFO     $fastcgi_path_info;
        fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```
