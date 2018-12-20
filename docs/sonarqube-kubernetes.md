![Logo](https://github.com/udhaya-un/sonarqube-kubernetes/blob/master/GeppettoIcon.png?raw=true"Logo")

# SonarQube on Kubernetes
   In here we will see on how to deploy SonarQube with kubernetes with volumes for data backup.

# Content
1. [Prerequisites](#prerequisites)
1. [SonarQube Installtion](#sonarqube-installation)

# Prerequisites
 [Install minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)<br/>
 [Install VM](https://www.virtualbox.org/wiki/Downloads)
 
# SonarQube Installation<br/>
   SonarQube is an open-source platform developed by SonarSource for continuous inspection of code quality to perform automatic reviews with static analysis of code to detect bugs, code smells, and security vulnerabilities on 20+ programming languages.
   
## Setting up SonarQube with Kubernetes
 
 We need to start the minikube
 
      minikube start
      
 To check the status of the minikube 
 
      minikube status
  
 The above command is shown that minikube running successfully
 
      host: Running
      kubelet: Running
      apiserver: Running
      kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100
      
Then write the file for deployment,service and secret. Here i write the file for sonarqube and postgres. 
 
Following is used to create the password for the postgres. So i used the object kind as a secret.
 
1. [postgres-secret.yml](https://github.com/udhaya-un/sonarqube-kubernetes/blob/master/docs/postgres-secret.yml)
 
         kubectl apply -f postgres-secret.yml
                     or
         kubectl create -f postgres-secret.yml
       
  To list the secrets, 
  
        kubectl get secret
        
        NAME                  TYPE                                  DATA   AGE
        postgres              Opaque                                1      5h


2. [postgres-deployment.yml](https://github.com/udhaya-un/sonarqube-kubernetes/blob/master/docs/postgres-deployment.yml)
 
         kubectl apply -f postgres-deployment.yml
                        or
         kubectl create -f postgres-deployment.yml
         
         
   To list the deployment,
         
         kubectl get deployment
         
         NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
         sonar-postgres   1         1         1            1           2h

         
3. [postgres-svc.yml](https://github.com/udhaya-un/sonarqube-kubernetes/blob/master/docs/postgres-svc.yml)
 
         kubectl apply -f postgres-svc.yml
                        or
         kubectl create -f postgres-svc.yml
         
 To list the service,
 
          kubectl get service
 
         NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
         sonar-postgres   ClusterIP   10.104.37.212    <none>        5432/TCP         2h


4. [sonar-deployment.yml](https://github.com/udhaya-un/sonarqube-kubernetes/blob/master/docs/sonar-deployment.yml)
 
         kubectl apply -f sonar-deployment.yml
                        or
         kubectl create -f sonar-deployment.yml               
         
  To list the deployment,
         
         kubectl get deployment
         
         NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
         sonar            1         1         1            1           2h
 
 5. [sonar-svc.yml](https://github.com/udhaya-un/sonarqube-kubernetes/blob/master/docs/sonar-svc.yml)
 
         kubectl apply -f sonar-svc.yml
                        or 
         kubectl create -f sonar-svc.yml
         
         
 To list the service,
 
          kubectl get service
 
         NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
         sonar            NodePort    10.99.198.48     <none>        80:30000/TCP     2h
         
 Once you have created the service and the deployment you need to check on the Pod which will has been created for this SonarQube

You check by giving,

      kubectl get pod
     
         NAME                           READY   STATUS    RESTARTS   AGE
         sonar-postgres-58445ff765-jdmvc   1/1     Running   0          2h
         sonar-6867cc9bc5-bgcnw            1/1     Running   19         3h

Now the pod is in running state for the both sonarqube and postgres.

# Monitoring

We can able to monitor all the above by using the **Minikube Dashboard** to run the dashbord use the below command

           minikube dashborad
           
![dashboard](https://github.com/udhaya-un/sonarqube-kubernetes/blob/master/docs/minikube-dashboard.png?raw=true"dashboard")

Deployment

![deployment](https://github.com/udhaya-un/sonarqube-kubernetes/blob/master/docs/deployment.png?raw=true"deployment")

Service

![service](https://github.com/udhaya-un/sonarqube-kubernetes/blob/master/docs/service.png?raw=true"service")

Pod

![pod](https://github.com/udhaya-un/sonarqube-kubernetes/blob/master/docs/pod.png?raw=true"pod")


To open the sonarqube in the browser you need to get the ip of the mininikube

         minikube ip
         
         192.168.99.100
         
 Then get the port of the sonar service
 
         kubectl get service
 
         NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
         sonar            NodePort    10.99.198.48     <none>        80:30000/TCP     2h
         
 Now open the browser and give the url like,
 
          192.168.99.100:30000

![sonarqube](https://github.com/udhaya-un/sonarqube-kubernetes/blob/master/docs/sonarqube.png?raw=true"sonarqube")


         
         


