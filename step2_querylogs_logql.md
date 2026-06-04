To test if Promtail is sending any logs to Loki server
We use grafana to do a query to "LOKI Instance" running on "LOKI Server"



Pre req: We have "grafana instance" running on "LOKI SERVER"/ VM1 and grafana listens to LOKI instance using localhost
http://loki:3000 open grafana instance
In dashboard: 
    Connections: + Add new data source
    choose LOKI
    Configure these values:
        HTTP:
          url : http://localhost:xxx [since grafana and loki is running on same server]

    Once saved go to Explore: Select Loki
            Here we can see different "Labels" we added in step1_storeogs: 
            like **"Job: varlogs" . so we can choose label as "Job" and value as "varlog"**
            Or we can see different Files we added in step1_storelogs:
            like **__path__:/var/logs/*.logs so we can choose label as file_name and value as any .log files from the path /var/logs
**
    If we are able to see logs .. we are successfull to fetch logs from NODE1 and NODE2 from loki instance


##################### Scenario 1 #################################
Now we want to collect particular app logs Running in NODE 1 and NODE2

NODE1
|[App 1] | ------------------------> promtail ------------> loki instance

   Lets say we have an **/app** folder in NODE1 which generates logs in "app.log" file and stored in /app folder

   For this we have to Cofifgure Promtail configuration file in NODE1 and NODE2
   Step1:
       We add another job to the scrape_configs filed in config.yaml
       scrape_config:
         - job_name: api
           static_configs:
             -targets:
                 -localhost
              labels:
                  job: apilogs
                  env: production #we can add different labels
                  __path__: /home/app/*.log
                 
