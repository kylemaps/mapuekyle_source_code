---
title: "Kubernetes Journey: From kubectl to GitOps"
date: 2025-07-28
author: "Kyle Mapue"
categories:
  - "devops"
  - "kubernetes"
---

### Introduction

For anyone diving into DevOps, there's a moment when the theory of Kubernetes—all the talk of Pods, Services, and Deployments—needs to meet the messy reality of practice. My journey with the "Log-Pong" application was exactly that: a hands-on, often challenging, but ultimately rewarding experience that took me from manually managing resources to orchestrating a fully automated GitOps pipeline.

This post isn't just a project showcase. It's a story about the power of starting with the fundamentals, embracing the "manual" way to build deep understanding, and then leveraging that knowledge to build a professional, automated workflow. If you're on a similar path, I hope my story resonates and offers some useful insights.

### The Log-Pong Application

Before we dive into the *how* (the deployment), let's quickly cover the *what*. The "Log-Pong" stack consists of two simple microservices designed to demonstrate key Kubernetes concepts:

1.  **The `pingpong-app` (The Backend):**
    *   **What it is:** A stateful backend service.
    *   **Technology:** Node.js with the Koa framework, connected to a PostgreSQL database.
    *   **What it does:** Its only job is to expose an API endpoint (`/ping`). When called, it increments a counter in the database and returns the new number of "pongs." This service represents a stateful component whose data must persist.

2.  **The `log-output-app` (The Frontend/Aggregator):**
    *   **What it is:** A frontend service that gathers information from various sources.
    *   **Technology:** Node.js with Koa, running in a unique two-container Pod.
    *   **What it does:**
        *   A **writer** container continuously writes the current timestamp to a shared file.
        *   A **reader** container waits for requests. When a request comes in, it reads the timestamp from the shared file, fetches a message from a Kubernetes ConfigMap, and makes an HTTP call to the `pingpong-app` to get the current pong count. It then combines all this information into a single string.

**How They Work Together:** The user interacts with the `log-output-app`, which in turn communicates with the `pingpong-app` over the cluster's internal network. The final output looks something like this:

```
file content: this text is from file
env variable: MESSAGE=hello world
2025-07-21T10:30:00.123Z: a1b2c3d4e5f6...
Ping / Pongs: 42
```

It's a simple system, but its architecture is perfect for exploring the core concepts of Kubernetes networking, state management, and configuration. Now, let's get into how it was deployed.

### Manual Deployment

Every project starts somewhere. Mine began with the two applications described above. Initially, getting them to run was a manual process of building Docker images and applying Kubernetes manifests one by one.

```bash
# 1. Build the image
docker build -t my-username/pingpong-app .

# 2. Push it to a registry
docker push my-username/pingpong-app

# 3. Deploy it to the cluster
kubectl apply -f manifests/deployment.yaml
kubectl apply -f manifests/service.yaml
```

I had to do this for both applications. It was tedious, but it was also the perfect way to learn the fundamentals. I wasn't just running a script; I was forced to understand what each `kubectl apply` command was actually doing. What's a `Deployment`? Why does it need a `Service`? How do labels and selectors work? This manual phase was my classroom.

![kubectl get pods,svc output](/blog/my-kubernetes-journey/kubectl-get-pods-svc.png)

### Service Connectivity

Running the apps was one thing; making them talk to each other was a whole new challenge. This is where the real learning began.

My first attempt involved the shared volume between the "writer" and "reader" containers, but the real breakthrough came when I truly understood Kubernetes **Services**. By giving my `pingpong` deployment a `ClusterIP` service, it received a stable DNS name inside the cluster.

Suddenly, my `log-output` app could reliably find and talk to it over the network.

```javascript
// In the log-output reader's code
const PINGPONG_URL = 'http://pingpong-svc:3001/ping';

// ...
const pingResponse = await axios.get(PINGPONG_URL);
const pingCount = pingResponse.data.pong;
```

This was a huge "aha!" moment. I had moved from a shared-filesystem model to true service-to-service communication, the foundation of microservice architecture.

```mermaid
      graph TD
          ExternalUser[External User] --> KubernetesIngress(Kubernetes Ingress\n(e.g., /log-output))
          KubernetesIngress --> ServiceLogOutput(Service: log-output\n(Type: ClusterIP))
          ServiceLogOutput --> PodLogOutput(Pod: log-output\n\n  - Reader Container\n  - Writer Container)
          ServiceLogOutput --> ServicePingpong(Service: pingpong\n(Type: ClusterIP))
          PodLogOutput --> ServicePingpong
          ServicePingpong --> PodPingpong(Pod: pingpong\n\n  - Koa App Container)
```

It's a simple system, but its architecture is perfect for exploring the core concepts of Kubernetes networking, state management, and configuration. Now, let's get into how it was deployed.

### GitOps Automation

The manual workflow had taught me a lot, but it was slow and error-prone. It was time to automate. I decided to refactor the entire project into a professional GitOps workflow using **ArgoCD**.

This meant a fundamental change in structure:

1.  **`pingpong-code` Repo**: Contained only the application source code.
2.  **`log-output-code` Repo**: Contained only its application source code.
3.  **`log-pong-config` Repo**: Contained *only* the Kubernetes YAML manifests. This became my single source of truth.

Next, I built a **CI/CD pipeline** with GitHub Actions for each application repository. On every push to `main`, the pipeline would:

1.  Build a new Docker image and tag it with the commit SHA.
2.  Push the image to Docker Hub.
3.  Check out the `log-pong-config` repository.
4.  **Automatically update the `image:` tag** in the deployment manifest.
5.  Commit and push the change to the config repo.

```yaml
# A snippet from the GitHub Actions workflow
- name: Update Image Tag
  run: |
    sed -i 's|image: .*|image: ${{ secrets.DOCKERHUB_USERNAME }}/pingpong-app:${{ github.sha }}|g' log-pong-config/manifests/rollout.yaml
```

The final piece was **ArgoCD**. I pointed it at my `log-pong-config` repository. Now, whenever the CI pipeline pushed an updated image tag, ArgoCD would detect the change and automatically sync it with my cluster. My job was done. I had created a fully automated, end-to-end deployment pipeline.

![ArgoCD UI showing Healthy and Synced - Part 1](/blog/my-kubernetes-journey/blog_ping1.png)

![ArgoCD UI showing Healthy and Synced - Part 2](/blog/my-kubernetes-journey/blog_ping2.png)

![ArgoCD UI showing Healthy and Synced - Part 3](/blog/my-kubernetes-journey/blog_ping3.png)

### Reliability Enhancements

With the core workflow in place, I added two more features to make the system truly robust.

1.  **Readiness Probes**: The `pingpong` app is useless without its database. I configured a `readinessProbe` that hits a `/healthz` endpoint. This endpoint only returns a `200 OK` if the database connection is active. Now, Kubernetes won't send traffic to a `pingpong` pod until it's confirmed it can do its job.

    ![kubectl describe pod events](/blog/my-kubernetes-journey/kubectl-describe-pod-events.png)

    To prove this, I simulated a database failure by deleting its `StatefulSet`. When the `pingpong` pod tried to restart, it couldn't connect. Running `kubectl describe pod` on the new pod revealed exactly what was happening in the events log:

    ```text
    Status:           Running
    IP:               10.68.2.70
    Ready:            False
    ...
    Events:
      Type     Reason     Age   From     Message
      ----     ------     ----  ----     -------
      Normal   Scheduled  15s   scheduler  Successfully assigned default/pingpong-rollout...
      Normal   Pulled     15s   kubelet    Container image "kylmps/pingpong:5" already present...
      Normal   Created    15s   kubelet    Created container pingpong
      Normal   Started    15s   kubelet    Started container pingpong
      Warning  Unhealthy  4s    kubelet    Readiness probe failed: HTTP probe failed with statuscode: 500
    ```

    The probe was working perfectly, preventing the broken pod from receiving traffic and protecting the system from cascading failures.

2.  **Canary Deployments & Automated Analysis**: To deploy updates safely, I switched from a `Deployment` to an Argo Rollouts `Rollout` resource. I configured it to perform canary releases and defined an `AnalysisTemplate` to monitor CPU usage. If a new version causes a CPU spike, the rollout is automatically aborted and rolled back. No human intervention required.

### Conclusion: The Journey is the Destination

Building the Log-Pong stack taught me more than any tutorial ever could. By starting manually, I was forced to learn the "why" behind every Kubernetes object. That fundamental knowledge was the bedrock upon which I could confidently build a complex, automated GitOps workflow.

If you're just starting out, embrace the manual steps. Break things. Fix them. Understand the moving parts. Because when you finally automate, you'll be doing it with the confidence of someone who knows exactly what's happening under the hood.
