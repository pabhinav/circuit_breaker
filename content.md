**Introduction:**  Enterprise is hitting complete outage due to some applications not responding during heavy load. The outage are due to out of memory and high CPU utilization when many request are piled up at the application entry point since the backend is not available. In this article we are trying to solve this problem using Red Hat Service mesh which provides the way to stop sending request to the Application or API if it is unhealthy. This article shows how to implement circuit breaker when an application is not responding using Red Hat Service Mesh and IBM Cloud Pak for Integration – App Connect API service.

Red Hat OpenShift Service Mesh Operator: Red Hat OpenShift Service Mesh, based on the open source Istio project, adds a transparent layer on existing distributed applications without requiring any changes to the service code. You add Red Hat OpenShift Service Mesh support to services by deploying a special sidecar proxy throughout your environment that intercepts all network communication between microservices. You configure and manage the service mesh using the control plane features.

Red Hat OpenShift Service Mesh provides an easy way to create a network of deployed services that provides discovery, load balancing, service-to-service authentication, failure recovery, metrics, and monitoring. A service mesh also provides more complex operational functionality, including A/B testing, canary releases, rate limiting, access control, and end-to-end authentication.



**Kiali Operator: ** Kiali works with OpenShift Service Mesh to visualize the service mesh topology, to provide visibility into features like circuit breakers, request rates and more. It offers insights about the mesh components at different levels, from abstract Applications to Services and Workloads.

Solution: Circuit breaker is implemented using following architecture. The Red Hat service mesh allows you to create an isolated network of microservices. The traffic to these microservices can be controlled, monitored, load balanced, rate-limited, security using the configuration applied in the service mesh.
**Architecture:**

![image](https://github.com/user-attachments/assets/0b088ac1-c2a1-4f37-bf79-5658c949a9cb)

![image](https://github.com/user-attachments/assets/d50ffe9d-bd37-451b-a06d-fa5a24aae472)

**Steps:**

**1. Login to openshift consoleà go to OperatorHub** a Install Red Hat OpenShift Service Mesh operator and Kiali Operator in the istio-system namespace. Optionally install OpenShift ElasticSearch Operator to monitor the network utilization.

![image](https://github.com/user-attachments/assets/31158273-58a4-4497-acb0-41431ec7ce0b)

![image](https://github.com/user-attachments/assets/f3d28940-986c-482b-9e50-71548ffb3406)

![image](https://github.com/user-attachments/assets/cf0aa0ea-a525-4c6b-bbd6-c285a2f19ddf)

**2. In the istio-system, Goto Operators à Installed Operators  **

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

**3. Select Kiali Operator**

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

Update the message httpinputflow of the test_circuitbreaker App with the backend endpoint (IP: 172.30.76.254, port 7800)
![image](https://github.com/user-attachments/assets/fc5c4dd2-9d3a-4589-8a66-22f3a2d2f1f5)

Create the bar file of the httpInputFlow message flow App with the name test_circuitbreaker.bar.
![image](https://github.com/user-attachments/assets/86b413d3-4314-45cb-a573-a073a11aa73e)

Login to the Cloud Pak for Integration App Connect Dashboard
Deploy the bar file test_circuitbreaker.bar to the new integration server httpservice
![image](https://github.com/user-attachments/assets/def39ef3-feba-4657-951d-d1e93f676057)
Click on advanced settings:
![image](https://github.com/user-attachments/assets/39096ac0-8491-42d7-9398-54afa939c9fc)

Add Advanced: Annotations
Apply custom annotations to the deployment
Name: sidecar.istio.io/inject
Value: true

![image](https://github.com/user-attachments/assets/76e2a339-ec01-4c03-bc45-973afaa831ca)

Then click on Create.

Now let us configure the Gateway, VirtualServer, DestinationRule and sidecar proxy in the existing microservice.
login to oc admin console by getting the oc login console token by clicking on "Copy login command" 
![image](https://github.com/user-attachments/assets/3d24ff95-30e0-48d6-ae5a-c05ec566e63b)

Create project service-mesh
  $oc project service-mesh

  Create Gateway.yaml file

apiVersion: networking.istio.io/v1alpha3

kind: Gateway

metadata:

   name: service-mesh-gateway

   namespace: service-mesh-demo

spec:

   selector:

     istio: ingressgateway # use istio default controller

   servers:

  - port:

       number: 80

       name: http

       protocol: HTTP

     hosts:

    - "*"

 

$oc apply -f Gateway.yaml

 

4. Create the virtualservice.yaml using following content to create wrapper virtual service to route your request to the microservice.  Here httpservice-is is the container name of the microservice and 7800 is the http server port listening of App Connect service.

 

apiVersion: networking.istio.io/v1alpha3

kind: VirtualService

metadata:

   name: virtual-service-httpservice

   namespace: service-mesh-demo

spec:

   hosts:

  - "*"

   gateways:

  - service-mesh-gateway

   http:

  - match:

    - uri:

         exact: /input

     route:

    - destination:

         host: httpservice-is

         port:

           number: 7800



5. Create destinationRule service named destination-rule.yaml. Here we are rejecting the request after 3 retries using parameter “consecutive5xxErrors: 3”

 

apiVersion: networking.istio.io/v1alpha3

kind: DestinationRule

metadata:

  name: httpservice

  namespace: service-mesh-demo

spec:

  host: httpservice-is

  trafficPolicy:

    connectionPool:

      tcp:

        maxConnections: 1

      http:

        httpMaxPendingRequests: 1

        maxRequestsPerConnection: 1

    outlierDetection:

      consecutive5xxErrors: 3

      interval: 5s

      baseEjectionTime: 3m

      maxEjectionPercent: 100

 
6. The sidecar is injected into the httpservice integration server pod while deploying the bar file.

7.  If you have already deployed the bar file, you can edit the integration server and add the Advanced: Annotations - Apply custom annotations to the deployment

Name: sidecar.istio.io/inject  

Value: true

8. We are ready to test the flow.

a. Select 'Istio-system" project 

b. got to Networking --> Route and Get the Gateway route location of istio-ingressgateway route. Here it is "

http://istio-ingressgateway-istio-system.apps.s-ocp.cp.fyre.ibm.com

![image](https://github.com/user-attachments/assets/b8eac1ab-ca45-4e49-a301-230da866cf70)

c. Test the httpInput flow service using any http client application by sending request
  to "http://istio-ingressgateway-istio-system.apps.s-ocp.cp.fyre.ibm.com/input"
  ![image](https://github.com/user-attachments/assets/f46510af-68d2-4044-a550-6a412cfd41ea)

  You will get a success response:
  ![image](https://github.com/user-attachments/assets/56434aef-28ea-4188-89b6-fed3a4d4703e)

  d. Now lets edit the backend service unavailable by changing the backend service internal port to 7801
  ![image](https://github.com/user-attachments/assets/68ca86bc-88ff-41d1-b836-ddbd31410438)


  ![image](https://github.com/user-attachments/assets/bd4837b8-f5c6-48f1-a8e3-df641471761f)

   Click on Save.

  Now backend service is running on 7801 port.

 e. Now test the http client again.

You will get socket operation timeout since the backend port 7800 is not listening.
![image](https://github.com/user-attachments/assets/87d00057-55e0-4a0e-8d28-c9557f2eb02c)
f. Remember in the destination-rule.yaml for httpservice-is we have set 

   consecutive5xxErrors: 3

 This parameter allows three retries to the httpservice-is and after which the circuit breaker is enforces 
 and the request will not hit httpservice-is service and gives output "no healthy upstream"

 ![image](https://github.com/user-attachments/assets/ebea9b77-4772-4823-ade5-789fc502e5f9)

 **Conclusion:**
The article shows how to implement a circuit breaker using Openshift Service Mesh for an App Connect service. The same steps can be used for any other pod service which is hosting http service on a URL. There are many other usecases of service mesh which includes load balancing, content based routing, routing service to different versions of API endpoint, rate limit the API request. Control the API request etc, The high level steps will remain same for all these usecases, only the Destination-rule yaml will change based on each usecase. 

**References:**

A practical example of the Istio service mesh and IBM App Connect Enterprise in IBM Cloud Pak for Integration





  





























