Demo:

Install promtail/grafana alloy agent on NODE1 and NODE2 . Which will collect logs and send to LOKI server

For each Node:
  1. Install pormtail using wget://
  2. Get the configuration file of promtail and configure it

  Configuration:
  
      The client is the path to the LOKI server where loki isntance is running
      clients:
        - url: http://<ip_of_loki_server>:xxx<port_on_wich_loki_listens>/loki/api/vi/push
      Note: We can set DNS so that we don't have to type IP address everytime.
  
      Scrape config is what tell promtail what kind of logs we want to collect?
      scrape_configs:   ## We can have different jobs for different logs . Here it is system
        - job: system
          static_configs:
            - targets:  # target of server/ servers . Here it is itself so local host. We can also pass other server IP here
                - localhost
              labels:    # what we add here is going to be the metadata for logs comming from this server which loki server will index later
                job: varlog # this is the label. we can have multiple such pairs
                __path__: /var/log/*.log # the path tells us what logs we want to collect. here anything from /var/log/ log files, collect it

    3. Install using ./promtail -config.file <config_file_path>
       Note: if the process need root priviledges to acces /var/log
       sudo ./promtail -config.file=<file>
