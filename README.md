# ic-bare-metal-with-metallb


# 1. Deploy and test an NGINX Plus Ingress Controller 

Open up your terminal application. Type the following command and press Enter (check the detail from my other project repo folder):

```
git clone https://github.com/ericausente/nginx-plus-ic.git
```

Once the cloning process is complete, type the following command and press Enter. This will change your current working directory to the newly cloned repository.

```
cd nginx-plus-ic
```

You should now be in the nginx-plus-ic directory and ready to start working with it. Run the script:
```
bash script.sh
```

This Configures RBAC, Creates Common Resources and Custom Resources And it will ask you for a JWT token obtained from NGINX in order for your to Deploy the Ingress Controller

We are using the NGINX IC Plus JWT token in a Docker Config Secret. This script does it for us wherein it will create a docker-registry secret on the cluster using the JWT token as the username (secret is named as "regcred" and it was added to the NGINX Plus IC deployment spec).

Make sure that the nginx-ingress pod is up and running by executing the below:
```
kubectl get pods -n nginx-ingress
```

In order to get access to the Ingress Controller, A NodePort Service was also created for the Ingress Controller Pods. 

But will change that to Loadbalancer type of Service which will look like the below: 

```
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"nginx-ingress","namespace":"nginx-ingress"},"spec":{"ports":[{"name":"http","nodePort":30080,"port":80,"protocol":"TCP","targetPort":80},{"name":"health","nodePort":30081,"port":8081,"protocol":"TCP","targetPort":8081},{"name":"https","nodePort":30443,"port":443,"protocol":"TCP","targetPort":443}],"selector":{"app":"nginx-ingress"},"type":"NodePort"}}
    metallb.universe.tf/ip-allocated-from-pool: config-addresspool
  creationTimestamp: "2023-07-25T07:29:50Z"
  name: nginx-ingress
  namespace: nginx-ingress
  resourceVersion: "16178399"
  uid: 70aeb331-3b7d-44f6-b571-492531ac2e0c
spec:
  allocateLoadBalancerNodePorts: true
  clusterIP: 10.107.238.198
  clusterIPs:
  - 10.107.238.198
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: http
    nodePort: 30080
    port: 80
    protocol: TCP
    targetPort: 80
  - name: health
    nodePort: 30081
    port: 8081
    protocol: TCP
    targetPort: 8081
  - name: https
    nodePort: 30443
    port: 443
    protocol: TCP
    targetPort: 443
  selector:
    app: nginx-ingress
  sessionAffinity: None
  type: LoadBalancer

```

# 2. Let's now Configure a MetalLB

MetalLB is a Kubernetes add-on that provides load balancing for bare metal Kubernetes clusters. To set up MetalLB, follow the steps below:


Install MetalLB:
    Run the following commands to install MetalLB:
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.3/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml
```

Create the MetalLB Config:
Create a file named metallb_config.yaml and add the following content to it:

```
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  namespace: metallb-system
  name: config-addresspool
spec:
  addresses:
  - 10.201.10.180-10.201.10..181
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: config-advertisement
  namespace: metallb-system
```

Apply the MetalLB Config:
Apply the configuration by running the following command:

```
kubectl apply -f metallb_config.yaml
```

With these steps completed, MetalLB should be set up and ready to provide load balancing for your Kubernetes cluster on bare metal.

```
$ kubectl get svc nginx-ingress -n nginx-ingress
NAME            TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                     AGE
nginx-ingress   LoadBalancer   10.107.238.198   10.201.10.180   80:30080/TCP,8081:30081/TCP,443:30443/TCP   44h
```
