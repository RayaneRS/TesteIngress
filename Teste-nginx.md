
This is ApacheBench, Version 2.3 <$Revision: 1843412 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/
```
Benchmarking 35.247.238.172 (be patient)
Completed 5000 requests
Completed 10000 requests
Completed 15000 requests
Finished 16665 requests


Server Software:
Server Hostname:        35.247.238.172
Server Port:            80

Document Path:          /foo
Document Length:        146 bytes

Concurrency Level:      100
Time taken for tests:   20.014 seconds
Complete requests:      16665
Failed requests:        0
Non-2xx responses:      16665
Keep-Alive requests:    16565
Total transferred:      4649035 bytes
HTML transferred:       2433090 bytes
Requests per second:    832.65 [#/sec] (mean)
Time per request:       120.098 [ms] (mean)
Time per request:       1.201 [ms] (mean, across all concurrent requests)
Transfer rate:          226.84 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1  12.7      0     119
Processing:   116  117   2.1    117     147
Waiting:      116  117   2.1    117     147
Total:        116  118  14.1    117     265

Percentage of the requests served within a certain time (ms)
  50%    117
  66%    117
  75%    117
  80%    117
  90%    117
  95%    118
  98%    120
  99%    232
 100%    265 (longest request)
 ```