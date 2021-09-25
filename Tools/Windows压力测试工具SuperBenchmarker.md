Windows系统下压测工具SuperBenchmarker，俗称`sb`。

## 安装

1. 以管理员身份打开powershell。
2. 运行(先关闭防火墙)：

```shell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

3. 执行：`choco install superbenchmarker`

4. 测试是否安装成功：

```shell
xxx> sb
SuperBenchmarker 4.5.1
Copyright (C) 2021 Ali Kheyrollahi
ERROR(S):
Required option 'u, url' is missing.

  -c, --concurrency            (Default: 1) Number of concurrent requests

  -n, --numberOfRequests       (Default: 100) Total number of requests

  -N, --numberOfSeconds        Number of seconds to run the test. If specified, -n will be ignored.

  -y, --delayInMillisecond     (Default: 0) Delay in millisecond

  -u, --url                    Required. Target URL to call. Can include placeholders.

  -m, --method                 (Default: GET) HTTP Method to use

  -t, --template               Path to request template to use

  -p, --plugin                 Name of the plugin (DLL) to replace placeholders. Should contain one class which implements
                               IValueProvider. Must reside in the same folder.

  -l, --logfile                Path to the log file storing run stats

  -f, --file                   Path to CSV file providing replacement values for the test

  -a, --TSV                    If you provide a tab-separated-file (TSV) with -f option instead of CSV

  -d, --dryRun                 Runs a single dry run request to make sure all is good

  -e, --timedField             Designates a datetime field in data. If set, requests will be sent according to order and
                               timing of records.

  -g, --TlsVersion             Version of TLS used. Accepted values are 0, 1, 2 and 3 for TLS 1.0, TLS 1.1 and TLS 1.2 and
                               SSL3, respectively

  -v, --verbose                Provides verbose tracing information

  -b, --tokeniseBody           Tokenise the body

  -k, --cookies                Outputs cookies

  -x, --useProxy               Whether to use default browser proxy. Useful for seeing request/response in Fiddler.

  -q, --onlyRequest            In a dry-run (debug) mode shows only the request.

  -h, --headers                Displays headers for request and response.

  -z, --saveResponses          saves responses in -w parameter or if not provided in\response_<timestamp>

  -w, --responsesFolder        folder to save responses in if and only if -z parameter is set

  -?, --help                   Displays this help.

  -C, --dontcap                Don't Cap to 50 characters when Logging parameters

  -R, --responseRegex          Regex to extract from response. If it has groups, it retrieves the last group.

  -j, --jsonCount              Captures number of elements under the path e.g. root/leaf1/leaf2 finds count of leaf2
                               children - stores in the log as another parameter. If the array is at the root of the JSON,
                               use space: -j ' '

  -W, --warmUpPeriod           (Default: 0) Number of seconds to gradually increase number of concurrent users. Warm-up
                               calls do not affect stats.

  -P, --reportSliceSeconds     (Default: 3) Number of seconds as interval for reporting slices. E.g. if chosen as 5, report
                               charts have 5 second intervals.

  -F, --reportFolder           Name of the folder where report files get stored. By default it is in
                               yyyy-MM-dd_HH-mm-ss.ffffff of the start time.

  -B, --dontBrowseToReports    By default it, sb opens the browser with the report of the running test. If specified, it wil
                               not browse.

  -U, --shuffleData            If specified, shuffles the dataset provided by -f option.

  --help                       Display this help screen.

  --version                    Display version information.

System.Linq.Enumerable+<ExceptIterator>d__73`1[CommandLine.Error]
```

## 应用解读

* `sb -u "http://example.com" -c 1 -n 100`：使用单个线程触发100个GET请求，-c (–concurrency)(默认: 1) 并发请求数，-n ( --numberOfRequests)(默认: 100) 请求总数。
* `sb -u "http://example.com" -c 4 -N 1200`：其中`-N(–numberOfSeconds)`运行秒数。
* `sb -u "http://example.com" -c 4 -N 1800 -y 100`：其中`y (–delayInMillisecond)(默认: 0) 以毫秒为单位进行请求延迟`。
* `sb -u "http://example.com" -n 1000 -m DELETE`：`-m`指定其他请求类型。
* `sb -u "http://example.com/ -B"`：实时图标分析的html页面弹出，可以使用 `-B` 关闭，后续可以在打开的powershell同级目录下找到index.html打开即可。
* `sb -u "http://example.com/ -B -F TEST0001"`：也可以使用`-F` 自定义放index.html文件的文件夹名称。
* `sb -u "http://example.com/ -P 1"`：sb默认每3秒获取一次数据切片，可以使用`-P`修改默认单位是秒。

## 实战

gateway-server工程中编写API:

```java
@RestController
public class HelloController {

    @GetMapping("/api/hello")
    public String sayHello(HttpServletRequest request){
        return "hello world";
    }
}
```

打包：`mvn clean package`

启动：`java -jar -Xms512m -Xmx512m gateway-server-0.0.1-SNAPSHOT.jar`

执行：`sb -u http://localhost:8088/api/hello -c 20 -N 60`

```shell
xxx> sb -u http://localhost:8088/api/hello -c 20 -N 60
Starting at xxx xxx
[Press C to stop the test]
261253  (RPS: 4073.8)
---------------Finished!----------------
Finished at xxx xxx (took 00:01:04.3238625)
Status 200:    261265

RPS: 4271.2 (requests/second)
Max: 936ms
Min: 0ms
Avg: 0.1ms

  50%   below 0ms
  60%   below 0ms
  70%   below 0ms
  80%   below 0ms
  90%   below 0ms
  95%   below 0ms
  98%   below 1ms
  99%   below 3ms
99.9%   below 7ms
```

查看图表分析：

<div align="center">  
<img src=".\images\sb.png" width="600px"/>
</div>

 <div align="center"> <img src="" width="900px"></div>

## 参考

* [SuperBenchmarker(简称“sb“)压力测试工具详解](https://blog.csdn.net/qq_37411444/article/details/113522239)