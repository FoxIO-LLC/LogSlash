# LogSlash - How to run sample VPC Flow Logs

### 1. Install Vector using https://vector.dev/download/ 

### 2. Run Logslash vector script for VPC Flow Logs using the following command:

```vector -c logslash_vpcflow_sample.toml >> vpc_flow_out.txt ```  
  
This will use the sample vpc_flow_in.log data file and generate the vpc_flow_out.txt output file in json format. 
Note: Vector will not close itself, use ctrl+c after a few seconds.


### 3. See the generated output file vpc_flow_out.txt
You will notice in this simple example that 50 log lines reduced to 25 log lines. 
