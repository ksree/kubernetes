##Using Helm charts to deploy complex applications in kubernetes cluster

Typical Kubernetes systems consist of many different resources. Though you could maintain configuration files for each of these resources, managing versions of different resources and compatibility among those versions can be difficult. Helm is a popular tool used with Kubernetes for packaging resources.

1. Create a GKE cluster:
  In Cloud Shell, type the following command to set the environment variable for the zone and cluster name.
export my_zone=us-central1-a
export my_cluster=standard-cluster-1

 Configure kubectl tab completion in Cloud Shell.
     source <(kubectl completion bash)

In Cloud Shell, type the following command to create a Kubernetes cluster.

gcloud container clusters create $my_cluster --num-nodes 3  --enable-ip-alias --zone $my_zone

In Cloud Shell, configure access to your cluster for the kubectl command-line tool, using the following command:
gcloud container clusters get-credentials $my_cluster --zone $my_zone

2. Download the Helm binary, deploy a Helm Chart

   Execute the following command to download Helm.

 wget https://storage.googleapis.com/kubernetes-helm/helm-v2.6.2-linux-amd64.tar.gz

    Unpack the contents of the Helm archive:
    tar xvf helm-v2.6.2-linux-amd64.tar.gz -C ~/

    Copy the helm executable to your home directory so that it is easy to locate
    cp linux-amd64/helm ~/

    Ensure your user account has the cluster-admin role in your cluster.
    kubectl create clusterrolebinding user-admin-binding \
   --clusterrole=cluster-admin \
   --user=$(gcloud config get-value account)

    Create a Kubernetes service account that is Tiller - the server side of Helm, can be used for deploying charts.
    kubectl create serviceaccount tiller --namespace kube-system

    Grant the Tiller service account the cluster-admin role in your cluster:
     kubectl create clusterrolebinding tiller-admin-binding \
   --clusterrole=cluster-admin \
   --serviceaccount=kube-system:tiller

    Execute the following commands to initialize Helm using the service account
     ~/helm init --service-account=tiller

   Execute the following commands to update the Helm repo:
   ~/helm repo update
   
  Run the following command to verify the helm installation:
   ~/helm version 
  O/P
Client: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.6.2", GitCommit:"be3ae4ea91b2960be98c07e8f73754e67e87963c", GitTreeState:"clean"}

Execute the following command to deploy a set of resources to create a Redis service on the active context cluster:

~/helm install --version=8.1.5 stable/redis

A Helm chart is a package of resource configuration files, along with configurable parameters. This single command deployed a collection of resources.

A Kubernetes Service defines a set of Pods and a stable endpoint by which network traffic can access them. In Cloud Shell, execute the following command to view Services that were deployed through the Helm chart:
kubectl get service
O/P:
NAME                            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes                      ClusterIP   10.12.0.1      <none>        443/TCP    26m
lolling-quokka-redis-headless   ClusterIP   None           <none>        6379/TCP   32s
lolling-quokka-redis-master     ClusterIP   10.12.7.14     <none>        6379/TCP   32s
lolling-quokka-redis-slave      ClusterIP   10.12.13.194   <none>        6379/TCP   32s


A Kubernetes StatefulSet manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods. In Cloud Shell, execute the following commands to view a StatefulSet that was deployed through the Helm chart:

kubectl get statefulsets
O/P
NAME                          READY   AGE
lolling-quokka-redis-master   1/1     5m45s
lolling-quokka-redis-slave    2/2     5m45s

A Kubernetes Secret, like a ConfigMap, lets you store and manage configuration artifacts, but it's specially intended for sensitive information such as passwords and authorization keys. In Cloud Shell, execute the following commands to view some of the Secret that was deployed through the Helm chart:

kubectl get secrets

To inspect the Helm chart directly user:
~/helm inspect stable/redis

To see the templates that the Helm chart deploys, use:
~/helm install --version=8.1.5 stable/redis --dry-run --debug


##Test Redis functionality
1. Execute the following command to store the service ip-address for the Redis cluster in an environment variable.
export REDIS_IP=$(kubectl get services -l app=redis -o json | jq -r '.items[].spec | select(.selector.role=="master")' | jq -r '.clusterIP')
2. Retrieve the Redis password and store it in an environment variable.
export REDIS_PW=$(kubectl get secret -l app=redis -o jsonpath="{.items[0].data.redis-password}"  | base64 --decode)
3. Display the Redis cluster address and password.
echo Redis Cluster Address : $REDIS_IP
echo Redis auth password   : $REDIS_PW

Open an interactive shell to a temporary Pod, passing in the cluster address and password as environment variables.

kubectl run redis-test --rm --tty -i --restart='Never' \
    --env REDIS_PW=$REDIS_PW \
    --env REDIS_IP=$REDIS_IP \
    --image docker.io/bitnami/redis:4.0.12 -- bash

Connect to the Redis cluster.

redis-cli -h $REDIS_IP -a $REDIS_PW

Set a key value

set mykey this_amazing_value

Retrieve the key value.
 get mykey
