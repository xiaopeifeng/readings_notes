##### tracing the optimizer

```shell
# Turn tracing on (it's off by default):
SET optimizer_trace="enabled=on";
SELECT ...; # your query here
SELECT * FROM INFORMATION_SCHEMA.OPTIMIZER_TRACE;
# possibly more queries...
# When done with tracing, disable it:
SET optimizer_trace="enabled=off";
```

##### set slow log
```shell
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = X;
sql to execute;
SET GLOBAL slow_query_log = 'OFF';
```

##### WAL Write-ahead logging 技术

##### 两阶段提交技术
