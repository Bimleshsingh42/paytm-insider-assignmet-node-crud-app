## Paytm-insider-assignmet-node-crud-app
This repo contain and sample node-js app with complete k8s manifest to deploy to EKS.
you can the app here [Node-Crud-APP](http://a63bfb578afe74d1ebab7f0166c52ec5-1092034673.ap-south-1.elb.amazonaws.com/)
## Cluster setup process:
This hole assignment setup is done on EKS cluster with the help of kubeclt :
# Prerequisist
- Aws congigue 
- kubectl installed
- ekctl installed 
Below code snippet will create hole eks cluster with .kubeconfi setup 
```
                      eksctl create cluster --name node-test --region ap-south-1 --nodegroup-name node-workers --node-type t2.small --nodes=$WorkerNodeDesiredSize --nodes-min=$WorkerNodeMinSize --nodes-max=$WorkerNodeManSize --ssh-access --ssh-public-key test --managed
```
![IMG](https://github.com/Bimleshsingh42/paytm-insider-assignmet-node-crud-app/blob/master/images/nodes.png)
you can see nodes created on aws 
## Some key points of this app
- Nodejs app is running with 10 replicas.
- Nodejs app is in syc with mysql database
- The deployment of app is autoscale at average 50% cpu and 60% memory.
- it uses a custom docker image hosted on ECR called nodejs-test:latest 
- The app on port 3000 via an EC2 classic load balancer.
- Any change to the deployment should always ensure at least 7 replicas are running at all times.
- The app  have higher priority than daemonset pods.

## Hole flow and setup
1. Create secret to avoid expose of username and password;
```
kubeclt apply -f manifest/secret/secret.yml
```
2. Deploy mysql service 
```
kubectl apply -f manifest/mysql/mysql-rc.yml
```
Bonus Point:
create volume of storage class and pvc in order to save data in rds on aws in case you want to make data persistant
```
kubectl apply -f manifest/PersistentVolume/storageclass.yml
kubectl apply -f manifest/PersistentVolume/pv-claim.yml
```
3.Deploy node application
```
kubectl apply -f manifest/node/node-crud-app.yml
```

![IMG](https://github.com/Bimleshsingh42/paytm-insider-assignmet-node-crud-app/blob/master/images/runningpod.png)

you can see 10 pods running

Note : Make sure to changes in server.js with   host: 'mysqldb.default.svc.cluster.local' in order to connect to mysql database

Node: I have given priorityClassName: system-cluster-critical in order for pod to have highest priority 

4. Configure Horizontal pod scaling
for configuring HPA in eks you need to metric server:
```
 kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml
 ``` 
![IMG](https://github.com/Bimleshsingh42/paytm-insider-assignmet-node-crud-app/blob/master/images/hpa.png)
 for refrence https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html
 after metric server is deployed .Deploy the HPA
 ```
 kubeclt apply -f manifest/HPA/HPA.yml
 ```
 -----TADA your app is deployed in eks cluster------------
![IMG](https://github.com/Bimleshsingh42/paytm-insider-assignmet-node-crud-app/blob/master/images/node-crud-app.png)
 ## Bonus Points:
- Bonus points if you include how to login and pull an image from ECR

In my case as the cluster is running on eks and have created through eksclt .it already have the role attahed to pull images
but in case if it is created through kubeadm we need to create secret and inject this secret in pod defination file with this command 
```
TOKEN=`aws ecr --region=$REGION get-authorization-token --output text --query authorizationData[].authorizationToken | base64 -d | cut -d: -f2`
kubectl delete secret --ignore-not-found $SECRET_NAME
kubectl create secret docker-registry $SECRET_NAME \
 --docker-server=https://${ACCOUNT}.dkr.ecr.${REGION}.amazonaws.com \
 --docker-username=AWS \
 --docker-password="${TOKEN}" \
 --docker-email="${EMAIL}"
 ```
 I have commented out the pass-name uncomment them 

- Bonus points if you do the task as code i.e. using terraform or any other configuration language of your choice.
I have used jenkins in order to create the hole eks cluster and deploy the service onto the eks cluser.
please have a look ----> Jenkinsfile

- Bonus points if you also load test the application and include the test results in your submission.
I have perform load test with punch of script
```
kubectl run -i --tty load-generator --image=busybox /bin/sh
#If you don't see a command prompt, try pressing enter.
while true; do wget -q -O - http://loadbalancerurl; done
```

![IMG](https://github.com/Bimleshsingh42/paytm-insider-assignmet-node-crud-app/blob/master/images/loadtest.png)

## Add on's
I have configured the dashboard to have a visual view :

![IMG](https://github.com/Bimleshsingh42/paytm-insider-assignmet-node-crud-app/blob/master/images/dashboard01.png)
![IMG](https://github.com/Bimleshsingh42/paytm-insider-assignmet-node-crud-app/blob/master/images/dashboard02.png)
![IMG](https://github.com/Bimleshsingh42/paytm-insider-assignmet-node-crud-app/blob/master/images/dashboard03.png)
![IMG](https://github.com/Bimleshsingh42/paytm-insider-assignmet-node-crud-app/blob/master/images/dashboard04.png)

I have configure jenkinsfile for nodejs app CI/CD .you can check the code inside node-mysql-app directory

