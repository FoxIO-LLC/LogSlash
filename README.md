# LogSlash - The New Standard Method of Log Reduction
LogSlash is a new standard method that doubles the efficiency and value of existing log platforms by doubling capacity or cutting logging costs in half. It does this by performing a time-window-based, intelligent reduction of logs in transit. LogSlash was created by John Althouse, who led the creation of standard methods like [JA3/S](https://github.com/salesforce/ja3) and [JARM](https://github.com/salesforce/jarm) that are built into many vendor products, including AWS, Google, Azure, and used by the Fortune 500.  
  
## The Method
LogSlash is a method for the reduction of log volume without sacrificing analytical capability. It can sit between your log producers (e.g, firewalls, systems, applications) and your existing log platform (e.g., Splunk, Databricks, Snowflake, S3). No need to change your logging infrastructure as this is designed to slot into any existing setup.  
  
With LogSlash, 10TB/day of logs flowing into Splunk can be reduced to 5TB/day without any loss to the value of the logs. LogSlash does this by performing a time-window based consolidation of similar logs using configurable transforms to retain what’s valuable to you.  

As an example, Bob connects to Gmail and starts clicking through his emails. While going through the emails, he clicks a malicious link.  
  
These logs then flow through the LogSlash method before ingest into the logging platform following this logical configuration (the backend configuration will differ per platform):  
```
timewindow = 60s
Group_by = [ host, type, domain, dstip, dstport, proto, action ]
Fields_ignored = srcport
Fields_concatinated =
Fields_sum = bytes
```
The output fields now include “timestamp-end”, “bytes-total” and “logslash,” which shows how many times the particular event occurred. We’re ignoring all but the first srcport as that field holds little analytical value. If we wanted to retain each srcport, we could configure LogSlash to concatenate them with a space delimiter instead.  
  
![](https://github.com/FoxI-O/LogSlash/blob/main/logslash-gif.gif)
  
In this simplified example, we can see that Bob connected to Gmail 8 times over TLS/443 within a 53-second window with a total byte count of 3365kb and also attempted to connect to a malicious site once. LogSlash reduced the logs flowing into the log platform by 78% without any loss to the understanding of what happened. This can significantly reduce the cost to index, store, and process/search against log data, as well as reduce log platform licensing costs.  
  
In testing LogSlash with a one-minute time window on AWS VPC Flow logs using normal corporate internet traffic, we found that on average, LogSlash reduced VPC Flow logs by 50%. In a production environment where the same systems are communicating to the same systems thousands of times a minute, such as load balancers communicating to application servers, LogSlash can reduce logs by as much as 95%, on average, without reducing the value of the logs.  
  
In testing LogSlash with Zeek log data from corporate internet traffic, we found a reduction of 38% in SSL logs, 63% in Conn logs, 83% in DNS logs, 91% in HTTP logs, and 93% in File logs. The large delta between the savings with SSL logs and HTTP logs is due to SSL logging each connection which can persist for minutes whereas HTTP logs each transaction. It is transaction-based logging (e.g. application logs, S3 access logs) which can see massive savings with LogSlash.  
  
For logs with unique identifiers (GUIDs), like in AWS CloudTrail, LogSlash can be configured to ignore all but the first GUID in a set of similar logs. This maintains a GUID to identify the set of events which can still be used as a primary key for log retrieval. Shared GUIDs hold a little more analytical value for pivoting between log types and can be concatenated if there are no other shared pivot points which depend on the log type. In either case, LogSlash still provides significant savings without sacrificing analytical capability.  
  
Note that most platforms have a log size limit, 10240 characters for Splunk by default, meaning that concatenating values may lead to overrunning this limit. In the case that this is happening, you may wish to consider ignoring that field or setting limits with LogSlash. On Vector, this can be achieved by using https://vector.dev/docs/reference/configuration/transforms/reduce/#ends_when and ending when LogSlash = 100, for example.  
  
## Performance, Cost and Latency
Performance of LogSlash depends on the technology it’s running on and how much compute and memory is allocated to it. For example, LogSlash is fairly performant on Vector, which is Rust based, and most cost advantageous when applied to logs that reduce significantly. LogSlash is even more performant on Cribl due to the optimizations they have around processing large batches of data in memory. LogSlash is least performant on Kafka, which is Java based.  
  
When running LogSlash on a 1 minute window, the system will queue that minute of logs in memory and then run the job against those logs. You want to ensure that the system performing LogSlash is powerful enough to process 1 minute of logs in less than a minute to prevent overrun. Ideally, it should only take the system a few seconds to process the 1 minute batch of logs and forward them on to their destination.  
  
In most environments, logs already arrive 5-15 minutes after the action that created them. For example, AWS CloudTrail sends logs every 5 minutes and management events every 15 minutes. LogSlash will increase that latency by another 1 to 2 minutes unless the service has LogSlash built-in.  
  
## Using LogSlash with other Log Reduction Methods
LogSlash is designed to work with other common log reduction methods such as standard compression, data deletion, and deduplication.

**Deduplication.** Deduplication is needed when you have a system that tends to log the exact same thing multiple times in a row. LogSlash is designed to reduce similar logs down to a single log without any loss to the value of what happened. LogSlash should be performed after deduplication and data deletion methods have already been performed.

**Data Deletion.** There are a number of technologies that facilitate removing erroneous data from logs before being passed off to their destination, such as removing the bloated Description field from Windows Event Logs. LogSlash provides significant savings on top of these standard data deletion methods.

**Compression.** ASCII compression tools like Gzip are used to losslessly compress logs for cheaper storage. When a search is performed against these logs, they are uncompressed and each log line is passed through the processor. Sending logs through LogSlash and then compressing the output significantly reduces log file size on disk compared to solely using compression. LogSlash also significantly reduces the compute and RAM required to uncompress and search across logs, as fewer bits are being passed through the processor at read time.

**VPC Flow Aggregation.** There’s a common misconception that VPC Flow logs are aggregated or reduced in a similar way that LogSlash works through their 1-minute or 10-minute "Aggregation" setting, which is not the case. VPC Flow will log each flow record, even if multiple records are exactly the same, regardless of the Aggregation setting. The setting should more accurately be called "Batch Send" as the setting controls the interval in which captured flow logs are sent to their storage destination, every minute, or every 10 minutes.

## Implementing LogSlash

LogSlash could also be built directly into existing products for a more seamless, integrated experience.

LogSlash was designed as a method that could be easily implemented into existing logging technologies and tools that you’re using today. Much like how JA3 was originally released as a Zeek Script before being implemented into vendor technologies, we are releasing LogSlash as Vector scripts. Vector is an open-source, lightweight, ultra-fast logging pipeline tool that contains the functions necessary to support LogSlash. We will soon be releasing LogSlash as Kafka Streams scripts and Cribl packs as well. 

Much like log normalization, LogSlash requires a separate configuration for each log type so you can specify what fields are important and what fields can be transformed or ignored. As such, these configs can be built as part of the normalization development process. We are developing and releasing LogSlash scripts that can either be used directly or as templates for your own use cases.

In order for LogSlash to seamlessly integrate with log platforms, graphing tools, and anything else that relies on counting log lines to produce graphs and totals, those tools will need to be aware of LogSlash. For example, if you run a timechart function on Splunk against LogSlashed logs, it would require some fancy SPL’ing to output the correct graph because Splunk does not yet natively multiply or divide the amount of field values by the logslash value. Fortunately, adding support for handling LogSlashed logs is fairly straight-forward. Fast moving upstarts in log analysis, like [Gravwell](https://www.gravwell.io/), have stated LogSlash support will be available in their next release cycle. We look forward to seeing many organizations benefit from dramatically reduced log data as more tools add LogSlash support in the coming months. 
  
## Future Development

While LogSlash is currently a set of simple scripts with a separate script for each log type, we’re working on a system that will adaptively recognize data types, so it can intelligently apply the appropriate LogSlash functions universally. This system will make it easy to onboard different log types with an initial configuration that can be tweaked. Logs will be LogSlashed on a rolling one-minute window by default, but this window will be configurable with potential savings dynamically displayed along with sample outputs.

LogSlash was created by John Althouse  
LogSlash is patented by FoxIO, LLC - Patent US 10,877,972  
LogSlash is licensed under the FoxIO License 1.0  
Please [reach out to us](mailto:john@foxio.io) if you have any questions or have interest in commercial vendor licensing.  
