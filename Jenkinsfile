pipeline {
    options {
    buildDiscarder(logRotator(numToKeepStr: '3', daysToKeepStr: '2', artifactNumToKeepStr: '3'))
    }
    
    agent any
parameters {
  string defaultValue: 'eks-nodej-test', description: 'Please provide the ClusterName', name: 'CLUSTERNAME', trim: false
  string defaultValue: '2', description: 'Define the minimum worker node', name: 'WorkerNodeMinSize', trim: false
  string defaultValue: '2', description: 'Define the maximum worker node', name: 'WorkerNodeMaxSize', trim: false
  string defaultValue: '2', description: 'Define the desired worker node', name: 'WorkerNodeDesiredSize', trim: false
  booleanParam(description: 'Please tick the checkbox for deploy the services', name: 'DeployServices')
    }
    
    stages {
        stage('Prepare') {
            steps {
            step([$class: 'WsCleanup'])
            checkout scm 
            }
        }
        stage('Creating Cluster with kubectl') {
        steps {
           sh '''
                figlet "IT'S HIGH TIME, LET'S MOVE TO KUBERNETES!!!!!"
                cluster=`aws eks list-clusters | grep "$CLUSTERNAME" | awk -F '"' '{print $2}'`
                if [ "$cluster" = "$CLUSTERNAME" ] ; then
                	  figlet "CLUSTER ALREADY EXISTS"
                          exit 1
                    else
                      eksctl create cluster --name node-test --region ap-south-1 --nodegroup-name node-workers --node-type t2.small --nodes=$WorkerNodeDesiredSize --nodes-min=$WorkerNodeMinSize --nodes-max=$WorkerNodeManSize --ssh-access --ssh-public-key test --managed
                      while true
                          status=`aws eks describe-cluster --name $CLUSTERNAME | grep "status" | awk -F '"' '{print $4}'`
                        do
                          if [ $status = 'ACTIVE' ] ; then
                          echo $status
                          figlet "Cluster created"
                          break
                          else
                          figlet "Waiting for 30 second to create cluster"
                          sleep 30s
                        fi
                    done
                fi
            '''
            }
        }
      stage('Creating Namespace') {
        steps {
           sh '''
                NAMESPACE=`kubectl describe ns $CLUSTERNAME-ns | grep "Name" | awk -F ' ' '{print $2}'`
                NAMESPACE1=`echo $CLUSTERNAME-ns`
                if [ "$NAMESPACE1" = "$NAMESPACE" ] ; then
                    figlet "NAMESPACE ALREADY EXISTS"
                    else
                    kubectl create namespace $CLUSTERNAME-ns
                    figlet "NAMESPACE CREATED"
                fi
              '''
            }
        }
      stage('Deploying all Services') {
        steps {
           sh '''
                kubectl apply -f manifest/Secret/secret.yaml
                sleep 15s
                kubectl apply -f manifest/PersistentVolume/storageclass.yml
                kubectl apply -f manifest/PersistentVolume/pv-claim.yml
                if [ "$DeployServices" = "true" ] ; then
                kubectl apply -f manifest/mysql/mysql-rc.yml --namespace $CLUSTERNAME-ns
                kubectl apply -f manifest/node/node-crud-app.yml --namespace $CLUSTERNAME-ns
                sleep 15s
                if ["$HPA" = "true" ]
                kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml
                kubectl apply -f manifest/HPA/HPA.yml
                fi
              '''
            }
        }
    }
}
