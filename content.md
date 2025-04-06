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



