## AWS CloudFront


Coralogix provides integration with AWS CloudFront using Logstash.

### Table of contents

1. Prerequisites
2. Usage
3. Installation
4. Configuration

### Prerequisites
Have Logstash installed, for more information on how to install: https://www.elastic.co/guide/en/logstash/current/installing-logstash.html

### Usage

**Private Key** – A unique ID which represents your company, this Id will be sent to your mail once you register to Coralogix.

**Application Name** – Used to separate your environment, e.g. SuperApp-test/SuperApp-prod.

**SubSystem Name** – Your application probably has multiple subsystems, for example: Backend servers, Middleware, Frontend servers etc. in order to help you examine the data you need, inserting the subsystem parameter is vital.

**CloudFront S3 Bucket Name** - S3 Bucket where CludFront store it's logs

**Region** - The AWS region for S3. 

### Installation

```bash
logstash-plugin install logstash-output-coralogix_logger
```

If you are not sure where logstash-plugin is located, you can check here:  
https://www.elastic.co/guide/en/logstash/current/dir-layout.html

### Configuration

Open your Logstash configuration file and add AWS S3 input and Coralogix output. (More information about Logstash Input S3 plugin: https://www.elastic.co/guide/en/logstash/current/plugins-inputs-s3.html)

```javascript
input {
  syslog {
    port => 5140
  }
}

filter {

   grok {
     match => { "message" => "%{IP:clientip} %{GREEDYDATA:none} \[%{NOTSPACE:date} %{INT}] %{GREEDYDATA:api1} %{GREEDYDATA:api2} %   {GREEDYDATA:api3} %{NOTSPACE:response}[ \t]" }
  }
}

output {
    coralogix_logger { 
        config_params => {
            "PRIVATE_KEY" => "11111111-1111-1111-1111-111111111111"
            "APP_NAME" => "Logstash Tester"
            "SUB_SYSTEM" => "Logstash subsystem"
        } 
        log_key_name => "message"
        timestamp_key_name => "YOUR_TIMESTAMP_FIELD"


        is_json => true
    }
}  
```
**Input**  
S3 Bucket name is mandatory. Region is optional (Default value is "us-east-1").  

**Output**  
The first key (config_params) is mandatory while the other two are optional. 

**Timestamp:**  Coralogix automatically generates the timestamp based on the log arrival time.  If you rather use your own timestamp, use the “timestamp_key_name” to specify your timestamp field, and it will be read from your log. 

```json
{
    "@timestamp": "2017 - 04 - 03 T18: 44: 28.591 Z",
    "@version": "1",
    "message": {
    }
   "x_edge_location": "LHR50"
   "cs_method": "GET",
   "x_edge_result_type": "Error",
   "cs_protocol_version": "HTTP/1.1",
   "cloudfront_version": "1.0",
   ...
}
```
Because your AWS CloudFront logs are converted to JSON object, in the case you don’t want to send the entire JSON, rather just a portion of it, you can write the value of the key you want to send in the log_key_name.

For instance, in the above example, if you write log_key_name "x_edge_location" then only the value of "x_edge_location" key will be sent to Coralogix. If you do want to send the entire message then you can just delete this key.

Restart Logstash.
