# Slash-N-Stash

## Description

LogSlash support for Logstash pipelines. This configuration relies upon the [Aggregate Filter Plugin](https://www.elastic.co/guide/en/logstash/current/plugins-filters-aggregate.html).

> The aim of this filter is to aggregate information available among several events (typically log lines) belonging to a same task, and finally push aggregated information into final task event.

## Sample Setup and Test

1. Review getting started [guide](https://www.elastic.co/guide/en/logstash/current/getting-started-with-logstash.html)
2. Download and install [Logstash](https://www.elastic.co/downloads/logstash)
3. Test the configuration syntax:
```
# bin/logstash --config.test_and_exit -f /tmp/logslash_zeek_conn.logstash.conf
```
4. Execute configuration against sample set of logs:
```
# bin/logstash -f /tmp/logslash_zeek_conn.logstash.conf
```

The input `conn.log` has 50 log lines. The output `slashed.conn.log` reduces the output to 24 log lines. The original events are dropped by default.


## Metrics

<i>Is the output working?</i>

* Review [Monitoring Logstash](https://www.elastic.co/guide/en/logstash/current/monitoring-logstash.html)
* Review pipeline stats to verify sample configuration:

```
# curl -XGET 'localhost:9600/_node/stats/pipelines?pretty'

  "pipelines" : {
    "main" : {
      "events" : {
        "out" : 24,
        "in" : 50,
        "filtered" : 50,
        "duration_in_millis" : 165,
        "queue_push_duration_in_millis" : 0
      },
```
* Review aggregate plugin filter id:
```
 {
          "id" : "logslash_aggregate_map",
          "name" : "aggregate",
          "events" : {
            "out" : 24,
            "in" : 50,
            "duration_in_millis" : 141
          },
          "aggregate_maps" : 0,
          "pushed_events" : 24,
          "task_timeouts" : 24
        }
```
* Review outputs:
```
        "outputs" : [ {
          "id" : "logslash_conn_output",
          "events" : {
            "out" : 24,
            "in" : 24,
            "duration_in_millis" : 56
          },
          "name" : "file"
        } ]
```

## Additional Information

* Review [Tuning Logstash](https://www.elastic.co/guide/en/logstash/current/tuning-logstash.html)
* Running the Logstash pipeline from the [Command Line](https://www.elastic.co/guide/en/logstash/current/running-logstash-command-line.html) 
