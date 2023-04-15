# Project-22
DEPLOYING APPLICATIONS INTO KUBERNETES CLUSTER

### UNDERSTANDING THE CONCEPT

### DEPLOYING APPLICATIONS INTO KUBERNETES CLUSTER

- Let us try to understand a bit more about how the `service object` is able to route traffic to the Pod.

- If you run the below command:

        kubectl get service nginx-service -o wide

- You will get the output similar to this:

        NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE   SELECTOR
        nginx-service   ClusterIP   10.100.71.130   <none>        80/TCP    4d    app=nginx-pod



- As you already know, the service’s type is `ClusterIP`, and in the above output, it has the IP address of `10.100.71.130` – This IP works just like an `internal loadbalancer`. It accepts requests and forwards it to an IP address of any Pod that has the respective `selector label`. In this case, it is `app=nginx-pod`. If there is more than one Pod with that label, service will distribute the traffic to all theese pods in a Round Robin fashion.

- Now, let us have a look at what the Pod looks like:

        kubectl get pod nginx-pod --show-labels

- Output:


        NAME        READY   STATUS    RESTARTS   AGE   LABELS
        nginx-pod   1/1     Running   0          31m   app=nginx-pod


- Notice that the IP address of the Pod, is NOT the IP address of the server it is running on. Kubernetes, through the implementation of network plugins assigns `virtual IP adrresses to each Pod.


        kubectl get pod nginx-pod -o wide


- Output:

        NAME        READY   STATUS    RESTARTS   AGE   IP               NODE                                              NOMINATED NODE   READINESS GATES
        nginx-pod   1/1     Running   0          57m   172.50.197.236   ip-172-50-197-215.eu-central-1.compute.internal   <none>           <none>
        Therefore, Service with IP 10.100.71.130 takes request and forwards to Pod with IP 172.50.197.236


### Expose a Service on a server’s public IP address & static port

Sometimes, it may be needed to directly access the application using the public IP of the server (when we speak of a K8s cluster we can replace ‘server’ with ‘node’) the Pod is running on. This is when the `NodePort service type `comes in handy.

- A Node port service type exposes the service on a static port on the node’s IP address. NodePorts are in the 30000-32767 range by default, which means a NodePort is unlikely to match a service’s intended port (for example, 80 may be exposed as 30080).

- Update the nginx-service yaml to use a NodePort Service.

        apiVersion: v1
        kind: Service
        metadata:
        name: nginx-service
        spec:
        type: NodePort
        selector:
            app: nginx-pod
        ports:
            - protocol: TCP
            port: 80
            nodePort: 30080


- What has changed is:

    * Specified the type of service (Nodeport)
    * Specified the NodePort number to use.


- To access the service, you must:

    * Allow the inbound traffic in your EC2’s Security Group to the NodePort range 30000-32767
    * Get the public IP address of the node the Pod is running on, append the nodeport and access the app through the browser.

_ You must understand that the port number 30080 is a port on the node in which the Pod is scheduled to run. If the Pod ever gets rescheduled elsewhere, that the same port number will be used on the new node it is running on. So, if you have multiple Pods running on several nodes at the same time – they all will be exposed on respective nodes’ IP addresses with a static port number.





### How Kubernetes ensures desired number of Pods is always running?

- When we define a Pod manifest and apply it – we create a Pod that is running until it’s terminated for some reason (e.g., error, Node reboot or some other reason), but what if we want to declare that we always need at least 3 replicas of the same Pod running at all times? 

- Then we must use an ResplicaSet (RS) object – it’s purpose is to maintain a stable set of Pod replicas running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

- Note: In some older books or documents you might find the old version of a similar object – ReplicationController (RC), it had similar purpose, but did not support set-base label selectors and it is now recommended to use ReplicaSets instead, since it is the next-generation RC.

Let us delete our nginx-pod Pod:

        kubectl delete -f nginx-pod.yaml

- Output:

        pod "nginx-pod" deleted

