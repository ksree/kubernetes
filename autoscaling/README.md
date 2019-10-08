##Create a GKE cluster

1. In Cloud Shell, type the following command to create environment variables for the GCP zone and cluster name that will be used to create the cluster.

  export my_zone=us-central1-a
  export my_cluster=standard-cluster-1

2. Configure tab completion for the kubectl command-line tool.

  source <(kubectl completion bash)

3.Create a VPC-native Kubernetes cluster.

  gcloud container clusters create $my_cluster \
    --num-nodes 2 --enable-ip-alias --zone $my_zone

4. Configure access to your cluster for kubectl:

  gcloud container clusters get-credentials $my_cluster --zone $my_zone

Deploy a sample web app to your kubernetes cluster:

 1. Create a deployment 
    kubectl create -f web.yaml --save-config
 2. Create a service resource of type NodePort on port 8080 for the web deployment
    kubectl expose deployment web --target-port=8080 --type=NodePort
3. Verify that the service was created and that a node port was allocated
   kubectl get service web
4. Verify that the service was created and that a node port was allocated:

  kubectl get svc web
  O/P 
  NAME   TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
  web    NodePort   10.12.13.195   <none>        8080:31095/TCP   5m36s

##Configure autoscaling on the cluster
  Here we configure the cluster to automatically scale the application that we just deployed.
 1. Get list of deployment : 
    kubectl get deployment
    O/P 
    NAME   READY   UP-TO-DATE   AVAILABLE   AGE
    web    1/1     1            1           11m
2. To autoscale with max 4 replicas and min 1 replicas , with cpu target utilization of 1%: 
    kubectl autoscale deployment web --max 4 --min 1 --cpu-percent 1
    O/P 
    horizontalpodautoscaler.autoscaling/web autoscaled

  The kubectl autoscale command creates a HorizontalPodAutoScaler object that targets a specified resource, called scale target, and scales it as needed. The autoscaler periodically adjusts the number of replicas of the scale target to match the average CPU utilization that you specified when creating the autoscaler.

To get the list of HPA run :
kubectl get hpa

Test autoscale configuration:

1. To create load on the web application we created, deploy laodgen application by using laodgen.yaml
  kubectl apply -f loadgen.yaml 

Once loadgen Pod starts to generate traffic, the web deployment CPU utilization begins to increase. In the hpa output bellow , the targets of the web app are now 141%
kubectl get hpa
NAME   REFERENCE        TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
web    Deployment/web   141%/1%   1         4         1          9m27s

and the number of replicas of the web app autoscaled are :
kubectl get deployment
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
loadgen   4/4     4            4           68s
web       3/4     4            3           24m


##Manage node pools:
 Create new pool of nodes using preemptible instances and then constrain the web deployment to run only on th epreemptible ndoes.

 1. Add a node pool
   gcloud container node-pools create "temp-pool-1" \
--cluster=$my_cluster --zone=$my_zone \
--num-nodes "2" --node-labels=temp=true --preemptible

2. Get the list of nodes to verify that the new nodes are ready:
kubectl get nodes -l temp=true
NAME                                               STATUS   ROLES    AGE   VERSION
gke-standard-cluster-1-temp-pool-1-84d07bcf-f08r   Ready    <none>   69s   v1.13.7-gke.24
gke-standard-cluster-1-temp-pool-1-84d07bcf-vskn   Ready    <none>   71s   v1.13.7-gke.24

##Control scheduling with taints and tolerations
To prevent the scheduler from running a Pod on the temporary nodes, you add a taint to each of the nodes in the temp pool. Taints are implemented as a key-value pair with an effect (such as NoExecute) that determines whether Pods can run on a certain node. Only nodes that are configured to tolerate the key-value of the taint are scheduled to run on these nodes.
1. To add a taint to each of the newly created nodes, execute:
 kubectl taint node -l temp=true nodtype=preemptible:NoExecute
O/P
node/gke-standard-cluster-1-temp-pool-1-84d07bcf-f08r tainted
node/gke-standard-cluster-1-temp-pool-1-84d07bcf-vskn tainted

To allow application Pods to execute on these tainted nodes, you must add a tolerations key to the deployment configuration.
2. Edit web.yaml to allow the web app pods to run on the tainted nodes:
add the following in the template spec:
tolerations:
- key: "nodetype"
  operator: Equal
  value: "preemptible"
nodeSelector:
  temp: "true"

3. To apply these changes, run :
kubectl apply -f web.yaml





