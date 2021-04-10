# k8s-ingress

## What is Ingress?

An API object that manages external access to the services in a cluster, typically HTTP.

Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

Ingress may provide load balancing, SSL termination and name-based virtual hosting


                                    |
Client --> Ingress Managed LB -->   | -> Ingress -> SVC -> Pods
                                    |

An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting. An Ingress controller is responsible for fulfilling the Ingress, usually with a load balancer, though it may also configure your edge router or additional frontends to help handle the traffic.

An Ingress does not expose arbitrary ports or protocols. Exposing services other than HTTP and HTTPS to the internet typically uses a service of type Service.Type=NodePort or Service.Type=LoadBalancer.


## Setup Ingress controller 

There are multiple ingress controller available.

We are going to use Nginx Ingress controller 

You can find installation based on your cluster type on follownig link

https://kubernetes.github.io/ingress-nginx/deploy/

Also to deploy on custom k8s cluster running on virtual machine we will use bare matel ingress deployment 


Deploy: 


```
git clone https://github.com/nginxinc/kubernetes-ingress/
cd kubernetes-ingress/deployment
git checkout v1.11.1

# Create namespace and serviceaccount
kubectl apply -f common/ns-and-sa.yaml

# Create RBAC
kubectl apply -f rbac/rbac.yaml

# Create a secret with a TLS certificate and a key for the default server in NGINX
kubectl apply -f common/default-server-secret.yaml

# Create a config map for customizing NGINX configuration
kubectl apply -f common/nginx-config.yaml

# Create an IngressClass resource (for Kubernetes >= 1.18):
kubectl apply -f common/ingress-class.yaml

# Create ingress controller 
# Edit and add hostNetwork: true and  -enable-custom-resources=false
kubectl apply -f daemon-set/nginx-ingress.yaml
```


```
helm install stable/nginx-ingress --set controller.hostNetwork=true,controller.service.type="",controller.kind=DaemonSet
```


Check status:

```
kubectl get pods -n nginx-ingress --watch
```

## Ingress rules

For each HTTP Rule:

- Host
- list of path /images /api /anything
- backend - combination of service and port 
- if no rule match then redirect to deafultbackend which is nginx page 



## Loadbalancer 

In Cloud environment Loadbalancer are available on-demand, its just 
service, you need to create LB.

But on on-prem you don't have this advantages, so there are following options:

1) MetalLB:  

MetalLB provides a network load-balancer implementation for Kubernetes clusters that do not run on a supported cloud provider, effectively allowing the usage of LoadBalancer Services within any cluster. https://metallb.universe.tf/

Use LoadBalancer in Service Type

2) Over a NodePort Service: Client can access http://myapp.example.com:NodePort where service object is created with NodePort type

3) Via the host network: One can configure ingress-nginx Pods to use the network of the host they run on instead of a dedicated network namespace. The benefit of this approach is that the NGINX Ingress controller can bind ports 80 and 443 directly to Kubernetes nodes' network interfaces, without the extra network translation imposed by NodePort Services.

because bare-metal nodes usually don't have an ExternalIP, one has to enable the --report-node-internal-ip-address flag

3) External IPs 

Use `hostNetwork: true` in controller deployment 


----

Usecase: Deploy web based game app and expose outside world using dns name 


```
kubectl create -f game-deployment.yml
kubectl create -f game-svc.yml
kubectl create -f game-ingress.yml
```

Add DNS entry or update /etc/hosts with your worker node ip and dns name used in ingress rule
