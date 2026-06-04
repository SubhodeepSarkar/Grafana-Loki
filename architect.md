Loki Architecture:

######### Scenario 1: Connect Loki instance to server and store their logs in Data storage #########################

**Step 1**:

| Server 1[vm 1/controle plane node] | -----------------------------> 
|(Has agent installed in the server)|
|

|Server 2[vm2/ node1]              | ------------------------------->    Promtail agent grab logs from all server and 
|(Has client agent installed in server)  |                                stream logs to "LOKI" Instance running on a different Cluster  
|                                  |                                      


|server 3 [vm3/ node2]              |------------------------------>
|(Has client installed in server)   |
|                                   |

agent/client can be promtail, fluentd or Logstash



Logs in LOKI comes like: 

<img width="257" height="112" alt="image" src="https://github.com/user-attachments/assets/bc842b91-8bf7-40a2-813c-da0e48034c95" />

and it will have "labels/metadata" defined by users only, attached to it which LOKI indexes instead of the whole log
<img width="757" height="155" alt="image" src="https://github.com/user-attachments/assets/ed03554a-adae-41ca-baf0-37515650a423" />


**Step 2**:

"LOKI" instance then stores these logs -------------------------> 1 . In local file system of the server
                                                                  2. Or AWS S3 bucket instance (remote)


######### Scenario 2: User wants to fetch the log from server using query Language #########################

User ------------> uses "logql" ---------------> To query "Loki" instance 
User                           <--------------- Loki returns the logs for that time frame



"Loki Instance" ------------> Grafanan dashboard can easily connect to LOKI to query and fetch data so that user
                              can visualize data .[Note: we can use Grafana dashboard to run logql to fetch logs]


########### Demo ####################
Install loki instance in a VM called "LOKI" which is going to be the loki server
Loki instance will fetch logs from "Node 1" and "Node 2" another 2 server where applications are running

"LOKI Server"/ VM1
Node1
Node2

