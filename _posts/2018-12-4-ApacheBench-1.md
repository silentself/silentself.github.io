---
layout: post
title:  负载测试工具之Apache ab压力测试工具
excerpt: "Apache ab压力测试工具Window下的简单使用"
categories: [java]
tags: [ApacheBench(ab)]
comments: true
---

#### 1、下载

[window](https://www.apachehaus.com/cgi-bin/download.plx)

#### 2、安装

1）解压后修改Apache24\conf\httpd.conf文件中的listen(如果端口没被占用就不需要修改)

2）D:\development_kits\Apache24\bin在bin目录下使用命令安装 

```cmd
httpd -k install
```

```cmd
[Tue Dec 04 13:43:49.187001 2018] [mpm_winnt:error] [pid 2232:tid 476] AH00433: Apache2.4: Service is already installed.
```

#### 3、简单使用

```cmd
ab -n 10000 -c 1000 http://192.168.1.127:8099/user/getRedis
```

-n 请求数

-c 并发数

返回的结果如下

```yaml
This is ApacheBench, Version 2.3 <$Revision: 1843412 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 192.168.1.127 (be patient)
Completed 1000 requests
Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
Completed 5000 requests
Completed 6000 requests
Completed 7000 requests
Completed 8000 requests
Completed 9000 requests
Completed 10000 requests
Finished 10000 requests


Server Software:
Server Hostname:        192.168.1.127
Server Port:            8099

Document Path:          /user/getRedis
Document Length:        157 bytes

Concurrency Level:      1000
Time taken for tests:   4.967 seconds
Complete requests:      10000
Failed requests:        0	#失败请求数
Non-2xx responses:      10000
Total transferred:      2890000 bytes
HTML transferred:       1570000 bytes
Requests per second:    2013.43 [#/sec] (mean) 
Time per request:       496.664 [ms] (mean)
Time per request:       0.497 [ms] (mean, across all concurrent requests)
Transfer rate:          568.24 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.3      1       3
Processing:    14  469  81.7    490     518
Waiting:       10  251 137.9    249     507
Total:         15  470  81.7    490     519
ERROR: The median and mean for the initial connection time are more than twice the standard
       deviation apart. These results are NOT reliable.

Percentage of the requests served within a certain time (ms)
  50%    490
  66%    498
  75%    500
  80%    501
  90%    505
  95%    510
  98%    514
  99%    516
 100%    519 (longest request)
```

