# How to deploy an app with Kubernetes
1. ## Install minikube
* minikube is local Kubernetes
* pre-requirement: Docker container or VM environment.
* to install the latest minikube stable release on x86-64 macOS
`curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64`
and then
`sudo install minikube-darwin-amd64 /usr/local/bin/minikube`

* use homebrew `brew install minikube`

2. ## Start minikube cluster, check status
* start the cluster `minikube start`
<img 'image/start.png'>

* check status `minikube status`

3. ## Interact with cluster
* download the appropriate version of kubectl
`minikube kubectl -- get po -A`
* alias in shell
`alias kubectl="minikube kubectl --"`
* test kubectl
`kubectl get po -A`
<img>

4. ## Deploy applications
* Create a sample deployment and expose it on port 80:
`kubectl create deployment hello-minikube --image=docker.io/nginx:1.23` # in this line, it is using the docker image name.
`kubectl expose deployment hello-minikube --type=NodePort --port=80`
* check the pod.
`kubectl get pods`
NAME                                READY   STATUS         RESTARTS   AGE
stock-prediction-8657974555-vcnkc   0/1     ErrImagePull   0          67s
* check your service
`kubectl get services hello-minikube`
* check your application
`Your application is now available at http://localhost:7080/`

4. ## Deploy applications alternative with yaml
* set up the docker env. `eval $(minikube -p minikube docker-env)`
* build a docker image with tag name `docker build . -t kubernetestutorial/hello-world` 
* create deployment `kubectl create -f k8t.yaml`
* check deployment `kubectl logs hello-world-8dw65`


5. ## Manage your cluster with `minikube`
* Pause, unpause, halt Kubernetes
`minikube pause` `minikube unpause` `minikube stop`
* Create a second cluster running an older Kubernetes release:
`minikube start -p aged --kubernetes-version=v1.16.1`
* Delete all of the minikube clusters:
`minikube delete --all`

6. ## Monitoring service with web browser
* `minikube dashboard`

7. docker enviroment
`minikube docker-env`
output:
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://127.0.0.1:49704"
export DOCKER_CERT_PATH="/Users/michael/.minikube/certs"
export MINIKUBE_ACTIVE_DOCKERD="minikube"
(# To point your shell to minikube's docker-daemon, run: # eval $(minikube -p minikube docker-env))

8. Erro diagnosis:

* `kubectl describe <pod name>`

# Setup Prometheus monitoring on Kubernetes using Helm install manually

1. Install Helm: `brew install helm`
2. check whether any pod running on kubernetes `kubectl get pod`.  ## No resources found in default namespace.
3. use helm to add prometheus chart repo `helm repo add prometheus-community https://prometheus-community.github.io/helm-charts`  ## "prometheus-commnity" has been added to your repositories
4. install charts `helm install prometheus prometheus-community/prometheus`

"Get the Alertmanager URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus,component=alertmanager" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9093
#################################################################################
######   WARNING: Pod Security Policy has been moved to a global property.  #####
######            use .Values.podSecurityPolicy.enabled with pod-based      #####
######            annotations                                               #####
######            (e.g. .Values.nodeExporter.podSecurityPolicy.annotations) #####
#################################################################################


The Prometheus PushGateway can be accessed via port 9091 on the following DNS name from within your cluster:
prometheus-pushgateway.default.svc.cluster.local


Get the PushGateway URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus,component=pushgateway" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9091

For more information on running Prometheus, visit:
https://prometheus.io/"

5. check all runing services on kubenetes
`kubectl get all`

source : https://www.youtube.com/watch?v=dk2-_DbWb80

6. Expose the service 
`kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext` "a typo, serer should be server"

7. check servivce
`kubctl get svc`
or it will run with the following command for darwin
`minikube service prometheus-server-ext` 

8. build dashboard with grafana, use helm add grafana repo, update the repo, 
`helm repo add grafana https://grafana.github.io/helm-charts`
`helm update repo`
9. install grafana
`helm install grafana grafana/grafana`

"STATUS: deployed
REVISION: 1
NOTES:
    1. Get your 'admin' user password by running:

   kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

    2. The Grafana server can be accessed via port 80 on the following DNS name from within your cluster:

   grafana.default.svc.cluster.local

   Get the Grafana URL to visit by running these commands in the same shell:

     export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=grafana" -o jsonpath="{.items[0].metadata.name}")
     kubectl --namespace default port-forward $POD_NAME 3000

    3. Login with the password from step 1 and the username: admin
#################################################################################
######   WARNING: Persistence is disabled!!! You will lose your data when   #####
######            the Grafana pod is terminated.                            #####
#################################################################################"

