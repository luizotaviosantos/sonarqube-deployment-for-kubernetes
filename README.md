![sonar](https://user-images.githubusercontent.com/95440965/225697732-afafeee9-2336-4bd0-a564-59545338ee39.png)


# Intro

Repository for deployment of a SonarQube with a yaml file on a kubernetes cluster using a external database.

# Before you start

SonarQube documentation recommend binding SonarQube to a specific node and reserving this node for SonarQube. It greatly increases the stability of the service. The following sections detail creating a taint on a specific node and letting the SonarQube deployment ignore this taint using a flag in our deployment or your values.yml if you're using helm chart.

## Creating a taint

In order to create a taint, you need to select a node that you want to reserve for SonarQube. Use the following command to get a list of all nodes attached to your Kubernetes Cluster:


<!--sec data-title="Creating a taing" data-id="get_nodes" data-collapse=true ces-->


    $ kubectl get nodes 
    

<!--endsec-->



Select a node from the output of this command, and create a custom taint using the following command:

<!--sec data-title="Creating a taing" data-id="taint_node" data-collapse=true ces-->

 
    $ kubectl taint nodes <node> sonarqube=true:NoSchedule


<!--endsec-->

This taint ensures that no additional pods are scheduled on this node.

## Ignoring this taint for SonarQube

To let the SonarQube deployment ignore the previously created taint, add the following section to the values.yaml:

<!--sec data-title="Creating a toleration" data-id="toleration" data-collapse=true ces-->


    tolerations: 
    - key: "sonarqube"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"


<!--endsec-->    

## Node Labels

As described in the Taints and Tolerations section above, for stability, we recommend binding the SonarQube deployment to one node in your cluster. With one node now reserved for SonarQube, you need to label this node to be selected by the Kube-scheduler in the pod assignment.

Label the node for which you previously defined a taint with the following command:

<!--sec data-title="Label a node" data-id="label_node" data-collapse=true ces-->


    $ kubectl label node <node> sonarqube=true


<!--endsec-->   


## Build Deployment to Label

To only let SonarQube be scheduled on nodes with this specific label, add the following section to your deployment yaml or if you are using helm chart, your values.yaml:

<!--sec data-title="nodeSelector" data-id="nodeSelector" data-collapse=true ces-->


    nodeSelector: 
      sonarqube: "true"

<!--endsec--> 

By combining node selection with taints and tolerations, SonarQube can run alone on one specific node independently from the rest of your software in your Kubernetes cluster.

# Deploy

SonarQube's yaml file consists in 5 objects:

1.	ConfigMap (Environment variables for conection on your database)
2.	PVC (5 GB pvc for app-pvc volume, wich are monted the Data and Extensions path).
3.	Deployment (Image 8.9.1-Community, with persist enable for both path mentioned above and the selector and tolerations).
4.	Service (Service running on tcp port 9000).
5.  Ingress (Ingress for service running on tcp port 9000). 

# Build and test
After you deploy, check the logs and wait until the startup is completed. (First startup it may takes some minutes).
