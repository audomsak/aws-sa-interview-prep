# Day 1 interview Q&A drill

**How to use this page:** cover the answers, read only the question, answer out loud as if the interviewer is sitting across from you, *then* check. The goal isn't to reread — it's to catch the gap between what you can say cold and what you can only recognize when you see it.

---

**1. You said you have experience with containers and Kubernetes. What is a container, technically — what's the kernel actually doing?**

> **Hook: "Namespaces control what it sees, cgroups control what it can use."**
> A container is a normal Linux process with two kernel mechanisms applied: namespaces (PID, network, mount, UTS, IPC, user) that give it an isolated *view* of system resources, and cgroups that put hard *limits* on the CPU, memory, and I/O it's allowed to consume. There's no separate "container" kernel object — it's these two mechanisms combined, plus a filesystem built from OCI (Open Container Initiative) image layers.

---

**2. Walk me through the architectural difference between Docker and Podman.**

> **Hook: "Docker asks a root daemon to babysit your container. Podman just becomes the container's parent process directly."**
> Docker uses a client/server model — the `docker` CLI talks over a Unix socket to `dockerd`, a background daemon traditionally running as root, which delegates to containerd and then runc. Podman is daemonless: it forks and execs the container directly as a child of the calling process, is rootless by default via user namespaces, and natively understands pods — hence the name, Pod Manager.

---

**3. Why is a root-owned Docker daemon considered a security concern?**

> **Hook: "Access to the socket is root, full stop."**
> Anyone who can talk to the Docker socket (`/var/run/docker.sock`) — including any container with that socket mounted in — has effectively root-equivalent access to the host, because the daemon itself runs as root and will do anything asked of it through that API. It's a single point of both failure and compromise for the whole host.

---

**4. Explain cgroups v1 versus cgroups v2. Why does the distinction matter?**

> **Hook: "v1 is separate elevators per resource; v2 is one shaft with clear stops for everything."**
> cgroups v1 mounts a separate hierarchy per controller (cpu, memory, blkio each as their own tree), which lets a process sit in inconsistent positions across different resource trees. cgroups v2 unifies everything into a single hierarchy with one consistent path per process, and makes systemd the sole cgroup manager. It matters because modern Kubernetes/OpenShift QoS enforcement (Guaranteed/Burstable/BestEffort) depends on the more consistent, accurate accounting that v2 provides.

---

**5. A pod keeps showing `OOMKilled`. Explain what's actually happening at the OS level.**

> **Hook: "There's no Kubernetes-specific memory police — it's the kernel's OOM killer acting on the cgroup's `memory.max`."**
> Kubernetes translates the pod's memory `limit` directly into that container's cgroup `memory.max` setting. When the process's resident memory crosses that ceiling, the Linux kernel's Out-Of-Memory killer terminates the process — Kubernetes then reports that as `OOMKilled`. The fix is either raising the limit, fixing an actual memory leak, or right-sizing the requests/limits based on real usage.

---

**6. What is the Container Runtime Interface (CRI), and why did Kubernetes remove `dockershim`?**

> **Hook: "Dockershim didn't die because Docker is bad — it died because it was the one runtime that refused to speak the standard language."**
> CRI is a gRPC API contract between the kubelet and any container runtime, covering runtime operations (`RuntimeService`) and image operations (`ImageService`). Docker itself never implemented CRI natively, so Kubernetes maintained a translation shim, `dockershim`, to keep supporting it — special-case code that added maintenance burden. That shim was removed starting Kubernetes v1.24. Docker-built images still work fine everywhere, since they're standard OCI images — it's specifically the Docker daemon as the kubelet's runtime that's no longer supported without a separate adapter.

---

**7. What's the difference between containerd and CRI-O, and where does each show up?**

> **Hook: "containerd came out of Docker; CRI-O was purpose-built for Kubernetes and nothing else."**
> containerd was extracted from Docker's internals and donated to the CNCF — it's a general-purpose runtime, and it's the default on most managed Kubernetes offerings including Amazon EKS. CRI-O was built specifically to be a minimal, Kubernetes-only CRI implementation with no extra surface area — it's the default container runtime in Red Hat OpenShift.

---

**8. If I deploy a Kubernetes `NetworkPolicy` on a cluster running Flannel, what happens?**

> **Hook: "NetworkPolicy is just an API object — Kubernetes never enforces it itself, the CNI plugin does."**
> Nothing happens — silently. Flannel doesn't implement policy enforcement, so the `NetworkPolicy` object gets created in the API but has zero actual effect on traffic. Enforcement is entirely the responsibility of the CNI (Container Network Interface) plugin — Calico, Cilium, and OVN-Kubernetes support it; Flannel by itself does not.

---

**9. Explain what actually happens, mechanically, when a pod gets its IP address.**

> **Hook: "A veth pair is a virtual patch cable — one end in the pod's namespace, one end on the host."**
> The kubelet creates the pod's network namespace, then execs the CNI plugin with an `ADD` command. The plugin creates a veth (virtual Ethernet) pair — one end placed inside the pod's network namespace as `eth0`, the other attached to the host's networking. An IPAM (IP Address Management) component, usually bundled with the plugin, assigns an IP from the cluster's pod CIDR range, and the plugin sets up routing (or BGP peering, or VXLAN tunnels, or eBPF rules, depending on the plugin) so that IP is reachable cluster-wide.

---

**10. What problem did CSI solve that "in-tree" volume plugins didn't?**

> **Hook: "In-tree meant a storage vendor's bug fix had to wait for a whole Kubernetes release."**
> Before the Container Storage Interface (CSI), every storage backend's integration code was compiled directly into Kubernetes core — coupling vendor code to the Kubernetes release cycle and running it with core-component privileges. CSI moves that out-of-tree: vendors ship independently versioned driver pods, split into a Controller plugin (cluster-wide operations like create/attach, run as a Deployment) and a Node plugin (per-node mount operations, run as a DaemonSet), coordinated through standard sidecar containers that translate Kubernetes API watches into CSI gRPC calls.

---

**11. Why does a CSI driver need sidecar containers at all — why not just call the CSI driver directly?**

> **Hook: "The sidecars are the translators — the driver only speaks gRPC, it doesn't watch the Kubernetes API."**
> A CSI driver only implements the CSI gRPC interface; it has no built-in awareness of Kubernetes objects like `PersistentVolumeClaim`. The standard sidecars (`external-provisioner`, `external-attacher`, `external-resizer`, `node-driver-registrar`) are the components that actually watch the Kubernetes API for relevant events and translate them into the appropriate CSI RPC calls against the driver.

---

**12. A container image that ran fine on Docker fails to start on OpenShift with a permissions error. What's likely happening, and how do you fix it?**

> **Hook: "SCC asks what's the most a pod is allowed to request — before the pod even gets to ask."**
> OpenShift's default Security Context Constraint (`restricted-v2`) forces containers to run as a randomly-assigned, non-root UID rather than honoring a hardcoded UID baked into the image (e.g. `USER 1000`). If the image assumes it can write to a specific path only as that fixed UID, it fails. The correct fix is usually to make the image UID-agnostic — set group ownership and `chmod g=u` so *any* UID in the right group can write — rather than reaching for a more permissive SCC, which should be a deliberate, justified exception, not a default workaround.

---

**13. What is an Operator, in your own words, and how is it different from a Deployment?**

> **Hook: "A Deployment keeps a replica count correct. An Operator encodes what a skilled human admin would actually do."**
> A Deployment's reconciliation logic is generic — keep N replicas of this pod template running. An Operator pairs a Custom Resource Definition (a new Kubernetes API type, like `KafkaCluster`) with a controller that encodes real operational domain knowledge — safe upgrades, backups, failover, scaling decisions — and continuously reconciles the cluster toward the declared custom resource, the same reconciliation pattern as core Kubernetes, just with far more specialized logic behind it.

---

**14. When would you choose an OpenShift Route over a Kubernetes Ingress, if both are available to you?**

> **Hook: "Even a plain Ingress on OpenShift quietly becomes a Route underneath — Route is the real mechanism."**
> If you need OpenShift-specific TLS termination modes — particularly re-encrypt, where traffic is decrypted at the router and re-encrypted with a different certificate to the backend, common in regulated environments needing end-to-end encryption — Route gives you that directly. If portability across non-OpenShift Kubernetes distributions matters more than that specific feature set, Ingress is the safer, more portable choice, understanding that on OpenShift itself, the Ingress Controller implements it by creating Route objects underneath anyway.
