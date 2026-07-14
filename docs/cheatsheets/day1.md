# Day 1 hooks — Linux, Containers, Kubernetes & OpenShift

Every hook from [Day 1](../day1/index.md), in page order — 49 in total. Each one is a compressed page: if a hook doesn't unfold back into its full answer in your head, that's the page to re-read. Built for the final day-before-interview pass.

## [Linux networking fundamentals](../day1/01-linux-networking.md)

- **A container's network is never physically real — it's virtual interfaces, virtual switches, and packet-rewriting rules, all implemented entirely in the host kernel.**
- *"DNAT changes where a packet is going. SNAT/MASQUERADE changes where a packet claims to be from. Port publishing is DNAT; outbound container internet access is MASQUERADE."*

## [Building a container from scratch — Linux containers the hard way](../day1/02-container-from-scratch.md)

- **`docker run` is just a convenient wrapper around `unshare` + `chroot`/`pivot_root` + writing some numbers into `/sys/fs/cgroup` files. Nothing more mystical happens underneath.**
- *"Root isn't one power, it's about 40 separate powers bundled together by default. Capabilities let you hand out exactly the ones a process actually needs, and not one more."*

## [cgroups and namespaces — the real mechanics of "container isolation"](../day1/03-cgroups-namespaces.md)

- **Namespaces control what a process can *see*. Control groups (cgroups) control what a process can *use*.**
- *"v1 is one tree per resource — like separate elevators for CPU, memory, and disk that don't talk to each other. v2 is one elevator shaft with clear stops for everything — a single source of truth."*

## [Docker vs Podman architecture](../day1/04-docker-vs-podman.md)

- *"Docker asks a root daemon to babysit your container. Podman just becomes your container's parent process directly — no babysitter, no daemon, no single point of failure or single point of compromise."*

## [Container networking deep dive](../day1/05-container-networking.md)

- **"Container networking" (Docker/Podman, one host) and "CNI" (Kubernetes, many hosts) are solving the same underlying problem at two completely different scales — confusing them is a very common, very catchable mistake.**
- *"Docker's bridge network solves 'containers on one box talking to each other.' CNI solves 'pods on a hundred boxes all pretending they're on one flat network.' Same primitives, completely different scale of problem."*

## [Kubernetes cluster setup the hard way](../day1/06-kubernetes-the-hard-way.md)

- **Every managed Kubernetes control plane is built from the same handful of manual steps: generate certificates, bootstrap etcd, start the control plane components as authenticated clients of each other, then join nodes that trust the same certificate authority.**
- *"In Kubernetes, nobody just trusts anybody. Every component-to-component connection is mutual TLS, and the CA you generate in step 2 is the root of trust for literally the entire cluster."*
- *"etcd is the cluster's memory, and Raft is the rule that memory can only ever be written to when more than half the cluster agrees — that's why the node count has to be odd."*
- *"Every Kubernetes failure has a layer. If you know the hard-way build order, you know exactly which layer to check first instead of guessing."*

## [Kubernetes architecture & the Container Runtime Interface (CRI)](../day1/07-kubernetes-cri.md)

- **Kubernetes never actually runs a container. It only ever tells something *else* to run a container — and the Container Runtime Interface (CRI) is the contract that lets it say that request in a way any compliant runtime understands.**
- *"Dockershim didn't die because Docker is bad — it died because it was the one runtime that refused to speak the standard language everyone else agreed on."*

## [Networking — Container Network Interface (CNI) deep dive](../day1/08-cni.md)

- **CNI isn't a network. It's a plugin contract — a way for the kubelet to say "give this pod a network interface" without caring how that plugin actually does it.**
- *"A veth pair is a virtual patch cable. One end goes into the pod's isolated network namespace, the other end plugs into the host's networking — that's the entire physical mechanism behind 'the pod has an IP now.'"*

## [Storage — Container Storage Interface (CSI) deep dive](../day1/09-csi.md)

- **CSI did for storage exactly what CRI did for runtimes and CNI did for networking: it moved vendor-specific code out of Kubernetes core and replaced it with a standard plugin contract.**
- *"The sidecars are the translators. The CSI driver only speaks gRPC; the sidecars are the ones actually reading Kubernetes objects and turning 'someone created a PVC' into an actual CSI API call."*

## [OpenShift architecture](../day1/10-openshift-architecture.md)

- **OpenShift's real architectural idea isn't any single feature — it's that almost everything OpenShift adds is itself built as a Kubernetes Operator, managed by other Operators, all the way up to the cluster's own upgrade process. It's "Operators all the way down."**
- *"Raw Kubernetes gives you a car engine. OpenShift gives you the whole car, and the Cluster Version Operator is the one component whose entire job is making sure every other part of the car gets upgraded together, in the right order, without you hand-coordinating it."*

## [OpenShift on-premises deployment architecture](../day1/11-openshift-onprem-deployment.md)

- **IPI automates infrastructure provisioning for you; UPI means you provision the infrastructure and OpenShift only bootstraps the cluster on top of it. On-prem, in security-conscious enterprises, UPI is usually the realistic answer.**
- *"IPI is 'let the installer drive.' UPI is 'you drive, the installer just gets in the car once it's built' — and most regulated on-prem enterprises insist on driving."*

## [OpenShift specifics — SCC, Operators, Routes vs Ingress](../day1/12-openshift-scc-operators-routes.md)

- *"SCC is OpenShift asking 'what's the most this pod is allowed to ask for' — before the pod even gets a chance to ask."*
- *"On OpenShift, even when you write a plain Kubernetes Ingress object, OpenShift quietly turns it into a Route under the hood — Route is the real mechanism, Ingress is a portable API layered on top of it."*

## [Day 1 interview Q&A drill](../day1/13-interview-qa.md)

- *"Namespaces control what it sees, cgroups control what it can use."*
- *"Docker asks a root daemon to babysit your container. Podman just becomes the container's parent process directly."*
- *"Access to the socket is root, full stop."*
- *"v1 is separate elevators per resource; v2 is one shaft with clear stops for everything."*
- *"There's no Kubernetes-specific memory police — it's the kernel's OOM killer acting on the cgroup's `memory.max`."*
- *"Dockershim didn't die because Docker is bad — it died because it was the one runtime that refused to speak the standard language."*
- *"containerd came out of Docker; CRI-O was purpose-built for Kubernetes and nothing else."*
- *"NetworkPolicy is just an API object — Kubernetes never enforces it itself, the CNI plugin does."*
- *"A veth pair is a virtual patch cable — one end in the pod's namespace, one end on the host."*
- *"In-tree meant a storage vendor's bug fix had to wait for a whole Kubernetes release."*
- *"The sidecars are the translators — the driver only speaks gRPC, it doesn't watch the Kubernetes API."*
- *"SCC asks what's the most a pod is allowed to request — before the pod even gets to ask."*
- *"A Deployment keeps a replica count correct. An Operator encodes what a skilled human admin would actually do."*
- *"Even a plain Ingress on OpenShift quietly becomes a Route underneath — Route is the real mechanism."*
- *"`unshare` + `pivot_root` + writing numbers into `/sys/fs/cgroup` — nothing more mystical than that."*
- *"`chroot` changes what you see. `pivot_root` changes what you see and removes the old view from reach."*
- *"Root isn't one power, it's about 40 separate powers bundled together by default — capabilities let you hand out only the ones actually needed."*
- *"Bridge networking solves one host. CNI solves a hundred hosts pretending to be one flat network."*
- *"Every pod secretly has a small, invisible 'pause' container that owns the real network namespace — everything else in the pod just joins it."*
- *"Certificates first, always — nothing in Kubernetes trusts anything else without mutual TLS."*
- *"Raft needs a strict majority — an even number doesn't buy you anything extra over one fewer node."*
- *"Almost everything OpenShift adds is itself an Operator, managed by the Cluster Version Operator — it's Operators all the way down."*
- *"A temporary bootstrap node solves the chicken-and-egg problem of needing a control plane to create a control plane."*
- *"IPI is 'let the installer drive.' UPI is 'you drive, the installer just gets in once it's built' — and regulated enterprises usually insist on driving."*
