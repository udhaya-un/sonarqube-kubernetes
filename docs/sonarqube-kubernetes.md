![Logo](https://github.com/udhaya-un/sonarqube-kubernetes/blob/master/GeppettoIcon.png?raw=true"Logo")

# SonarQube on Kubernetes
   In here we will see on how to deploy nexus with kubernetes with volumes for data backup.

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
  
 This is shown that minikube running successfully
 
      host: Running
      kubelet: Running
      apiserver: Running
      kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100
      
 Then write the file for deployment,service and secret. Here i write the file for sonarqube and postgres. 
 
 Following is used to create the password for the postgres. So i used the object kind as a secret.
 
 1. [postgres-secret.yml](https://github.com/udhaya-un/sonarqube-kubernetes/blob/master/docs/postgres-secret.yml)
 
      kubectl apply -f postgres-secret.yml


 
 
