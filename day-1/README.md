# Day 1 – Foundations & Cloud Native Mindset

## Objectives of the Day

At the end of this day, you should be able to:

* Explain what **cloud native architecture** means beyond buzzwords
* Identify the **core architectural constraints** of cloud native systems
* Distinguish cloud native applications from “containerized legacy” systems
* Understand the role of containers and **immutability**
* Set up and validate a **local Kubernetes cluster** using `kind`
* Deploy and run a **first containerized service**, locally and in Kubernetes

Day 1 is about **foundations**.
We deliberately move slowly to ensure everyone starts from a solid and shared base.

---

## Context

Cloud native architecture is not defined by tools, platforms, or vendors.
It is defined by a set of **constraints** imposed by modern environments:

* Infrastructure is ephemeral
* Failures are normal, frequent, and expected
* Scaling is horizontal
* Systems evolve continuously

Kubernetes does not magically solve these problems.
It **enforces** assumptions that architectures must respect.

Today, we focus on:

* building a shared mental model,
* challenging implicit assumptions,
* preparing a local environment that will be used for the rest of the module.

---

## Morning – What “Cloud Native Architecture” Really Means

### Objectives

The goals of the morning session are to:

* Align vocabulary and mental models
* Break common misconceptions around “cloud native”
* Frame the **architectural constraints** that drive design decisions

Before touching Kubernetes or YAML, it is essential to understand *why* these systems exist.

---

### What Cloud Native *Is* (and Is Not)

Topics discussed:

* Cloud native is **not**:

  * “running in containers”
  * “using Kubernetes”
  * “deploying on the cloud”
* Cloud native **is**:

  * an architectural style
  * a response to scale, change, and failure
  * a set of constraints that shape system design

We revisit the CNCF definition with a **2025 perspective**, focusing on intent rather than tooling.

---

### Core Architectural Constraints

We will come back to these constraints throughout the module:

* **Statelessness**
* **Immutability**
* **Horizontal scalability**
* **Failure as a normal condition**

Key idea:

> In cloud native systems, infrastructure is unreliable by design.
> Architecture must assume this from day one.

---

### Cloud Native vs “Containerized Legacy”

We compare:

* monolithic or stateful applications packaged in containers
* applications *designed* for elasticity, replication, and failure

The difference is architectural, not technological.

---

### DevOps, Platform Teams, and SRE (Updated View)

We clarify roles that are often confused:

* Developers
* Platform / DevOps teams
* Site Reliability Engineering (SRE)

And how cloud native architecture reshapes responsibilities between them.

---

### CNCF Ecosystem – An Updated View

Rather than listing tools, we focus on **layers**:

* Runtime
* Networking
* Observability
* Delivery

Important framing points:

* Kubernetes is the **control plane**, not “the solution”
* Tool sprawl is itself an architectural risk
* Simplicity is a feature

---

### Short Exercise (Discussion)

> Take an application you know.
> What breaks if you run **five instances** of it at the same time?

This question will guide many decisions later in the module.

---

## Afternoon – Containers & Local Kubernetes Setup

The afternoon is focused on **making cloud native constraints concrete**.

Expect friction. This is normal.

---

### Local Environment Setup (Important)

Before moving forward, we must ensure that **everyone has a working local environment**.
All the following steps are prerequisites for the rest of the module.

⚠️ **Do not skip this section.**
Small issues here will become major blockers later.

---

#### 1. Container Runtime

You need a **working container runtime**.

Supported options:

* **Docker Desktop** (recommended on macOS / Windows)
* **Docker Engine** (Linux)
* **Podman** (advanced users only)

##### Verify installation

Run:

```bash
docker version
```

You should see:

* a client version
* a server version

If Docker is installed but **not running**, start it now.

If this command fails:

* fix it **before continuing**
* ask for help if needed

---

#### 2. `kubectl` (Kubernetes CLI)

`kubectl` is the command-line tool used to interact with Kubernetes clusters.

##### Verify installation

```bash
kubectl version --client
```

Expected result:

* a client version is displayed
* no error message

If `kubectl` is missing, install it using the **official Kubernetes documentation**:

👉 [https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)

> Do **not** install Kubernetes itself manually.
> We will use `kind` to create and manage the cluster.

---

#### 3. `kind` (Kubernetes IN Docker)

We use **kind** to run Kubernetes locally inside containers.

Why `kind`:

* fast startup
* reproducible clusters
* close to real Kubernetes behavior
* well suited for local development and teaching

##### Verify installation

```bash
kind version
```

If the command is not found, install `kind` following the **official documentation**:

👉 [https://kind.sigs.k8s.io/docs/user/quick-start/](https://kind.sigs.k8s.io/docs/user/quick-start/)

Choose the installation method appropriate for your operating system.

After installation, verify again:

```bash
kind version
```

---

#### 4. Creating the Local Kubernetes Cluster

Each student will run **their own local Kubernetes cluster**.

We will use a **simple, default cluster configuration**.

Create the cluster:

```bash
kind create cluster
```

⏳ This may take a few minutes.

---

#### 5. Verify the Cluster Is Working

Run:

```bash
kubectl get nodes
```

Expected result:

* one node
* status `Ready`

Example:

```text
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   1m    v1.xx.x
```

If the node is **not Ready**:

* stop here
* do not proceed
* ask for help

---

#### 6. Common Problems (Read This)

Some issues are **very common**:

* Docker is installed but not running
* Docker does not have enough memory allocated
* A VPN interferes with networking
* An old or broken Kubernetes configuration exists in `~/.kube/config`

If something behaves strangely:

* do **not** randomly reinstall everything
* read the error message carefully
* you may use an LLM to help *understand* the error, but remain in control of the fix

---

#### 7. Final Checkpoint (Mandatory)

Before continuing, you must be able to run **all three commands successfully**:

```bash
docker version
kubectl get nodes
kind version
```

If any of these fail:

* stop
* fix the issue
* ask for help

Cloud native systems assume unstable environments.
Your local setup must not be one of them.

> This setup step is intentionally boring and sometimes frustrating. This is what real cloud-native work looks like before the fun begins.


---

### Step 1 – Containers as Immutable Artifacts

#### Objectives

In this step, you will:

* Build and run a **containerized application**
* Understand the difference between an **image** and a **container**
* Discover why **immutability** is a core cloud native principle
* Observe how configuration is injected **at runtime**, not baked into the image

This step deliberately avoids business complexity.
The goal is to understand **how applications are packaged and executed** in cloud native environments.

---

#### Context

In cloud native architectures:

* Applications are **built once**
* Deployed **many times**
* Run in **multiple environments**
* Replaced rather than modified

This is only possible if application artifacts are **immutable**.

Containers are not interesting because they are “lightweight VMs”.
They are interesting because they allow us to treat applications as **immutable artifacts**.

---

#### The Application: `echo-service`

For this module, we use a very small HTTP service called **`echo-service`**. You can find it in the `services/echo-service` directory.

It exposes a few endpoints:

* `GET /healthz`
  Basic health endpoint (used later for Kubernetes probes)

* `GET /info`
  Returns runtime information (hostname, PID, uptime, Node.js version)

* `GET /config`
  Returns a *safe* subset of configuration values

* `POST /echo`
  Returns the request body with metadata

This service is intentionally simple, but it already demonstrates:

* ephemeral instances
* runtime configuration
* reproducibility

---

#### Step 1.1 – Inspect the Service

Go to the service directory:

```bash
cd services/day-1/echo-service
```

You should see:

* `server.js` – the application code
* `package.json` – Node.js dependencies
* `Dockerfile` – instructions to build the container image

👉 Take a few minutes to **read the Dockerfile**.

Questions to ask yourself:

* What base image is used?
* When are dependencies installed?
* What files are copied into the image?
* What command starts the application?

Do not rush this step.

---

#### Step 1.2 – Build the Container Image

Build the image locally:

```bash
docker build -t echo-service:dev .
```

What happens here:

* Docker reads the `Dockerfile`
* It produces a **container image**
* This image is a **static artifact**

Important:

> Once built, this image will **never change**.
> Any change requires building a **new image**.

---

#### Step 1.3 – Run the Container Locally

Run the container:

```bash
docker run --rm -p 8080:8080 \
  -e APP_NAME=echo-service \
  -e APP_VERSION=dev \
  -e FEATURE_FLAG_X=off \
  echo-service:dev
```

What this does:

* Starts a container **from the image**
* Injects configuration via **environment variables**
* Exposes port `8080` to your local machine

---

#### Step 1.4 – Test the Service

In another terminal, test the endpoints:

```bash
curl http://localhost:8080/healthz
```

```bash
curl http://localhost:8080/info
```

```bash
curl http://localhost:8080/config
```

```bash
curl -X POST http://localhost:8080/echo \
  -H "Content-Type: application/json" \
  -d '{"hello":"world"}'
```

Observe carefully:

* `/info` returns runtime-specific data (hostname, PID)
* `/config` reflects values passed at startup
* The container has **no memory** of previous runs

---

#### Step 1.5 – Stop and Restart the Container

Stop the container (`Ctrl+C`) and run it again.

Then call `/info` again.

Ask yourself:

* Did the hostname change?
* Did the PID change?
* Did the application “remember” anything?

This is **expected behavior**.

---

#### Key Concepts Illustrated

This step demonstrates several fundamental cloud native ideas:

##### Image vs Container

* **Image**: immutable, built once
* **Container**: a running instance of an image

You can run:

* one image
* as many containers
* as many times as needed

##### Immutability

You never:

* “patch” a running container
* log into it to fix things
* modify files inside it

You:

* rebuild the image
* redeploy

---

##### Configuration Outside the Image

Configuration is provided at runtime via:

* environment variables
* (later) ConfigMaps and Secrets

The image itself remains unchanged.

---

##### Things to Pay Attention To

* Logs are written to **standard output**
* Stopping the container destroys it completely
* Restarting creates a **new instance**
* The filesystem inside the container is ephemeral

If this feels uncomfortable, that’s normal.
Cloud native systems trade comfort for predictability.

---

#### End of Step Checkpoint

Before moving on, make sure you can explain:

* The difference between an image and a container
* Why immutability matters
* Why configuration should not be baked into the image
* Why restarting a container is not a problem

If you cannot explain it, ask questions **now**.

---

#### Reflection (2–3 minutes)

Answer briefly:

* What would break if we ran **five containers** of this image?
* What would happen if we tried to store data in the container filesystem?
* Why is this model better suited for automation?

We will reuse this same service in the next steps — **without changing the code**.


---

### Step 2 – Deploying `echo-service` to Kubernetes

#### Objectives

In this step, you will:

* Run the **exact same container image** as a Kubernetes workload
* Understand the role of:

  * **Pods** (where containers run)
  * **Deployments** (desired state + self-healing)
  * **Services** (stable networking)
* Learn the basic workflow to **inspect and debug** a Kubernetes deployment

We still keep the system simple: one service, one replica, no database.

---

#### What We Are Doing

So far, you ran `echo-service` with `docker run`.

Now we run it **inside Kubernetes**. The application does not change — only the execution environment changes.

Important mental model:

* You do **not** manage containers directly anymore
* You declare what you want
* Kubernetes continuously tries to make reality match your **desired state**

---

#### Step 2.1 – Ensure the Image Is Available to `kind`

When using `kind`, Kubernetes runs inside Docker.
Your cluster will **not automatically see images** you built locally unless you:

* push them to a registry, or
* load them into the cluster

For Day 1, we use the simplest option: **load the image into kind**.

From the `services/echo-service` directory (or from repo root), make sure you have built the image:

```bash
docker images | head
```

You should see `echo-service:dev`.

Load the image into `kind`:

```bash
kind load docker-image echo-service:dev
```

If you rebuilt the image after creating the cluster, re-run `kind load docker-image ...` (kind does not auto-sync local builds).

✅ Checkpoint: this command completes without errors.

---

#### Step 2.2 – Apply the Kubernetes Manifest

We provide a Kubernetes manifest for Day 1.

From the repository root:

```bash
kubectl apply -f k8s/day-1/echo-service.yaml
```

What this typically creates:

* A **Deployment** for `echo-service`
* A **Service** to expose it inside the cluster

---

#### Step 2.3 – Verify That It’s Running

Check Pods:

```bash
kubectl get pods
```

You should see something like:

* `echo-service-...` in `Running`

If it’s not running, check what is happening:

```bash
kubectl describe pod <pod-name>
```

Check Deployments:

```bash
kubectl get deployments
```

Check Services:

```bash
kubectl get services
```

✅ Checkpoint: you have:

* 1 Deployment
* 1 Pod running
* 1 Service created

```bash
kubectl get all -l app=echo-service
```

---

#### Step 2.4 – Access the Service

Inside Kubernetes, services are reachable **inside the cluster network**.
To access it from your laptop, we use **port-forwarding** (simple and reliable for local learning).

Run:

```bash
kubectl port-forward service/echo-service 8080:8080
```

Keep that terminal open.

Now test from another terminal:

```bash
curl http://localhost:8080/healthz
curl http://localhost:8080/info
curl http://localhost:8080/config
curl -X POST http://localhost:8080/echo \
  -H "Content-Type: application/json" \
  -d '{"hello":"kubernetes"}'
```

#### What to observe

Run `/info` multiple times.

You should see:

* a hostname that looks like a Pod name
* runtime values that come from inside the cluster

This is your first “distributed system feeling”:

> You are no longer interacting with “your machine”, but with a managed runtime.

✅ Checkpoint: you can call `/info` and `/echo` through port-forward.

---

#### Step 2.5 – Debug Like an Engineer

##### View logs

```bash
kubectl logs deployment/echo-service
```

or, to follow logs:

```bash
kubectl logs -f deployment/echo-service
```

Logs should include the startup message from the service.

##### Exec into the running container (optional)

Find the pod name:

```bash
kubectl get pods
```

Then:

```bash
kubectl exec -it <pod-name> -- sh
```

Inside the container, you can explore (this is mostly for learning/debugging):

```sh
env | sort
ls -la
exit
```

Important warning:

> You can inspect containers, but you should **not rely** on manual changes inside them.
> Any change here is ephemeral and will be lost.

---

#### Step 2.6 – Prove Self-Healing (The “Aha” Moment)

Kubernetes keeps the Deployment running.
Let’s test that.

Delete the running Pod:

```bash
kubectl delete pod -l app=echo-service
```

Immediately check Pods:

```bash
kubectl get pods -w
```

What you should see:

* the old Pod disappears
* a new Pod is created automatically

Now re-run `/info` (your port-forward may need to be restarted if the Pod changed).

This demonstrates the most important Kubernetes idea:

> You manage **desired state**.
> Kubernetes manages **actual state**.

If port-forward stops responding after the Pod was recreated, stop it and run the command again.

✅ Checkpoint: you successfully deleted a Pod and Kubernetes recreated it.

---

#### Things to Pay Attention To

* **Pods are disposable**: Kubernetes will replace them without asking
* **You should not depend on Pod identity**

  * hostnames change
  * IP addresses change
* **The container filesystem is ephemeral**
* **Logs go to stdout/stderr**

  * you retrieve them with `kubectl logs`
* **Images are immutable**

  * if you change code, you must rebuild the image and reload it into kind

If this feels uncomfortable, that’s expected.
Cloud native systems are opinionated — and they force better habits.

---

#### End of Step Checkpoint

Before moving on, make sure you can explain:

* Why Kubernetes needs a **Deployment** (not just “run a container”)
* Why a **Service** exists (stable networking)
* What happens when a Pod is deleted
* Why `kind load docker-image` is needed in a local cluster workflow

---




### Optional Explorations (Time Permitting)

These steps are **optional** and meant for exploration and discussion.
If you are short on time, skip them without regret.

---

### Optional Step A – Scaling the Application (Replicas)

#### Goal

Observe what happens when the **same application** runs multiple times at once.

This reinforces:

* statelessness
* horizontal scalability
* ephemerality of instances

---

#### Step A.1 – Increase the Number of Replicas

Scale the Deployment to 3 replicas:

```bash
kubectl scale deployment echo-service --replicas=3
```

Watch what happens:

```bash
kubectl get pods -w
```

You should see:

* new Pods being created
* all Pods eventually reaching `Running`

---

#### Step A.2 – Call the Service Repeatedly

If your port-forward is still running, reuse it.
Otherwise:

```bash
kubectl port-forward service/echo-service 8080:8080
```

Now call `/info` several times:

```bash
curl http://localhost:8080/info
```

Repeat the command multiple times.

#### What to Observe

Look at:

* `hostname`
* `pid`

You should notice:

* different hostnames across requests
* the service responding from **different Pods**

This is your first concrete experience of **horizontal scaling**.

---

#### Discussion Prompt

Ask yourself:

* Why does this work without changing the code?
* What would break if the application stored state in memory?
* Why is this different from running 3 processes on the same machine?

---

### Optional Step B – Load Balancing Inside Kubernetes

#### Goal

Understand **where load balancing actually happens**.

Many people assume Kubernetes uses a “big load balancer”.
That’s not exactly true.

---

#### Step B.1 – Inspect the Service

Describe the Service:

```bash
kubectl describe service echo-service
```

Look at:

* the selector
* the list of endpoints

Then list endpoints directly:

```bash
kubectl get endpoints echo-service
```

You should see:

* multiple IP addresses
* one per Pod

---

#### Step B.2 – How Requests Are Distributed

When you access the service:

* requests go to the **Service IP**
* Kubernetes forwards them to **any matching Pod**

Important idea:

> Kubernetes load balancing is **simple and stateless** by default.

There is:

* no session affinity (unless configured)
* no application-level awareness

---

#### Optional Experiment (Advanced Curiosity)

In one terminal, follow logs:

```bash
kubectl logs -f deployment/echo-service
```

In another terminal, repeatedly call:

```bash
curl http://localhost:8080/info
```

Watch how:

* requests appear in logs from different Pods

---

### Optional Step C – Killing Pods While Serving Traffic

#### Goal

Demonstrate resilience **without special tooling**.

---

#### Step C.1 – Delete a Pod While Making Requests

In one terminal:

```bash
kubectl get pods
```

Pick one Pod and delete it:

```bash
kubectl delete pod <pod-name>
```

At the same time, continue calling:

```bash
curl http://localhost:8080/info
```

#### What to Observe

* Requests continue to succeed
* A new Pod is created automatically
* The Service keeps routing traffic

This demonstrates:

* self-healing
* decoupling between clients and instances

---

### Key Takeaways from Optional Steps

These experiments demonstrate that:

* Applications must be **stateless**
* Instances are **replaceable**
* Scaling does not require code changes
* Load balancing is **infrastructure-driven**
* Kubernetes enforces cloud native assumptions

If any of these points feel surprising, that’s a good sign.

---

### End-of-Day 1 Summary

Before ending Day 1, you should be comfortable explaining:

* Why containers are immutable artifacts
* Why Kubernetes manages **desired state**
* Why Pods are disposable
* Why Services exist
* Why scaling breaks bad architectures

Day 2 will build on this by:

* externalizing configuration properly
* managing updates and rollouts
* treating Kubernetes as a **platform**, not just a scheduler

---

### Reflection (5–10 minutes)

Answer briefly:

* What was new or surprising today?
* What assumptions were challenged?
* What would break if we ran **10 instances** of this service?

We will build on these answers tomorrow.

