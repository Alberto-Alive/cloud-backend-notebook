
---

## 1. Cloud Fundamentals

### Q1. What does “cloud computing” mean to you?

```text
Cloud computing means renting computing resources (like servers, storage, databases, and networking) from a provider over the internet instead of buying and running your own hardware.

You pay for what you use, and the provider handles the physical infrastructure, scaling, and a lot of the operational heavy lifting.

````

---

### Q2. Explain IaaS, PaaS, and SaaS with simple examples.

```text
IaaS – Infrastructure as a Service:
- The provider gives you raw infrastructure: virtual machines, networks, storage.
- You manage the OS, runtime, and applications.
- Example: AWS EC2, Azure Virtual Machines, Google Compute Engine.

PaaS – Platform as a Service:
- The provider gives you a managed platform to run your code: runtime, scaling, and some managed services.
- You focus on the application; you don’t manage servers or OS.
- Example: Heroku, Azure App Service, AWS Elastic Beanstalk.

SaaS – Software as a Service:
- The provider delivers a complete application you just use in the browser or via an API.
- No infrastructure or deployment concerns for you.
- Example: Gmail, Salesforce, Slack.

```

---

### Q3. Why might a company move from on-prem infrastructure to the cloud?

```text
Common reasons:
- Scalability: easier to scale up/down based on demand without buying hardware.
- Cost model: pay-as-you-go instead of large upfront CapEx.
- Reliability: providers offer managed backups, multi-zone deployments, SLAs.
- Speed: faster to provision environments for new projects.
- Focus: the team can focus more on building features rather than managing hardware.

```

---

### Q4. What are some common building blocks in AWS / Azure / GCP for a typical web application?

(Think: compute, database, storage, networking.)

```text
Conceptually, the building blocks are similar across clouds:

- Compute:
  - Virtual machines (EC2, Azure VM, Compute Engine).
  - Container services (ECS/EKS, AKS, GKE).
  - Serverless functions (Lambda, Azure Functions, Cloud Functions).

- Database:
  - Managed relational DB (RDS, Azure Database for PostgreSQL, Cloud SQL).
  - Managed NoSQL (DynamoDB, Cosmos DB, Firestore).

- Storage:
  - Object storage (S3, Azure Blob Storage, Cloud Storage).
  - Block storage attached to VMs.

- Networking & security:
  - VPC / Virtual Network, subnets, security groups / firewalls, load balancers.

```

---

### Q5. What is the difference between vertical scaling and horizontal scaling? Give an example of each.

```text
Vertical scaling:
- Making a single machine more powerful (more CPU, RAM).
- Example: upgrading a DB instance from 2 vCPUs / 4 GB RAM to 8 vCPUs / 32 GB RAM.

Horizontal scaling:
- Adding more instances and distributing load across them.
- Example: instead of one API server, run 5 containers behind a load balancer and spread traffic.

```

---

### Q6. How would you roughly deploy a simple backend service and database in the cloud?

(High-level: which components would you use, how would they talk?)

```text
High-level idea:

- Use a managed compute platform for the backend:
  - e.g. a container service (ECS/EKS/AKS/GKE) or a PaaS (Azure App Service / AWS Elastic Beanstalk).

- Use a managed database:
  - e.g. managed PostgreSQL (RDS, Azure Database for PostgreSQL, Cloud SQL).

- Put a load balancer in front of the backend:
  - It receives incoming HTTP(S) requests and routes them to healthy instances/containers.

- Networking:
  - Backend and DB live in a private network (VPC / Virtual Network).
  - Backend connects to the DB via a private endpoint, not exposed to the internet.

- DNS:
  - A domain (e.g. api.example.com) points to the load balancer.

So: client → DNS → load balancer → backend containers → database.

```

---

## 2. Docker & Containers

### Q7. What is a container, and how is it different from a virtual machine?

```text
A container is a lightweight, isolated environment for running an application and its dependencies, using the host OS kernel.

Differences vs a VM:
- A VM virtualises the entire hardware and runs its own full OS on top of a hypervisor.
- A container shares the host kernel and isolates processes using namespaces/cgroups, so it’s much lighter and starts faster.

```

---

### Q8. What problem does Docker solve for you as a developer?

```text
Docker gives me reproducible, isolated environments.

- I can package the app plus its dependencies into an image.
- The same image runs the same way on my machine, in CI, and in production.
- This reduces “it works on my machine” issues and simplifies deployment.

```

---

### Q9. Explain, at a high level, what happens when you run `docker run my-image`.

```text
High-level steps:

- Docker looks for `my-image` locally (or pulls it from a registry if needed).
- It creates a new container from that image:
  - sets up a filesystem from the image layers,
  - configures networking, environment variables, etc.
- It starts the default process defined in the image (e.g. `python main.py` or `gunicorn ...`).
- When that process exits, the container stops.

```

---

### Q10. What is a Docker image vs a Docker container?

```text
- A Docker image is a read-only template: it contains the filesystem (code, dependencies, OS libraries) and metadata for how to run the app.
- A Docker container is a running (or stopped) instance of that image: a live process with its own isolated filesystem, network, and configuration.

```

---

### Q11. What are some best practices when writing a Dockerfile for a backend service?

```text
Examples of best practices:

- Use a minimal base image and multi-stage builds to keep the image small.
- Add a .dockerignore file so you don’t copy unnecessary files into the image.
- Install only the dependencies you actually need.
- Run the service as a non-root user inside the container where possible.
- Externalise configuration via environment variables, not hardcoded values.
- Expose the right port and define a clear ENTRYPOINT/CMD.

```

---

### Q12. Why are smaller Docker images generally better?

```text
Because they:

- Download faster (quicker deployments and CI runs).
- Start faster (less to load).
- Have a smaller attack surface (fewer tools and libraries inside).
- Use less disk space and bandwidth.

Overall, they improve performance and security.

```

---

### Q13. What is `docker-compose` and when would you use it?

```text
docker-compose is a tool for defining and running multi-container applications using a YAML file.

I’d use it to:
- Start a whole stack with one command (e.g. backend + DB + Redis).
- Define how containers connect to each other (networks, ports, environment variables).
- Make local development easier and more consistent for the team.

```

---

### Q14. Imagine you have a backend, a Postgres DB, and Redis. Conceptually, how would `docker-compose` help you run them locally?

```text
I’d define a docker-compose.yml with three services: backend, postgres, redis.

- Each service runs in its own container.
- Compose sets up a shared network so the backend can talk to "postgres" and "redis" by service name.
- I can start everything with `docker-compose up` and have a complete environment locally without installing Postgres or Redis directly on my machine.

```

---

## 3. Container Orchestration & Kubernetes (Conceptual)

### Q15. Why do we need something like Kubernetes or an orchestrator if we already have Docker?

```text
Docker runs individual containers, but doesn’t manage them at scale.

An orchestrator like Kubernetes helps with:
- Running many containers across multiple machines.
- Scheduling: deciding where containers run.
- Health checks and automatic restarts.
- Scaling up and down based on load.
- Rolling updates and rollbacks.
- Service discovery and stable networking.

So it turns Docker from “run this container” into “run this service reliably in a cluster”.

```

---

### Q16. What is a Pod in Kubernetes?

```text
A Pod is the smallest deployable unit in Kubernetes.

It usually contains one container (sometimes a few tightly coupled containers) that share:
- the same network namespace (same IP),
- the same storage volumes.

In practice, you think of a Pod as “one instance of your application”.

```

---

### Q17. What is a Deployment in Kubernetes, and what problem does it solve?

```text
A Deployment is a higher-level object that manages a set of identical Pods.

It defines:
- the desired number of replicas,
- the Pod template (image, env vars, etc.).

Kubernetes’ Deployment controller:
- ensures the desired number of Pods are running,
- handles rolling updates (gradually replacing old Pods with new ones),
- allows easy rollbacks to previous versions.

It solves “keep N healthy instances of this app running and update them safely”.

```

---

### Q18. What is a Service in Kubernetes?

```text
A Service is an abstraction that gives a stable network identity (DNS name and port) for a set of Pods.

Pods come and go, their IPs change, but the Service:
- load-balances traffic across the healthy Pods that match a label selector,
- provides a single, consistent endpoint (e.g. my-service:80) for other services or clients to use.

```

---

### Q19. How would you explain “horizontal scaling” of a service in Kubernetes?

```text
Horizontal scaling means changing the number of Pod replicas.

- If load increases, we add more Pods (e.g. from 3 to 10).
- If load decreases, we scale back down.

Kubernetes can do this automatically with a Horizontal Pod Autoscaler based on CPU, memory, or custom metrics.

```

---

### Q20. At a high level, how would you expose a Kubernetes service to the internet?

(Think: Service types / Ingress / load balancer.)

```text
High-level options:

- Use a Service of type LoadBalancer:
  - The cloud provider creates an external load balancer with a public IP that routes traffic to the Service.

- Or use an Ingress:
  - An Ingress controller plus an Ingress resource allows HTTP(S) routing based on host/path.
  - You point DNS (e.g. api.example.com) to the load balancer created for the Ingress, which then routes to the right Service inside the cluster.

```

---

## 4. CI/CD Fundamentals

### Q21. What is Continuous Integration (CI)? What is Continuous Delivery / Deployment (CD)?

```text
Continuous Integration (CI):
- Developers merge changes frequently into a shared main branch.
- Every change triggers automated builds and tests.
- Goal: catch integration issues early and keep the main branch healthy.

Continuous Delivery / Deployment (CD):
- Automating the steps to get code from CI into an environment (staging/production).
- Continuous Delivery: the system is always in a releasable state; deploying to production usually still requires a manual approval.
- Continuous Deployment: every successful change is automatically deployed to production with no manual step.

```

---

### Q22. Why do teams adopt CI/CD pipelines instead of deploying manually?

```text
Because CI/CD:
- Reduces human error (less manual, ad-hoc steps).
- Makes deployments consistent and repeatable.
- Gives faster feedback (tests run automatically on each change).
- Speeds up delivery: smaller, more frequent releases.
- Improves confidence: you can ship more often with less risk.

```

---

### Q23. What typical steps would you include in a CI pipeline for a backend service?

```text
Typical CI pipeline steps:

1. Checkout code.
2. Install dependencies.
3. Run static checks (linting, formatting, maybe type checking).
4. Run unit and integration tests.
5. Build artifacts (e.g. a Docker image).
6. Optionally run security checks (dependency scanning, SAST).
7. On success, push the built image or artifact to a registry.

```

---

### Q24. What is the difference between Continuous Delivery and Continuous Deployment?

```text
Both automate building, testing, and preparing a release.

- Continuous Delivery:
  - Every change is automatically built and tested.
  - The result is always deployable.
  - Deploying to production usually requires a human to press a button / approve.

- Continuous Deployment:
  - Takes Continuous Delivery one step further.
  - Every change that passes the pipeline is automatically deployed to production with no manual approval.

```

---

### Q25. What tools have you used or are familiar with for CI/CD? (e.g. GitHub Actions, GitLab CI, Jenkins)

(Here you can list tools and briefly what you used them for.)

```text
Examples I’m familiar with:

- GitHub Actions:
  - Run tests, linters, and build Docker images on each push/PR.
  - Build & publish images to a container registry.

- GitLab CI:
  - Similar use: pipelines defined in .gitlab-ci.yml for testing and building images.

- Jenkins:
  - Classic CI server for running pipelines, building artifacts, and integrating with other tools.

```

---

## 5. Building & Deploying with CI/CD + Docker

### Q26. How would you integrate Docker into a CI pipeline?

(High-level steps.)

```text
docker-compose containers see if changes do not break docker
```

---

### Q27. Suppose your pipeline currently: runs tests → builds a Docker image → pushes it to a registry.

What additional steps might you add to make it safer for production deployment?

```text
Possible improvements:

- Add static analysis and security checks:
  - Linting, type checking.
  - Dependency vulnerability scanning, image scanning.

- Add integration and smoke tests:
  - Spin up the service and run basic end-to-end checks.

- Add staging deployment:
  - Automatically deploy to a staging environment and run tests there.

- Add manual approval gates:
  - Require human approval before production deploys.

- Implement canary or blue–green deployments:
  - Gradually roll out to a subset of instances/users and monitor before full rollout.

```

---

### Q28. How would you use CI/CD to deploy to a staging environment before production?

```text
Typical pattern:

- Use branches or tags to trigger different pipelines:
  - e.g. every push to main triggers a deploy to staging.

- Pipeline steps:
  1. Build and test the code.
  2. Build and push the Docker image.
  3. Apply configuration for the staging environment (env vars, secrets).
  4. Deploy the new image to the staging cluster or service.
  5. Run smoke tests against staging.

- Production deployment may then be:
  - a manual approval stage after staging succeeds, or
  - triggered by a tag/release.

```

---

### Q29. What is a rollback, and how could you support rollbacks in a containerised deployment?

```text
A rollback is reverting to a previous, known-good version of the application after a bad deployment.

In a containerised setup, you can support rollbacks by:
- Tagging images with versions and keeping old images in the registry.
- In Kubernetes, using Deployments’ built-in rollout history and `kubectl rollout undo`.
- In simpler setups, updating the service to point back to the previous image tag and redeploying.

The key is to keep previous artifacts and configuration so you can quickly switch back.

```

---

### Q30. How does using infrastructure as code (IaC) tools (like Terraform) fit with CI/CD?

```text
IaC lets you describe infrastructure (networks, databases, clusters, etc.) as code.

In CI/CD:
- You can version-control infrastructure definitions.
- Pipelines can run `terraform plan` / `apply` (or similar for other tools) to:
  - create/update infrastructure,
  - keep environments consistent (dev/staging/prod),
  - review changes via code review.

This makes infrastructure changes auditable, repeatable, and automatable, similar to application code.

```

---

## 6. Configuration, Secrets & Environments

### Q31. How do you typically manage configuration for different environments (dev, staging, prod) in a containerised app?

```text
Typical approach:

- Store configuration in environment variables, not hardcoded in the image.
- Have separate config per environment:
  - dev: local .env files (kept out of git) or docker-compose overrides.
  - staging/prod: environment-specific config in the orchestrator or config service.

- Use the same image, but inject different values at runtime:
  - DB connection strings, API URLs, feature flags, etc.

```

---

### Q32. How should secrets (passwords, API keys) be handled in Docker / CI/CD setups?

```text
Principles:

- Never hardcode secrets in code or Docker images.
- Don’t commit secrets to git.

Instead:
- Use a secrets manager or the cloud provider’s secret store (e.g. AWS Secrets Manager, Azure Key Vault).
- In Kubernetes, use Secrets resources, ideally mounted/injected at runtime.
- In CI/CD, use the platform’s encrypted secrets (GitHub Actions secrets, GitLab CI variables) and pass them as environment variables only at runtime.

Access should be restricted and audited.

```

---

### Q33. What is the downside of baking configuration directly into the Docker image?

```text
Downsides:

- You need a new image for every environment (dev/staging/prod).
- Changing a simple config value requires rebuilding and redeploying the image.
- If you accidentally bake secrets into images, they become harder to rotate and more likely to leak.

It’s more flexible and safer to keep the image generic and inject config at runtime.

```

---

## 7. Monitoring, Logging & Reliability (Cloud/CI/CD Angle)

### Q34. After deploying a service via CI/CD, what would you monitor to ensure it’s healthy?

```text
Typical things to monitor:

- Availability and error rates:
  - HTTP 5xx rates, request success/failure.
- Latency:
  - p50/p95/p99 response times.
- Resource usage:
  - CPU, memory, disk, database connections.
- Business metrics:
  - Sign-ups, requests per second, key workflows succeeding.
- Logs:
  - Application logs, error logs, exceptions.

You want to quickly detect regressions in performance, reliability, or user behaviour after a deploy.

```

---

### Q35. How would you integrate tests or checks in the pipeline to prevent “broken” deployments from reaching production?

```text
I’d include multiple layers of checks:

- Unit tests and integration tests in CI.
- Static analysis (linting, type checks) and security scans.
- Build-time checks (e.g. build must succeed, no failed tests).
- Deploy to staging and run smoke/end-to-end tests there.
- Use canaries or limited rollouts with monitoring before full production.

The pipeline should fail fast if any of these checks fail, blocking the production step.

```

---

### Q36. What is a “health check” for a container or service, and why is it important?

```text
A health check is an automated way to determine if a container or service is working correctly.

Examples:
- HTTP endpoint like /health that returns 200 when the service is ready.
- Command inside the container that verifies dependencies (DB, message broker) are reachable.

Important because:
- Orchestrators/load balancers use it to route traffic only to healthy instances.
- It enables automatic restarts of unhealthy containers.
- It helps avoid serving traffic from half-broken instances.

```

---

## 8. Scenario Questions

### Q37. A client has a monolithic backend that they currently deploy manually via SSH to a VM.

They want to move towards containers and CI/CD.
How would you roughly approach this migration?

```text
High-level approach:

1. Containerise the monolith:
   - Write a Dockerfile that can build and run the existing app.
   - Ensure it runs locally in a container.

2. Introduce basic CI:
   - On each push, run tests and build the Docker image.
   - Push the image to a registry.

3. Automate deployment:
   - Replace manual SSH deployment with a scripted or pipeline-driven deploy:
     - e.g. a stage in CI that pulls the new image on the VM and restarts the container (or deploys to a container service).

4. Gradually improve:
   - Add staging environment.
   - Add health checks, rolling or blue–green deployments.
   - Later, consider splitting the monolith into services if there’s a strong reason.

The key is to get repeatable, automated builds and deployments first before over-complicating the architecture.

```

---

### Q38. You notice that deployments sometimes break production because “it worked on my machine”.

How would Docker + CI help reduce this type of issue?

```text
Docker:
- Standardises the runtime environment: same base image, same dependencies everywhere.
- If it works in the container locally, it should work the same in CI and production.

CI:
- Runs the same container build and tests on every commit.
- Ensures that only changes that pass tests and builds get deployed.

Together, they reduce environment drift and make it much less likely that code only works on one person’s machine.

```

---

### Q39. You have a backend service that needs to scale up during traffic spikes (e.g. during a big marketing campaign).

How would you design the deployment so it can scale, using containers and cloud primitives?

```text
Conceptual design:

- Package the backend as a container image.
- Run it on a container platform that supports scaling (ECS/EKS/AKS/GKE or a managed PaaS).
- Put the service behind a cloud load balancer.

For scaling:
- Configure auto-scaling based on metrics (CPU, requests per second, custom metrics).
- During spikes, the platform automatically starts more container instances.
- When traffic drops, it scales back down.

Also ensure:
- The database and other dependencies can handle the load (connection pooling, caching, read replicas if needed).

```

---

### Q40. A deployment pipeline is slow (e.g. 20–30 minutes for each change).

What are some ways you might speed it up?

```text
Possible improvements:

- Optimise Docker builds:
  - Use layer caching effectively.
  - Move rarely changing steps earlier in the Dockerfile.
  - Use smaller base images.

- Optimise tests:
  - Split tests into fast unit tests vs slower integration tests.
  - Run unit tests in parallel.
  - Only run heavy integration tests on certain branches or before release.

- Cache dependencies between pipeline runs.

- Reduce unnecessary steps:
  - Avoid rebuilding images or redeploying services that haven’t changed.

- Scale CI infrastructure:
  - Give more resources or parallel runners to pipelines so steps run faster.

```

---

### Q41. Your API is deployed as containers behind a load balancer in the cloud.

How would you roll out a new version with minimal downtime?

```text
I would use a rolling or blue–green deployment strategy.

Rolling update:
- Gradually replace old containers with new containers:
  - Start new version containers.
  - When they are healthy, remove a subset of old ones.
  - Continue until all are updated.
- The load balancer routes traffic only to healthy instances, so there’s no hard downtime.

Blue–green:
- Run the old version as "blue" and a full new version as "green".
- Once the green environment is healthy, switch the load balancer from blue to green.
- If something goes wrong, switch back to blue (rollback).

Both approaches keep the service available during the rollout.

```

---

### Q42. A client is worried about vendor lock-in if they fully adopt AWS / Azure services.

From a container/CI/CD perspective, what could you do to reduce lock-in risk?

```text
From a container/CI/CD angle:

- Use containers and Kubernetes (or another portable orchestrator) as the main runtime, instead of deep proprietary PaaS.
- Package the app in a way that it can run on any Kubernetes cluster (AWS, Azure, GCP, on-prem).
- Use standard tools and formats (Docker images, Helm charts, Terraform) rather than very provider-specific deployment mechanisms.
- Keep CI/CD pipelines cloud-agnostic where possible:
  - e.g. pipeline builds and pushes images, then applies Kubernetes manifests, rather than relying heavily on proprietary services.

This doesn’t remove lock-in completely, but it makes it much easier to move between providers if needed.

```

---
