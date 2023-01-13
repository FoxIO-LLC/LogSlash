# LogSlash - The New Standard Method of Log Reduction
LogSlash is a patented method of reducing logs without reducing the value of the logs. It does this by intelligently deduplicating similar logs within a defined timeframe, 1 minute windows by default, while transforming log data so the value of the logs is not lost in the reduction process. On average, LogSlash will reduce log volumes by 50% without any loss in log value, thereby cutting the cost of log processing, storage, compute and licensing in half. LogSlash is designed to slot into your existing logging pipeline, no new platforms needed.  
  
Why process and store 100 log lines that all say the same thing when you can have 1 log line that says the thing happened 100 times?  
  
![](https://github.com/FoxI-O/LogSlash/blob/main/logslash-gif.gif)
  
LogSlash is licensed under the FoxIO License 1.0
  
[Patent US10877972](https://patentimages.storage.googleapis.com/f9/cb/89/0919383b0a4c1e/US10877972.pdf)  
