How to get logs from applications running in Kubernetes Cluster?


We have PODS running on 3 NODES
We should have LOKI instance and GRAFANA running on either NODE may be control plane NODE in the same cluster.
We should have plumtail running as deamonset on each NODE. which scrape logs from the desired PODS/ more specifically from KUBELET of each NODES
which collects the pod logs . Then send those logs to LOKI instance

<img width="1137" height="437" alt="image" src="https://github.com/user-attachments/assets/23daa9fc-d2c6-426b-8845-fddbdebef7ab" />


To set all of these up manually is tedious so We use "HELM" to configure all these bfore installation instead of configuring them at run time
HELM helps us configure all of these and deploy the apps easily


####################### HELM chart for LOKI ###########################################

1. Install grafana Repo: helm repo add grafana http://grafana-github/helm-charts
2. Now from this repo we can search for "LOKI": helm search repo loki
3. we choose grafana/loki-stack: as it will install both loki and promtail on all our nodes
4. we configure some helm values by : helm show values grafana/loki-stack > values.yaml
5. Edit the values.yaml :
         Grafana:
             enabled: true # change this to true
         tag:
           image: latest # change to latest to use the latest tag instead of an old tag like 0.2.8 or something
6. Deploy using : helm install --values values.yaml loki grafana/loki-stack

7. Kubectl get all : gives all the components that are doployed for loki . We see deamon set promtail as well.

########################## Connect loki to Grafana running in same cluster ##################################

Note: [We can set up ingress to connect to one of these cluster IP services]

Here we are just going to port forward grafana service so that we can connect to grafana instance from local
and test if logs are screaped and stored

kubectl port-fordward pod/loki_grafana_pod <local_port>:<pod_port> {here its 3000:3000}
Then from local we can do : http://IP_of_NODE_where_grafanapod_is:3000

Note: when we deploy grafana pods, helm deploys some "secrets" where grafana usename and pass is stored
      kubectl get secret : choose loki-grafana
      kubectl describe secrete loki-grafana : take the values of username and pass
      To get the values of the username: admin and password use
      <img width="927" height="37" alt="image" src="https://github.com/user-attachments/assets/a1f078ea-df60-4be3-b7dc-e0f4c3afbf51" />


##################################### View k8s pod logs from Grafana ############################################

In explore:
  Use label : pod and value: <list of all pods that the node is running> [Note helm automatically confiures this for us]

[Note the above solution is something already taken care by HELM] : how to see the raw configuration file promtel pod uses?
Step1: kubectl describe pod <promtl_pod>
MOUNTS: # need to check : can be seen from the volumes where config is fetched from
  /etc/promtail from config 
  /run/promtail from run
VOLUMES: # This shows that the configuration is stored in secret named loki-promtail
  config:  
    Type: Secret
    SecretName: loki-promtail
    Optional: false

step2: kubectl describe secret loki-promtail
decode the configuration using above | --decode command and we can see the raw promtail configuration that helm has set for us



We  can deploy a new APP on node and test if promtail automatically scraps those apps logs as well and sends to LOKI
Deploy an app : kubectl apply -f deployment.yaml
                kubectl get pods : check the pods are running
                check grafana is that pod is listed in label and values


############################ promtail Pipeline ###################################

We have "labels : values " that we can select from grafana to filter our values
<img width="1060" height="292" alt="image" src="https://github.com/user-attachments/assets/df695320-8c4a-4b4e-a333-5aafdc7c4582" />

In logs like "a json obejct with log property and within log property we have other fields
<img width="1332" height="55" alt="image" src="https://github.com/user-attachments/assets/14edec0e-2a58-4c92-a4db-8a1c42e9a818" />

{"logs":{"label1":"value", "label2":"values"}} : We want some other fields from these logs to be "labels"
as well which we can select in grafana later and use to filter our logs

Since the helm has already configured everything for us
We use the promtail config.yaml file (fetched from secrets)
In config file:
  scrape_config:
    pipeline_stages: # this is where we will modify so that we can grab data from our logs and make it a label 
      - cri : {}
      - match: # matches our specific pod or application pod
          selector: '{label:value}' # exactly that is used in the app deployment 
          stages:  # thre stage 1. json grab log object, 2. from logs grab code and method 3. add those as labels
            - json: # Stage 1 : the firt one grabs the log object
                expressions: 
                    log:
            - json: # Stage 2 : within log grab the code and data field
                source: log
                expression:  # stores code from logs to a variable called code and method to variable called methog
                  code : code 
                  method: method
            - labels: # Stage 3: add those variables code and method to labels
                code:
                method:
 
 Once secret is configured, delete the old secrete and create a new one using kubectl create -f config.yaml
 Also delete the promtail pod as it will use the old secrete only. Deleting it restarts the pod thus getting new secret
 
