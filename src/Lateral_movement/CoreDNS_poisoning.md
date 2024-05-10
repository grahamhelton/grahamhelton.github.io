# CoreDNS poisoning
CoreDNS is the "new" DNS system for Kubernetes which replaced the old KubeDNS system. 

By default, the CoreDNS configuration is stored as a configmap in the `kube-system` namespace

The following is an example of a CoreDNS configuration file. 
```yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        log
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        hosts {
           192.168.49.1 host.minikube.internal
           fallthrough
        }
        forward . /etc/resolv.conf {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
metadata:
  creationTimestamp: "2024-03-29T04:00:45Z"
  name: coredns
  namespace: kube-system
  resourceVersion: "417"
  uid: 40770875-a1f7-4bf0-aeb5-4b71f60035a1

```

# Attacking 
If an attacker is able to edit this ConfigMap, they could redirect DNS traffic. In the following example, running `kubectl exec -it tcpdump -- nslookup grahamhelton.com` returns the name servers of `grahamhelton.com`
- `18.165.83.26`
- `18.165.83.67`
- `18.165.83.47`
- `18.165.83.124`

Running `nslookup` outside of a pod returns the same results. 

![](../images/Pasted%20image%2020240404133500.png)

The CoreDNS config map file can be queried using `kubectl get configmap coredns -n kube-system -o yaml`.

![](../images/Pasted%20image%2020240404135109.png)
If an attacker can edit this ConfigMap, they can add a `rewrite` rule that redirects traffic from `grahamhelton.com` to `kubenomicon.com` by adding in `rewrite name grahamhelton.com kubenomicon.com` into the config map. 

![](../images/Pasted%20image%2020240404133622.png)

Editing the config map can be accomplished by running `kubectl get configmap coredns -n kube-system -o yaml > poison_dns.yaml`, manually adding the file, and then running `kubectl apply -f poison_dns.yaml`, or by running `kubectl edit configmap coredns -n kube-system` and making changes.

![](../images/Pasted%20image%2020240404133817.png)



Once the ConfigMap has been edited, CoreDNS usually needs to be restarted. To do so run `kubectl rollout restart -n kube-system deployment/coredns`. Finally, we can re-run the previous `nslookup` command inside a pod to prove that our traffic to `grahamhelton.com` will be routed to `kubenomicon.com` by running `kubectl exec -it tcpdump -- nslookup grahamhelton.com`. This time, instead of the name servers being returned being the valid name server for `grahamhelton.com`, they are instead the name server for `kubenomicon.com`.

![](../images/Pasted%20image%2020240404133935.png)

# Defending

**Fundamental Defense**

To prevent unauthorized access to editing critical configurations in a Kubernetes environment, such as Helm deployments or network policy definitions, a security engineer should enforce strict access controls. Here are key strategies to implement:

**Role-Based Access Control (RBAC)**
RBAC is essential in Kubernetes to control who can access and modify resources within the cluster. By defining roles and role bindings, you can limit access based on the principle of least privilege.

**Implementation Steps:**

Define Roles or ClusterRoles to specify permissions that include `get, list, watch, create, update, patch, delete` on resources like deployments, pods, and network policies.

Create RoleBindings or ClusterRoleBindings to associate those roles with specific users, groups, or service accounts.
Example of a Role that allows read-only access to network policies:

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: dns
  name: network-policy-viewer
rules:
- apiGroups: ["networking.k8s.io"]
  resources: ["networkpolicies"]
  verbs: ["get", "list", "watch"]
```
**Service Account Management** 

Service accounts are used by applications running in Kubernetes to interact with the Kubernetes API. Ensure that these accounts are secured and only given necessary permissions.

**Implementation Steps:**

Create service accounts specifically for each application or pod that requires access to Kubernetes resources, and bind them to minimal permission roles.
Avoid using default service accounts which may have broader than necessary permissions.

**Use Namespace Isolation**

Isolating resources within specific namespaces can help limit access to those resources based on namespace-specific roles. Discussed in more detail in the Advanced Defense Section.

**Implementation Steps:**

Deploy applications and their related resources within dedicated namespaces.
Apply RoleBindings within those namespaces to restrict access to resources within the same namespace.

**Advanced Defense**

**Isolate Kubernetes DNS pods, such as those running CoreDNS, and apply network policies.**

When deploying CoreDNS or any DNS service using a Helm chart in a Kubernetes cluster, you often have greater flexibility and customization through Helm values, but you must integrate network policies and namespace configurations manually, as they typically aren't specified directly in the chart itself.

**Modify the Helm Values for Namespace Deployment**
Helm charts usually support specifying a namespace directly in the values file or via the command line during installation. Ensure that CoreDNS is deployed in your dedicated dns namespace:
```
helm install my-coredns stable/coredns --namespace dns --create-namespace
```
Alternatively, if you're using a values file for customization, you can specify the namespace directly in your deployment configurations or ensure it's considered during the Helm deployment command.

**Create a Dedicated Namespace for DNS Pods (Manual)** 

To start, you should place your DNS pods in a dedicated namespace. This helps manage security policies more effectively and isolates DNS traffic from other pod traffic

```
kubectl create namespace dns
```
**Deploy CoreDNS in the Dedicated Namespace**
If not already deployed in the dedicated namespace, configure the CoreDNS deployment to run within this namespace. This is often managed by default configurations or Helm charts, but you can specify the namespace directly in the deployment YAML if manually deploying:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coredns
  namespace: dns
  ...
```


**Define Network Policies**


Network policies in Kubernetes are used to specify how groups of pods are allowed to communicate with each other and other network endpoints.Network policies need to be managed outside of the Helm deployment unless the chart specifically includes them. Most DNS-related Helm charts won’t include network policies by default, so you will need to apply them separately.

Since the namespace and pod selectors might depend on labels that are specific to how the Helm chart deploys CoreDNS, make sure these match up. For example, if the Helm chart uses a different label than app: coredns, adjust your network policy selectors accordingly.

For DNS pods, you'll want to create policies that:

Allow DNS Pods to Receive Traffic Only from Specific Namespaces or Pods: This prevents unauthorized access to DNS services.
Restrict DNS Pods from Initiating Unnecessary Connections: DNS pods typically do not need to initiate connections to other pods, except for required services like monitoring or logging.

Here’s an example of how to create such network policies:

**Allow DNS Queries Only from Approved Namespaces**

This policy allows pods in approved namespaces (e.g., default) to send DNS queries to DNS pods in the dns namespace.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-queries
  namespace: dns
spec:
  podSelector:
    matchLabels:
      app: coredns
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: default
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```
**Restrict Outgoing Connections from DNS Pods**
This policy ensures that DNS pods can only initiate connections to certain endpoints (e.g., monitoring or logging services).

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-dns-pod-egress
  namespace: dns
spec:
  podSelector:
    matchLabels:
      app: coredns
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: monitoring
    ports:
    - protocol: TCP
      port: 9090  # Example port for monitoring service
```
**Apply the Network Policies** 

Apply these network policies by using kubectl:
```
kubectl apply -f allow-dns-queries.yaml
kubectl apply -f restrict-dns-pod-egress.yaml
```

**Validate the Deployment**
After deploying the Helm chart and applying the network policies, validate that everything is functioning correctly:

Check that CoreDNS pods are running in the dns namespace.
Test DNS resolution within the cluster to ensure that the network policies haven’t inadvertently blocked legitimate DNS traffic.
Use kubectl get pods --namespace dns to see the CoreDNS deployment and kubectl describe networkpolicy -n dns to verify active network policies.

**Monitor and Adjust**

After deploying these policies, monitor your DNS service's functionality and performance. Ensure that no legitimate traffic is inadvertently blocked, and adjust the network policies as necessary based on observed traffic and evolving security requirements.


# References & Resources
- [The redguard threat matrix has a great writeup on this](https://kubernetes-threat-matrix.redguard.ch/lateral-movement/coredns-poisoning/)! 
- [DNS spoofing on Kubernetes clusters](https://www.aquasec.com/blog/dns-spoofing-kubernetes-clusters/) 