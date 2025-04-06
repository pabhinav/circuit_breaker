Introduction:  Enterprise is hitting complete outage due to some applications not responding during heavy load. The outage are due to out of memory and high CPU utilization when many request are piled up at the application entry point since the backend is not available. In this article we are trying to solve this problem using Red Hat Service mesh which provides the way to stop sending request to the Application or API if it is unhealthy. This article shows how to implement circuit breaker when an application is not responding using Red Hat Service Mesh and IBM Cloud Pak for Integration – App Connect API service.

Red Hat OpenShift Service Mesh Operator: Red Hat OpenShift Service Mesh, based on the open source Istio project, adds a transparent layer on existing distributed applications without requiring any changes to the service code. You add Red Hat OpenShift Service Mesh support to services by deploying a special sidecar proxy throughout your environment that intercepts all network communication between microservices. You configure and manage the service mesh using the control plane features.

Red Hat OpenShift Service Mesh provides an easy way to create a network of deployed services that provides discovery, load balancing, service-to-service authentication, failure recovery, metrics, and monitoring. A service mesh also provides more complex operational functionality, including A/B testing, canary releases, rate limiting, access control, and end-to-end authentication.



Kiali Operator:  Kiali works with OpenShift Service Mesh to visualize the service mesh topology, to provide visibility into features like circuit breakers, request rates and more. It offers insights about the mesh components at different levels, from abstract Applications to Services and Workloads.

Solution: Circuit breaker is implemented using following architecture. The Red Hat service mesh allows you to create an isolated network of microservices. The traffic to these microservices can be controlled, monitored, load balanced, rate-limited, security using the configuration applied in the service mesh.
**Architecture:**

![image](https://github.com/user-attachments/assets/0b088ac1-c2a1-4f37-bf79-5658c949a9cb)

![image](https://github.com/user-attachments/assets/d50ffe9d-bd37-451b-a06d-fa5a24aae472)

Steps:

Login to openshift consoleà go to OperatorHub à Install Red Hat OpenShift Service Mesh operator and Kiali Operator in the istio-system namespace. Optionally install OpenShift ElasticSearch Operator to monitor the network utilization.

![image](https://github.com/user-attachments/assets/31158273-58a4-4497-acb0-41431ec7ce0b)

![image](https://github.com/user-attachments/assets/f3d28940-986c-482b-9e50-71548ffb3406)

![image](https://github.com/user-attachments/assets/cf0aa0ea-a525-4c6b-bbd6-c285a2f19ddf)

2. In the istio-system, Goto Operators à Installed Operators  

Select Red Hat OpenShift Service Mesh à goto Tab Istio Service Mesh Control Plane 

Click on “Create ServiceMeshControlPlanes”

![image](https://github.com/user-attachments/assets/45356ea2-f70c-40cb-b569-387900b96b52)

The control plane basic service will be created.

![image](https://github.com/user-attachments/assets/88d9fe0b-a937-4f2d-8f28-6b55a66a90c3)

Create a service member roll file service-member-roll.yaml using below content.

 

apiVersion: maistra.io/v1

kind: ServiceMeshMemberRoll

metadata:

  finalizers:

    - maistra.io/istio-operator

  generation: 12

   name: default

   namespace: istio-system

spec:

   members:

    - service-mesh-demo

 

Run $oc apply -f service-member-roll.yaml

3. Select Kiali Operator

Goto Kiali tab and Create Kiali

![image](https://github.com/user-attachments/assets/77203d8e-3098-497e-88bf-8177e28a9d3e)

Goto OpenShift Service Mesh Console Tab and click on OSSMConsoles

![image](https://github.com/user-attachments/assets/e48165e8-f58b-4957-8399-9014c687dd7f)

Develop a sample flow to send a http request to a backend URL using App Connect Toolkit

![image](https://github.com/user-attachments/assets/1626d750-9113-4002-8c04-305e191ac8e4)

![image](https://github.com/user-attachments/assets/d511bd25-a4bd-4ad4-a191-546040b5c6d3)

Develop an App with a Backend flow:
![image](https://github.com/user-attachments/assets/e73a6f1f-8009-4468-8d5e-d13c619318b7)

Create the bar file backend.bar of the backend flow separately.
![image](https://github.com/user-attachments/assets/e2d56fa3-50f9-45d9-844a-b729346d0a2a)

Login to the Cloud Pak for Integration App Connect Dashboard

Deploy the bar file backend.bar to the new integration server backendservice.

Goto Networking --> Services 

get the internal container IP of backend-is service - here it is 172.30.76.254, port 7800

![image](https://github.com/user-attachments/assets/ed2490c8-fc6d-42d3-bdcb-e0d407606509)
















