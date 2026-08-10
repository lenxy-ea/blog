---
title: 把QAT塞到nginx中
date: 2026-04-23 13:28:19
tags: 折腾日记
---

# 我就要把QAT塞到nginx中

最近在研究各家的tls offload方案，于是有了本片文章的契机。恰巧手里有一块intel xeon 8562y+ （es）能让我顺手来测试。（为什么 我的xeon max不能跑QAT!

## 准备过程

### 安装qatlib

> 该软件包提供**用户空间库**，可用于访问 **Intel(R) QuickAssist** 设备，并提供 **Intel(R) QuickAssist API** 及示例代码。

在开始前先确认自己的硬件版本是否在qatlib的支持范围内，QAT HW 2.0以前的设备均不受qatlib支持

>      * platform must have an Intel® Communications device installed.
>         Supported devices:
>         ---------------------------
>         Driver Name | PFid | VFid
>         ---------------------------
>         4xxx        | 4940 | 4941
>         401xx       | 4942 | 4943
>         402xx       | 4944 | 4945
>         420xx       | 4946 | 4947
>         ---------------------------

如果不在qatlib的支持范围，如C3xxx设备可以参考我后续<咕咕咕>的文章。

```bash
    #在开始之前，需要开启iommu
    sudo grubby --update-kernel=/boot/vmlinuz-$(uname -r) --args="intel_iommu=on iommu=pt"
    reboot
    #重启后可以开始安装
    git clone https://github.com/intel/qatlib.git
    cd qatlib
    ./autogen.sh
    ./configure --enable-service
    make -j
    sudo make install
    #安装完成后记得观察一下 qat service的状态
    ● qat.service - QAT service
     Loaded: loaded (/usr/lib/systemd/system/qat.service; enabled; preset: 5:185mdisabled)
     Active: active (running) since Thu 2026-04-23 12:01:57 HKT; 1h 39min ago
 Invocation: f18aa9e6c7404c60944a332c39df44a3
    Process: 121054 ExecStartPre=/bin/sh -c test $(getent group qat) (code=exited, status=0/SUCCESS)
    Process: 121058 ExecStartPre=/usr/sbin/qat_init.sh (code=exited, status=0/SUCCESS)
    Process: 138320 ExecStart=/usr/sbin/qatmgr --policy=${POLICY} (code=exited, status=0/SUCCESS)
   Main PID: 138832 (qatmgr)
      Tasks: 1 (limit: 97198)
     Memory: 1.2M (peak: 18.4M)
        CPU: 690ms
     CGroup: /system.slice/qat.service
             └─138832 /usr/sbin/qatmgr --policy=2

Apr 23 12:01:55 adlink.lab systemd[1]: Starting qat.service - QAT service...
Apr 23 12:01:57 adlink.lab systemd[1]: Started qat.service - QAT service.
```

qatlib完成后意味着你已经可以正常的调用qat硬件进行加速，此时你可以使用 *cpa_sample_code*  这个命令来进行测试

### 安装qat engine

这里有一个问题要注意一下，如果使用系统自带的openssl，你卸载失败后可能会遇到ssh无法正常访问的情况。所以我手动编译了一个openssl 3.5.6用来做测试。（构建

` /usr/local/ssl/bin/openssl  version
OpenSSL 3.5.6 7 Apr 2026 (Library: OpenSSL 3.5.6 7 Apr 2026)`



```
git clone https://github.com/intel/QAT_Engine.git
git submodule update --init
./autogen.sh 
./configure --with-openssl_install_dir=/usr/local/ssl
make
make install
```

安装完成后会生成一个qatengine.so（这里用到qat engine 而不用qat provider是因为后续配置nginx 文档要求是qatengine）

随后需要配置 openssl.cnf

**注意注意注意**！！！如果要在nginx中使用，下面的内容请不要配置！

> Loading Engine via OpenSSL.cnf is not viable in Nginx. Please use the SSL Engine Framework which provides a more powerful and flexible mechanism to configure Nginx SSL engine such as Intel® QAT Engine directly in the Nginx configuration file (`nginx.conf`).
>
> 在 Nginx 中，通过 OpenSSL.cnf 加载引擎是不行的。请使用 SSL 引擎框架，该框架提供了一种更强大、更灵活的机制，可直接在 Nginx 配置文件（nginx.conf）中配置 Nginx SSL 引擎（例如 Intel® QAT 引擎）。

```
[openssl_init]
#providers = provider_sect
engines = engine_section
[ engine_section ]
qat = qat_section
[ qat_section ]
engine_id = qatengine
dynamic_path = /usr/local/ssl/lib64/engines-3/qatengine.so
default_algorithms = ALL
```

就此已经完成了openssl的配置 可以用以下命令进行测试。

```bash
[root@adlink ~]# /usr/local/ssl/bin/openssl engine -t -c -v qatengine
(qatengine) Reference implementation of QAT crypto engine(qat_hw) v1.9.0
 [RSA, EC, AES-256-CBC-HMAC-SHA256, id-aes128-CCM, id-aes192-CCM, id-aes256-CCM, TLS1-PRF, X25519, X448]
     [ available ]
     ENABLE_EXTERNAL_POLLING, POLL, SET_INSTANCE_FOR_THREAD, 
     GET_NUM_OP_RETRIES, SET_MAX_RETRY_COUNT, SET_INTERNAL_POLL_INTERVAL, 
     GET_EXTERNAL_POLLING_FD, ENABLE_EVENT_DRIVEN_POLLING_MODE, 
     GET_NUM_CRYPTO_INSTANCES, DISABLE_EVENT_DRIVEN_POLLING_MODE, 
     SET_EPOLL_TIMEOUT, SET_CRYPTO_SMALL_PACKET_OFFLOAD_THRESHOLD, 
     ENABLE_INLINE_POLLING, ENABLE_HEURISTIC_POLLING, 
     GET_NUM_REQUESTS_IN_FLIGHT, INIT_ENGINE, SET_CONFIGURATION_SECTION_NAME, 
     ENABLE_SW_FALLBACK, HEARTBEAT_POLL, DISABLE_QAT_OFFLOAD, HW_ALGO_BITMAP
     
[root@adlink ~]# /usr/local/ssl/bin/openssl speed  -elapsed -async_jobs 72  rsa2048
You have chosen to measure elapsed time instead of user CPU time.
Doing 2048 bits private rsa sign ops for 10s: 593553 2048 bits private RSA sign ops in 10.00s
Doing 2048 bits public rsa verify ops for 10s: 3672717 2048 bits public RSA verify ops in 10.00s
Doing 2048 bits public rsa encrypt ops for 10s: 2869273 2048 bits public RSA encrypt ops in 10.00s
Doing 2048 bits private rsa decrypt ops for 10s: 593701 2048 bits private RSA decrypt ops in 10.00s
Doing rsa2048 keygen ops for 10s: 288 rsa2048 KEM keygen ops in 10.01s
Doing rsa2048 encaps ops for 10s: 2375105 rsa2048 KEM encaps ops in 10.00s
Doing rsa2048 decaps ops for 10s: 593493 rsa2048 KEM decaps ops in 10.01s
Doing rsa2048 keygen ops for 10s: 284 rsa2048 signature keygen ops in 10.01s
Doing rsa2048 signs ops for 10s: 593553 rsa2048 signature sign ops in 10.00s
Doing rsa2048 verify ops for 10s: ^[[A^[[A3486014 rsa2048 signature verify ops in 10.00s
version: 3.5.6
built on: Wed Apr 22 16:27:30 2026 UTC
options: bn(64,64)
compiler: gcc -fPIC -pthread -m64 -Wa,--noexecstack -Wall -O3 -DOPENSSL_USE_NODELETE -DL_ENDIAN -DOPENSSL_PIC -DOPENSSL_BUILDING_OPENSSL -DNDEBUG
CPUINFO: OPENSSL_ia32cap=0x7ffef3ffffebffff:0xfb417ffef3bfb7ef:0x00001c30ffdd4432:0x0000000000040000:0x0000000000000000
                   sign    verify    encrypt   decrypt   sign/s verify/s  encr./s  decr./s
rsa  2048 bits 0.000017s 0.000003s 0.000003s 0.000017s  59355.3 367271.7 286927.3  59370.1
                               keygen    encaps    decaps keygens/s  encaps/s  decaps/s
                    rsa2048 0.034757s 0.000004s 0.000017s      28.8  237510.5   59290.0
                               keygen     signs    verify keygens/s    sign/s  verify/s
                    rsa2048 0.035246s 0.000017s 0.000003s      28.4   59355.3  348601.4
#硬件加速器已经有数据
[root@adlink ~]# cat /sys/kernel/debug/qat_*/fw_counters
AE          Requests        Responses
 0:         17949407         17949407
 1:         13057357         13057357
 2:          8720807          8720807
 3:          8706497          8706497
 4:         13761988         13761988
 5:         13744950         13744950
 6:         13719391         13719391
 7:         13573711         13573711
AE          Requests        Responses
 0:                0                0
 1:                0                0
 2:                0                0
 3:                0                0
 4:                0                0
 5:                0                0
 6:                0                0
 7:                0                0
```

### 编译

```sh
git clone https://github.com/intel/asynch_mode_nginx.git
./configure \
  --prefix=$NGINX_INSTALL_DIR \
  --with-http_ssl_module \
  --add-dynamic-module=modules/nginx_qatzip_module \
  --add-dynamic-module=modules/nginx_qat_module/ \
  --with-cc-opt="-DNGX_SECURE_MEM -I$OPENSSL_LIB/include -I$ICP_ROOT/quickassist/include -I$ICP_ROOT/quickassist/include/dc -I$QZ_ROOT/include -Wno-error=deprecated-declarations" \
  --with-ld-opt="-Wl,-rpath=$OPENSSL_LIB/lib64 -L$OPENSSL_LIB/lib64 -L$QZ_ROOT/src -lqatzip -lz"
  make
  make install
```

```nginx
#nginx配置文件

load_module modules/ngx_ssl_engine_qat_module.so;
load_module modules/ngx_http_qatzip_filter_module.so;

worker_processes  1;

ssl_engine {
    use_engine qatengine;
    default_algorithms RSA,EC,DH,DSA,PKEY;
    qat_engine {
        qat_offload_mode async;
        qat_notify_mode poll;
        qat_poll_mode heuristic;
        qat_sw_fallback on;
    }
}


events {
    use epoll;
    worker_connections 102400;
    accept_mutex off;
}


http {

    gzip on;
    gzip_min_length     128;
    gzip_comp_level     1;
    gzip_types  text/css text/javascript text/xml text/plain text/x-component application/javascript application/json application/xml application/rss+xml font/truetype font/opentype application/vnd.ms-fontobject image/svg+xml;
    gzip_vary            on;
    gzip_disable        "msie6";
    gzip_http_version   1.0;

    qatzip_sw failover;
    qatzip_min_length 128;
    qatzip_comp_level 1;
    qatzip_buffers 16 8k;
    qatzip_types text/css text/javascript text/xml text/plain text/x-component application/javascript application/json application/xml application/rss+xml font/truetype font/opentype application/vnd.ms-fontobject image/svg+xml application/octet-stream image/jpeg;
    qatzip_chunk_size   64k;
    qatzip_stream_size  256k;
    qatzip_sw_threshold 256;

    # HTTP server with QATZip enabled.
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
    }

    # HTTPS server with async mode.
    server {
        #If QAT Engine enabled, add `ssl_asynch  on;` to the context.
        ssl_asynch on;
        listen       443 ssl;
        server_name  localhost;

        ssl_protocols       TLSv1.3;
        ssl_ecdh_curve prime256v1; #使用X
        ssl_certificate      conf/server.crt;
        ssl_certificate_key  conf/server.key;

        location / {
            root   html;
            index  index.html index.htm;
        }
    }
}
```

## wrt测试结果

```sh
 #开启qat加速
 ./wrk -t 20 -c 20000 https://172.16.10.44 
Running 10s test @ https://172.16.10.44
  20 threads and 20000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     4.38ms    1.90ms  97.47ms   76.84%
    Req/Sec     5.79k     4.48k   17.64k    53.97%
  874735 requests in 10.10s, 730.02MB read
  Socket errors: connect 369, read 0, write 0, timeout 0
  Non-2xx or 3xx responses: 1457
Requests/sec:  86630.58
Transfer/sec:     72.30MB
#关闭qat加速
./wrk -t 20 -c 20000 https://172.16.10.44 
Running 10s test @ https://172.16.10.44
  20 threads and 20000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     9.42ms    8.11ms 211.61ms   95.67%
    Req/Sec     4.58k     3.76k   26.35k    70.61%
  612236 requests in 10.08s, 511.14MB read
  Non-2xx or 3xx responses: 641
Requests/sec:  60711.28
Transfer/sec:     50.69MB

```

| 指标 | 开启 QAT | 未开启 QAT | 差异 |
|---|---:|---:|---:|
| Avg Latency | 4.38 ms | 9.42 ms | **-53.5%** |
| Latency Stdev | 1.90 ms | 8.11 ms | **-76.6%** |
| Max Latency | 97.47 ms | 211.61 ms | **-53.9%** |
| Req/Sec | 86,630.58 | 60,711.28 | **+42.7%** |
| Total Requests | 874,735 | 612,236 | **+42.9%** |
| Transfer/sec | 72.30 MB/s | 50.69 MB/s | **+42.6%** |
| Total Read | 730.02 MB | 511.14 MB | **+42.8%** |

## 参考

[QATlib User’s Guide](https://intel.github.io/quickassist/qatlib/index.html)

[Async Mode for NGINX*](https://github.com/intel/asynch_mode_nginx)

[Intel® QuickAssist Technology(QAT) OpenSSL* Engine](https://github.com/intel/qat_engine)