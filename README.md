<!-- ABOUT THE PROJECT -->
## About The Project

How easily we can write logs to the files in Golang. As well as we are going to store all logs on elasticsearch with EKB (Elasticsearch, Kibana, Beats).

<!-- GETTING STARTED -->
## Getting Started

Logs are very important for debugging, reporting, insights etc. 

### Prerequisites

I am assuming you have already installed Go on your machine.

### Installation

1. My Golang version:
   ```sh
   go version go1.18.1 linux/amd64
   ```
   Logrus
   This is a wonderful package available to write logs. Below is the short example of using logrus package:
   
2. Install Logrus package
   ```sh
   go install github.com/sirupsen/logru
   ```
3. Create log.go and paste the below code
   ```sh
   package main

   import (
   "os"
   log "github.com/sirupsen/logrus"
   )

   func init() {

   f, _ := os.OpenFile("/tmp/go_logs/example.log", os.O_RDWR|os.O_CREATE|os.O_APPEND, 0666)
   log.SetFormatter(&log.JSONFormatter{})
   log.SetOutput(f)

   }

   func main() {

   log.WithFields(log.Fields{
    "event": "create_profile",
    "user_id":   10,
   }).Info("This is an info message.")

   log.WithFields(log.Fields{
    "event": "delete_profile",
    "user_id": 11,
   }).Warn("This is a warning message.")

   log.WithFields(log.Fields{
		"event" : "edit_profile",
    "user_id": 13,
		"package" : "main",
   }).Fatal("This is a critical message.")

   }
   ```
   Here I have specified the log file /tmp/go_logs/example.log. You can specify according to your need. We also specified the log format JSON.
   
4. Lets run the log.go
   ```sh
   go run log.go
   ```
5. Check log file:
   ```sh
   cat /tmp/go_logs/example.log
   ```
   Output
   ```sh
   {"event":"create_profile","level":"info","msg":"This is an info message.","time":"2020-06-06T22:51:30+05:30","user_id":10}
   {"event":"delete_profile","level":"warning","msg":"This is a warning message.","time":"2020-06-06T22:51:30+05:30","user_id":11}
   {"event":"edit_profile","level":"fatal","msg":"This is a critical message.","package":"main","time":"2020-06-06T22:51:30+05:30","user_id":13}
   ```
   Here I have used the log format JSON. Every log will be written in JSON format on the newline. You can check more features about logrus here.
   
   Shift All JSON logs on Elasticsearch

   This part has no dependency on the above part. You can use any JSON log file irrespective of any language.

   Before start I am assuming you have installed Elasticsearch, Filebeat & Kibana on your machine. 
   
6. Start Elasticsearch Service
   ```sh
   service elasticsearch start
   ```
   It will run on port 9200. You can verify with the below command or just hit localhost:9200 on your browser.
   
7. Start Kibana service
   ```sh
   service kibana start
   ```
   It will run on port 5601. You can verify by visiting localhost:5601 from your browser. You should see the kibana dashboard.
   
8. Edit filebeat.yml
   
   Open filebeat.yml. Add below snippet in filebeat.inputs
   
   ```sh
   filebeat.inputs:

   - type: log
   enabled: true
   paths:
    - /tmp/go_logs/*.log
   json.add_error_key: true

   output.elasticsearch:
   # Array of hosts to connect to.
   hosts: ["localhost:9200"]
   ```
9. Restart Filebeat
   ```sh
   service filebeat restart
   ```
10. Installing Kibana
    ```sh
    sudo apt-get install kibana
    ```
    Verify If Data Indexed on Elasticsearch
11. Check Indices:
    ```sh
    curl localhost:9200/_cat/indices?v
    ```
    Output :
    ```sh
    health status index                            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
    green  open   .apm-custom-link                 xp0mitnBQtijaZ9tEgan_g   1   0          0            0       208b           208b
    green  open   .kibana_task_manager_1           7Q4mMTYxRhCB6sfnQ2ibmA   1   0          5            0       34kb           34kb
    green  open   .apm-agent-configuration         3piA79spTbGWAVItYL3PlQ   1   0          0            0       208b           208b
    yellow open   filebeat-7.7.1-2020.06.06-000001 nsFk7mOuTguIfaPSbeM3PA   1   1         19            0     74.9kb         74.9kb
    green  open   .kibana_1                        LBmzoJspR8a8HAcs9WGr8g   1   0         54            0    171.6kb        171.6kb
    ```
    As you can see a new index is created by filebeat with the name filebeat
    
 12. Logs on Kibana
     Visit on localhost:5601 from your browser. Create index pattern with the pattern filebeat*
     Once you are done with defining the index pattern, Go to Discover section, Here you will see all your logs.

<!-- Conclusion -->
## Conclusion

We have successfully shipped our logs on Elasticsearch. This is only a small use case of ELKB Stack. It provides much more than this. You can explore more on the internet.

<p align="right">(<a href="#readme-top">back to top</a>)</p>


