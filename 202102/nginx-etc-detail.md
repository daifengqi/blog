# Nginx 配置概述

# Nginx Configuration Overview

## 2021 年 2 月 7 日

## Feb 7 2021

---

Nginx 配置位于`/etc/nginx/`目录下，文件名可能是`nginx.conf`或者其它以`.conf`为后缀的文件。

首先，Nginx 有一些全局配置项，定义在全局环境，如下。

- user：定义工作进程使用的用户和组。比如，`user: root`。
- worker_processes：定义工作进程数量，除非需要限制使用，建议设置为`auto`。
- error_log：定义报错时日志的位置，有默认值。
- pid：定义存储主进程的进程 ID 的文件，有默认值。
- include：Nginx 动态模块的位置，你可以在其它地方进行 Nginx 配置，很有可能是为了某个其它应用。

此外，对于 HTTP 协议相关的配置，这些配置行要放在一个 http 块。

```nginx
http {
  参数名: 内容
}
```

http 块比较重要的参数有以下这些。

- tcp_nopush：这个参数直接控制操作系统 TCP_NOPUSH 的 Socket 选项。当值为 on 时，TCP 将延迟发送任何数据，直到套接字关闭或内部发送缓冲区填满为止。否则，当发送方的 TCP 设置为 push 时，每次用户调用 write 或 writev 结束时，数据将会立即开始传输（如果允许）。
- tcp_nodelay：在默认情况下，TCP 启用 Nagle 算法：延迟小数据包的传输，只有当接收方成功地发回先前包的所有 Acknowledegments 时，之前收集的小数据包才会被传输。Nagle 算法旨在防止服务器被大量的小数据包淹没，然而这可能会导致持久连接上的一些临时死锁。所以我们应该禁止这种算法，设置 tcp_nodelay 的值为 on。
- keepalive_timeout：HTTP 长连接的时间设定，在设置期间保持客户端的 HTTP 连接将在服务器端保持开启状态。设置为零将禁用保持活动的客户端连接。建议将该值设置为 60 秒或更高一些。

Nginx 作为 Web 服务器，需要设置 Nginx 监听的端口，这些在 server 块设置，如下。

```nginx
http {
  # ...
  server {
    参数名: 内容
  }
}
```

在 server 块，一些常见的参数是：

- listen：代表监听的端口，通常是 80，指代 HTTP 服务的端口。
- root：网页根目录的位置，默认是`/usr/share/nginx/html/`。
- include：http 服务可以加载的模块配置，是一些第三方框架需要放置的 config 文件的位置。
- location、error_page 等：用来设置重定向、错误返回页面（比如 404）等。

在这篇文章的后半部分，会给出一些常见的优化手段，如下。

- worker_rlimit_nofile：全局配置项。在 UNIX 系统里，一切皆文件。这个参数指定了一个 Nginx 进程可以打开多少个文件，即上限。建议设置为一个比较大的值，比如 2048。

- gzip：http 配置项。gzip 模块是一个过滤器，它使用“gzip”方法压缩响应。这有助于将传输数据的大小减少一半甚至更多。在此，推荐一个 gzip 有关的配置。

  ```nginx
  gzip on;               # enable gzip
  gzip_http_version 1.1; # turn on gzip for http 1.1 and higher
  gzip_disable "msie6";  # IE 6 had issues with gzip
  gzip_comp_level 5;     # inc compresion level, and CPU usage
  gzip_min_length 100;   # minimal weight to gzip file
  gzip_proxied any;      # enable gzip for proxied requests (e.g. CDN)
  gzip_buffers 16 8k;    # compression buffers (if we exceed this value, disk will be used instead of RAM)
  gzip_vary on;          # add header Vary Accept-Encoding (more on that in Caching section)

  # 定义gzip可以传输的文件类型（以下配置涵盖绝大部分文件类型）
  gzip_types text/plain;
  gzip_types text/css;
  gzip_types application/javascript;
  gzip_types application/json;
  gzip_types application/vnd.ms-fontobject;
  gzip_types application/x-font-ttf;
  gzip_types font/opentype;
  gzip_types image/svg+xml;
  gzip_types image/x-icon;
  ```

- add_header：http 配置项。通过向 http 请求头部添加内容，可以提高性能。对于多次访问的用户，我们可以将内容缓存到用户本地，通过添加如下配置。

  ```nginx
  add_header Cache-Control public;
  add_header Pragma public;
  ```

最后，建议设置一个 Nginx 的 HTTPS 服务器并将所有 HTTP 请求重定向到 HTTPS，这是另外的话题了。

**Appendix**

修改 Nginx 配置后需要用到的命令行：

```shell
# 测试配置
nginx- t
# 重新加载
nginx -s reload
```

**Reference**

Nginx 官方文档. http://nginx.org/en/docs/.

FreeBSD 操作手册. https://www.freebsd.org/cgi/man.cgi?query=tcp&sektion=4&manpath=FreeBSD+9.0-RELEASE.

Nginx 性能优化教程. https://www.netguru.com/codestories/nginx-tutorial-performance

Nginx 的 HTTPS 设置. http://nginx.org/en/docs/http/configuring_https_servers.html
