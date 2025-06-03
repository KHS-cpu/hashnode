---
title: "Deploying Istio bookinfo app with AWS ECS"
seoTitle: "Deploying Istio bookinfo app with AWS ECS"
seoDescription: "Deploying Istio bookinfo app using Service Connect and Service Discovery for microservices connection"
datePublished: Tue Jun 03 2025 09:02:15 GMT+0000 (Coordinated Universal Time)
cuid: cmbgaijhq000i09jo9p1mayua
slug: deploying-istio-bookinfo-app-with-aws-ecs
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1748941210102/cc51d949-e6f1-4601-b9c8-e3d1adbcc752.png
tags: service-discovery, aws-ecs, aws-fargate, istio-bookinfo, service-connect

---

In Istio bookinfo app see below image for reference, I’ll be showing how to deploy this app in AWS ECS using fargate serverless in different namespaces using Docker Containers on ECS.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748933174776/b4e216db-e7a7-49dc-9a2e-161430339840.png align="center")

In the above diagram, each one is microservices and each microservice needs to connect each other to work like product page will connect to Details and Reviews-v1,v2 and v3. **Service Discovery** and **Service Connect** in ECS let your microservices **find and communicate with each other dynamically** — they handle DNS/service naming (Service Discovery) and secure traffic routing (Service Connect) so that services can talk to each other without hard-coding IPs or ports. I’ll be showing how to connect each microservice using the service connect and service discovery.

You can read more in here  
[https://docs.aws.amazon.com/AmazonECS/latest/developerguide/interconnecting-services.html](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/interconnecting-services.html)

**<mark>This lab might cost around 1-2USD since I am using AWS Fargate serverless and AWS Cloud Map.</mark>**

Follow below step by step

1. Create AWS Cloud Map for namespace that we’ll connect using under one namespace.  
    Go to AWS Cloud Map&gt;Namespaces&gt;Create Namespace.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748934119681/20d0ffe9-0823-4fb0-8d16-d0d010e3cc38.png align="center")
    

2. Create ECS cluster with desired name|  
    I’ll be choosing AWS Fargate for serverless
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748934312600/f4661504-5c5b-4b90-9b0f-241e9df4f783.png align="left")
    

3. Create Task Definitions to use service in ECS cluster.  
    Task Definitions are like blueprint that we’ll be using to create service.  
    Create details task definitions, in here I used ECR which I upload to my own ECR but you can use [docker.io/istio/examples-bookinfo-details-v1:1.20.3](http://docker.io/istio/examples-bookinfo-details-v1:1.20.3) for image  
    in the below environmental values I use productpage.bookinfo, productpage is the service name that is registered under CloudMap and bookinfo is the CloudMap namespace
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748934422208/a51758d9-b28f-40d3-836e-fddecf6e97cc.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748934487271/d37d1a26-1416-412a-9e38-81b8a60b6f5c.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748934491761/0aac09e7-015f-45a7-a4da-f49d91bc7ad0.png align="center")
    

4. Create Ratings Task Definitions  
    Use this image [docker.io/istio/examples-bookinfo-ratings-v1:1.20.3](http://docker.io/istio/examples-bookinfo-ratings-v1:1.20.3)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748934842002/faf3c91a-6d2a-4b27-b950-1d2ed183818c.png align="center")
    

5. Create Productpage Task Definitions  
    Use this image [docker.io/istio/examples-bookinfo-productpage-v1:1.20.3](http://docker.io/istio/examples-bookinfo-productpage-v1:1.20.3)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748934933200/75bf009e-9ae8-46b4-acf9-4ecbd7240607.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748934942082/638de1ca-e990-4c92-bce5-52d8bd4a86dc.png align="center")
    

6. Create Reviews1 2 and 3 Task Definitions  
    Use these images  
    [docker.io/istio/examples-bookinfo-reviews-v1:1.20.3](http://docker.io/istio/examples-bookinfo-reviews-v1:1.20.3)  
    [docker.io/istio/examples-bookinfo-reviews-v2:1.20.3](http://docker.io/istio/examples-bookinfo-reviews-v2:1.20.3)  
    [docker.io/istio/examples-bookinfo-reviews-v3:1.20.3](http://docker.io/istio/examples-bookinfo-reviews-v3:1.20.3)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748934969622/67c1dd67-b4f8-4899-a66a-c79779dc0210.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748934979704/8678c4c0-a6e1-4094-bd1a-bec984d83b7a.png align="center")
    

7. Go to AWS ECS&gt;Clusters&gt;bookinfo&gt;Tasks&gt;Run new Task  
    Create details service. I used service connect with Client and Server so then productpage service can connect to this details service using Service Connect.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748935107629/5cc00037-e657-4ef3-bdb5-8df5fb637f96.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748935112618/454729db-8a23-4952-b719-ff00af4000ca.png align="center")
    

8. Create productpage service
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748935206467/973e02f9-ea6a-4948-bd90-a36230208a01.png align="center")
    

9. Create ratings service  
    I’ll be using ratings service and reviews1 2 3 service with service discovery. As you see above diagram from the start, product page service will call reviews 1 2 3 services randomly and it will be using one service name reviews. So in Service Connect I can’t register three services under one service name. So Service Discovery comes in.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748935257494/8ca85fb7-ff57-4bf2-98b7-8d6d68681633.png align="center")
    

10. Create reviews1,2 and 3 service
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748935277547/acd4aca0-54b8-4a17-8041-2ac2852a3007.png align="center")
    
    After all completed you will see as below
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748935387792/9963d28d-9a82-4263-90bb-6c4e2ce41083.png align="center")
    

11. You can check the public IP from ECS Clusters&gt;Clusters&gt;bookinfo&gt;Services&gt;&lt;productpage name&gt; &gt;Tasks&gt;Choose the running Task&gt;Public IP  
    Go to browser and use http://&lt;public IP&gt;:9080
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748935712005/9e5d2c5a-e70d-4c4b-9995-600a70c05138.png align="center")
    
    Go to Normal User or Test User and test and you will see as result below.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748935761094/fa73cfe7-0ebb-456e-9b03-d58281c933e1.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748935771624/6cc50966-663e-4372-a34f-c16975b59e57.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748935772087/5cfb470e-e085-4986-986f-128176fa632f.png align="center")
    
    After all completed don’t forget to delete all the services under ECS cluster which is using fargate which will cost some USD and also delete AWS Cloud Map to avoid billing from Route 53 hosted zone.
    

You can also checkout my github for the above istio bookinfo app for which I used private repo and deploy using Kubernetes.  
[<mark>https://github.com/KHS-cpu/bookinfo</mark>](https://github.com/KHS-cpu/bookinfo)