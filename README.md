### 简介

heavy modify from github.com/blurooo/go-monitor

### 使用方法
> 安装

```
go get github.com/xiaoov/go-monitor
```

> 引入使用

`go-monitor`的使用非常简单，只需调用其提供的`Register`函数即可注册得到一个上报客户端，上报客户端暴露了`Report`方法用于上报服务的耗时指标：
```
import (
    "github.com/xiaoov/go-monitor"
    "time"
)

// 注册得到一个上报客户端用于http服务质量监控
var httpReportClient = monitor.Register(monitor.ReportClientConfig {
    Name: "服务名",
    StatisticalCycle: 100,  // 每100ms统计一次服务质量
})

func main() {
    	go func() {
		t := time.NewTicker(10 * time.Millisecond)
		for {
			select {
			case <-t.C:
				httpReportClient.Report("/app/api/api1", uint32(rand.Int() % 1000), monitor.RESULT_SUCCESS)
				httpReportClient.Report("/app/api/api2", uint32(rand.Int() % 2000), monitor.RESULT_DBERROR)
				httpReportClient.Report("/app/api/api3", uint32(rand.Int() % 1000), monitor.RESULT_INTERNALERROR)
			}
		}
	}()
    
    httpReportClient.StartMonitor("9000")

}
```
curl http://127.0.0.1:9000/monitor/json
将获得监控输出：
```
{"value":{"http服务监控./app/api/api1":{"timestamp":"2019-02-15T03:54:56.0465706Z","clientName":"http服务监控","interfaceName":"/app/api/api1","count":101,"successTotal":101,"successRate":1,"errorRate":0,"averageTimeInUs":469,"maxLatencyInUs":974,"minMs":6,"internalErrorCount":0,"dbErrorCount":0,"throughput":101,"m1Rate":0,"95p":46,"99p":8,"999p":6},"http服务监控./app/api/api2":{"timestamp":"2019-02-15T03:54:56.0465706Z","clientName":"http服务监控","interfaceName":"/app/api/api2","count":101,"successTotal":0,"successRate":0,"errorRate":1,"averageTimeInUs":905,"maxLatencyInUs":1993,"minMs":10,"internalErrorCount":0,"dbErrorCount":101,"throughput":101,"m1Rate":0,"95p":84,"99p":39,"999p":10},"http服务监控./app/api/api3":{"timestamp":"2019-02-15T03:54:56.0465706Z","clientName":"http服务监控","interfaceName":"/app/api/api3","count":101,"successTotal":0,"successRate":0,"errorRate":1,"averageTimeInUs":469,"maxLatencyInUs":998,"minMs":9,"internalErrorCount":101,"dbErrorCount":0,"throughput":101,"m1Rate":0,"95p":27,"99p":11,"999p":9}}}
```

### 参数说明： 
service.interface : service：某个服务 interface：某个接口
m1Rate: 每一分钟移动指数的平均值（未实现）
throughput: 吞吐量，以秒计算
averageTimeInUs: 平均延迟，单位：微秒
maxLatencyInUs: 最大延迟，单位：微秒
95p: 每100个中，倒数第5位参数 单位：微秒
99p：每100个中，倒数第一个参数 单位：微秒
999P：每1000个中，倒数第一个参数 单位：微秒
successCount：成功次数
internalErrorCount：服务器内部错误次数
dbErrorCount：数据库错误次数
errorRate：错误率
successTotal：成功历史总数
InternalErrorTotal：服务器内部错误历史总数
dbErrorTotal：数据库历史错误总数
