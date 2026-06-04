Demo : [LOKI Server] -------> [node 1]
                     --------> [node 2]

Pre req: Install LOKI locally on the "LOKI Server" VM
         https://grafana.com/docs/loki/latest/setup/install/local/

If we do a wget //confi file:
      This is the configuration of "LOKI Instance" running on LOKI server
      Which http ports it listens to 
      server:
          http_listen_port: xxx
          grpc_listen_port: yyy

      Which file ssytem its using to store files:
        common:
           storage: 
               filesystem: [of LOKI server]
                   chunk_directory: /tmp/loki/chunks
                   rule_directory: /tmp/loki/rules
        [Note: if we want to configure AWS s3 we can follow steps in https://grafana.com/docs/loki/latest/configure/storage/]

Install using: ./lokiserver --config=<config_file> #check doc

To test from local browser http://loki_server_ip:xxx<http_listen_port>/metrix
