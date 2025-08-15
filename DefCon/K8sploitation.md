
[REPO](https://github.com/k8sploit33/k8sploitation_labs)

# Why attack K8s

- runs cloud stuff and KochEdge for us
- Single compormised pod can lead to full cluster or cloud compromise
- Default RBAC and networking settings often lack isolation

# K8s basics
- Orchestrates across multiple nodes
- API server is central control point
- Pods = Smallest deployable unit
- Declarative Config
- Everything talks to K8s API
- etcd is the juicy secrets store
- Kubelet - agent on the node
- Kubeproxy - network handling for the pods
- Runtime - docker, containerd etc
- ### Control plane
	- API server
	- etcd
	- scheduler
	- controller manager
- ### Worker Nodes
	- Kubelet
	- Kube-proxy
# K8s auth basics
- ServiceAccounts let pods auth to API server
- Default srv account always on pods (and mounted) unless disabled
- RBAC defines perms
- RoleBindings and ClusterBindings are the attachments to pods as subjects
- Weak RBAC and default tokens is a common privilege escalation path
- Disable default tokens

# Namespaces and Isolation
- Namespaces partition cluster resources into logical groups
- Default namespace is auto-used if not explicit namespace set
- Overly broad defaulr namespace bindings can create issues easily
- Namespaces isolate cluster resources
- Namespaces alone doesn't stop attackers - it's a separation feature not a security feature

# K8s networking overview
- Flat Pod-Pod network by default - every pod can reach every other pod
- Services expose workloads at different scopes
- CoreDNS provides in-cluster DNS resolution
- CNI plugins like Calico or Flannel implement the cluster network layer
- Network policies let you segment and isolate pods
- kube-proxy manages service IPs and routes traffic on each node

# K8s trust model
- Assumes all workloads are trusted by default
- Flat network by default
- Nodes all trust the control plane (and vice versa)
- Many clusters enable powerful features like hostPath, priv containers
- Too much trust everything implicitly

# API server and etcd Attack Surface
AIP Server:
- Mis-configured admission plugins (PSP, Gatekeeper) allow malicious pods
- Unauthenticated health/readiness endpoints (/healthz/metrics)
- Exploit example: lax audit policy -> kubectl proxy + payload injection etcd
- Default listens on 2379/2380 without TLS/auth in many clusters
- Snapshot/raw-get access -> full cluter state & secrets dump
- Some kubectl commands may not get logged
- Example attack snippet:
	- `ETCDCTL_API=3 etcdctl --endpoints=http://<node_ip>:2379 get "" "\x00" --prefix -keys-only`

# Controller Manager and Scheduler Risks
- K8s is full, multi tennant and big
- K3d is small and useful for small deployments
- **Controller Manager**
	- Runs as a static pod under etc/kubernetes/manifests -> hostPath exposure
	- kind of like a hypervisor for the kubernetes cluster
	- overlo-broad RBAC roles can allow "create" on any namespace
- **Scheduler**
	- Similar static-pod setup
	- mis-scoped perms let atackers hijack scheduling logic
	- Case Study: CVE-2020-8565 - malicious ConfigMap injection leading to code execution

# RBAC & ServiceAccount Token Risks
- **Over-permissive RBAC**
	- ClusterRoleBindings that grant cluster-admin to bound groups (eg: system:unauthenticated)
	- demo: `kubectl auth can-i create deployments --as=system:unauthenticated`
- **ServiceAccount Token Hijacking**
	- "automountServiceAccountToken: true" by default -> tokens auto-mounted at /var/run/secrets/....../token
	- Easy to exfiltrate via "kubectl cp" or shared volumes
	- Mitigation Highlights:
		- Scope RBAC bindings narrowly
		- Set "automountServiceAccountToken: false" for crit pods

# Default ServiceAccount & OIDC/Webhook Pitfalls
- **Default ServiceAccount Pitfalls**
	Every namespace's default SA exists nd often has unintended privs
- Misconfigured OIDC/Webhook Authorizers
	- External auth webhooks without mTLS or fail-closed mode can be spofed
	- A single malicious webhook response can escalate to cluster-admin

# Worker Node & Pod Runtime Risks

- Kubelet attack surface
	- How worker contacts control plane to get work
	- Ports 10250 (authenticated) & 10255 (read-only, unauthenticated)
	- Enumeration: curl http://<node_ip>:10255/pods
- PodSecurityContext Abuse
	- runAsUser: 0, hostPID: true, hostNetwork: true will bypass namespace isolation
- Linux Capabilities misuse
	- `kubectl exec attacker-pod --capsh --print`

# Insecure Volume Mounts & SecComp Bypass
- Unconfined Seccomp/AppArmor
	- Default profiles allow syscalls like mount, ptrace, clone
	- Test seccomp lockdown via `kubectl debug -l mage=busybox -attach`
- Mitigations:
	- Disallow broad hostPath mounts - use PodSecurity Admission to restrict volumes
	- Enforce strict seccomp

# CTF

Flag 1 and 2 - 192.168.60.224
Flag 3 and 4 - 192.168.60.181
Flag 5 - 192.168.60.173

