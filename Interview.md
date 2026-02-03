# Three-Tier Architecture Demo - Interview Preparation Guide
## 🎤 10-Minute Project Explanation Script
### Opening (30 seconds)
> "I deployed a production-grade, microservices-based e-commerce application called **Stan's Robot Shop** on **AWS EKS** using **Helm charts** and **Infrastructure as Code** principles. This project demonstrates my understanding of containerization, Kubernetes orchestration, cloud-native architecture, and DevOps best practices."
---
### Architecture Overview (2 minutes)
> "The application follows a **three-tier architecture**:"
| Tier                  | Components                      | Purpose                      |
| --------------------- | ------------------------------- | ---------------------------- |
| **Presentation Tier** | Nginx + AngularJS               | User-facing web interface    |
| **Application Tier**  | 7 microservices                 | Business logic               |
| **Data Tier**         | MongoDB, MySQL, Redis, RabbitMQ | Data persistence & messaging |
> "The microservices are built using polyglot technologies:"
- **NodeJS (Express)** → Catalogue, User, Cart services
- **Java (Spring Boot)** → Shipping service
- **Python (Flask)** → Payment service
- **Golang** → Dispatch service
- **PHP** → Ratings service
> "This polyglot approach simulates real-world enterprise environments where different teams use different technology stacks."
---
### Deployment Architecture (2 minutes)
> "I deployed this on AWS EKS with the following components:"
1. **EKS Cluster** - Managed Kubernetes cluster using `eksctl`
2. **OIDC Provider** - For IAM roles to Kubernetes service accounts (IRSA)
3. **AWS Load Balancer Controller** - Manages ALB for ingress traffic
4. **EBS CSI Driver** - For persistent volumes (databases)
5. **Helm Charts** - Package manager for Kubernetes deployments
> "The traffic flow is:  
> User → ALB (Internet-facing) → Ingress → Web Service → Backend Microservices → Databases"
---
### Key Technical Decisions (3 minutes)
#### Why EKS?
> "I chose EKS because:
>
> - Managed control plane reduces operational overhead
> - Native AWS integrations (ALB, EBS, IAM)
> - Production-ready with automatic updates and patches
> - Multi-AZ high availability out of the box"
#### Why Helm?
> "I used Helm because:
>
> - **Templatization** - One chart deploys 12 microservices with consistent config
> - **Parameterization** - Easy to customize via `values.yaml` for different environments
> - **Version Control** - Rollback to previous releases with `helm rollback`
> - **Dependency Management** - Easy to manage related Kubernetes resources together"
#### Why ALB Ingress Controller?
> "The AWS Load Balancer Controller:
>
> - Creates an Application Load Balancer automatically from Ingress resources
> - Supports path-based and host-based routing
> - Integrates with AWS WAF for security
> - Uses IP mode for direct pod communication (better performance)"
#### Why EBS CSI Driver?
> "For databases like MongoDB and MySQL:
>
> - Need persistent storage that survives pod restarts
> - EBS provides durable block storage with snapshots
> - CSI driver enables dynamic provisioning via StorageClass"
---
### Infrastructure Highlights (2 minutes)
> "Key infrastructure patterns I implemented:"
1. **IRSA (IAM Roles for Service Accounts)**
   - ALB Controller uses `AWSLoadBalancerControllerIAMPolicy`
   - EBS CSI uses `AmazonEBSCSIDriverPolicy`
   - No hardcoded credentials - follows least privilege principle
2. **Ingress Configuration**
   ```yaml
   annotations:
     kubernetes.io/ingress.class: alb
     alb.ingress.kubernetes.io/scheme: internet-facing
     alb.ingress.kubernetes.io/target-type: ip
   ```
3. **Helm Values for Environment-Specific Configuration**
   - Image repository and tags
   - Resource limits and requests
   - Node affinity for workload placement
---
### Closing (30 seconds)
> "This project demonstrates my ability to deploy complex, distributed systems on cloud infrastructure using industry-standard tools and practices. I'm comfortable with both the infrastructure layer (AWS, Kubernetes) and the application layer (microservices, databases)."
---
---
## ❓ Interview Questions & Answers
### Basic Questions
#### Q1: What is a three-tier architecture?
> **Answer:** A three-tier architecture separates an application into three logical layers:
>
> 1. **Presentation tier** - User interface (web/mobile)
> 2. **Application tier** - Business logic (microservices/APIs)
> 3. **Data tier** - Data storage (databases, caches)
>
> **Benefits:** Separation of concerns, independent scaling, easier maintenance, and security isolation.
#### Q2: What is the difference between Kubernetes Service and Ingress?
> **Answer:**
>
> - **Service** - Internal load balancing within the cluster (ClusterIP, NodePort, LoadBalancer)
> - **Ingress** - External HTTP/HTTPS routing to services. Provides path-based routing, SSL termination, and hostname-based routing.
>
> In this project, services connect microservices to each other, while Ingress exposes the web frontend to the internet via ALB.
#### Q3: What is OIDC and why is it used with EKS?
> **Answer:** OIDC (OpenID Connect) provider enables **IAM Roles for Service Accounts (IRSA)**. Instead of storing AWS credentials in pods, Kubernetes service accounts assume IAM roles using OIDC tokens. This follows the principle of least privilege and eliminates credential management overhead.
---
### Intermediate Questions
#### Q4: Why did you use Helm instead of plain Kubernetes manifests?
> **Answer:**
>
> 1. **Templatization** - I have 12 services; instead of 24+ YAML files, I use templates with variables
> 2. **Configuration Management** - Different environments (dev/staging/prod) use different `values.yaml`
> 3. **Release Management** - `helm list`, `helm rollback`, `helm upgrade` for version control
> 4. **Dependency Management** - Single command deploys all related resources
> 5. **Reusability** - Same chart can be used across multiple clusters
#### Q5: How does the ALB Ingress Controller work?
> **Answer:**
>
> 1. I deploy the AWS Load Balancer Controller via Helm
> 2. When I create an Ingress resource with `kubernetes.io/ingress.class: alb`, the controller watches for it
> 3. Controller creates an ALB in AWS with target groups
> 4. With `target-type: ip`, ALB routes directly to pod IPs (not through NodePort)
> 5. Controller continuously syncs Ingress changes to ALB configuration
#### Q6: Explain the service discovery pattern in this architecture.
> **Answer:** The microservices use **Kubernetes DNS-based service discovery**:
>
> - Each service gets a DNS name: `<service-name>.<namespace>.svc.cluster.local`
> - Example: Cart service connects to Redis using `redis.robot-shop.svc.cluster.local`
> - Kubernetes CoreDNS resolves these names to ClusterIP addresses
> - No hardcoded IP addresses; services discover each other by name
---
### Advanced Questions
#### Q7: How would you handle database persistence in Kubernetes?
> **Answer:**
> In this project, I use:
>
> 1. **EBS CSI Driver** for dynamic PersistentVolume provisioning
> 2. **StorageClass** `gp2` for Redis StatefulSet
> 3. **PersistentVolumeClaim (PVC)** attached to database pods
>
> For production, I would consider:
>
> - **RDS/DocumentDB** for managed databases (better HA, backups)
> - **EBS snapshots** for backup/restore
> - **Multi-AZ storage** for high availability
#### Q8: How would you implement zero-downtime deployments?
> **Answer:**
>
> 1. **Rolling Updates** (default in Kubernetes)
>    - `maxUnavailable: 0` - Keep all old pods running during update
>    - `maxSurge: 25%` - Create new pods before terminating old ones
> 2. **Readiness Probes** - Only route traffic to healthy pods
> 3. **Health Checks** - The services have `/health` endpoints
> 4. **Blue-Green/Canary** - Using Istio or AWS App Mesh for advanced strategies
#### Q9: What monitoring/observability would you add?
> **Answer:**
>
> 1. **Prometheus + Grafana** - Metrics (this app exposes `/metrics` endpoints)
> 2. **FluentD + ELK/CloudWatch** - Log aggregation (fluentd dir already exists)
> 3. **Jaeger/X-Ray** - Distributed tracing across microservices
> 4. **Instana** - The app is pre-configured for Instana APM
> 5. **CloudWatch Container Insights** - AWS-native monitoring for EKS
---
### Troubleshooting Questions
#### Q10: A pod is in CrashLoopBackOff. How do you debug?
> **Answer:**
>
> ```bash
> # 1. Check pod status and events
> kubectl describe pod <pod-name> -n robot-shop
>
> # 2. Check logs
> kubectl logs <pod-name> -n robot-shop --previous
>
> # 3. Check if dependencies are running
> kubectl get pods -n robot-shop
>
> # 4. Check resource limits
> kubectl top pod <pod-name> -n robot-shop
>
> # 5. Exec into container (if running)
> kubectl exec -it <pod-name> -n robot-shop -- sh
> ```
>
> Common causes: OOMKilled, failed readiness probe, missing dependencies, config errors.
#### Q11: The application is slow. How do you troubleshoot?
> **Answer:**
>
> 1. **Check metrics** - CPU/memory usage of pods
> 2. **Check HPA** - Is autoscaling working?
> 3. **Check database** - Connection pools, slow queries
> 4. **Check network** - Latency between pods, DNS resolution time
> 5. **Distributed tracing** - Find which service is the bottleneck
> 6. **Load testing** - Use the `load-gen` tool in the repo to reproduce
---
## 🔧 Challenges Faced & Solutions
### Challenge 1: OIDC Provider Configuration
> **Problem:** The ALB Controller pods couldn't create ALBs because they lacked AWS permissions.
>
> **Root Cause:** OIDC provider wasn't associated with the EKS cluster.
>
> **Solution:**
>
> ```bash
> # Verify OIDC provider
> aws iam list-open-id-connect-providers | grep $oidc_id
>
> # If not present, associate it
> eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
> ```
>
> **Learning:** IRSA requires OIDC provider to be configured before any service accounts can assume IAM roles.
---
### Challenge 2: Persistent Volume Binding Failure
> **Problem:** MongoDB and MySQL pods were stuck in `Pending` state.
>
> **Root Cause:** EBS CSI driver wasn't installed; PVCs couldn't be bound.
>
> **Solution:**
>
> ```bash
> # Create IAM role for CSI driver
> eksctl create iamserviceaccount \
>     --name ebs-csi-controller-sa \
>     --namespace kube-system \
>     --cluster <cluster-name> \
>     --role-name AmazonEKS_EBS_CSI_DriverRole \
>     --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy
>
> # Install the addon
> eksctl create addon --name aws-ebs-csi-driver --cluster <cluster-name> ...
> ```
>
> **Learning:** EKS doesn't include CSI drivers by default; they must be installed as addons.
---
### Challenge 3: Ingress Not Creating ALB
> **Problem:** Ingress resource was created but no ALB appeared in AWS console.
>
> **Root Cause:**
>
> 1. ALB Controller wasn't installed
> 2. Missing annotation `kubernetes.io/ingress.class: alb`
>
> **Solution:**
>
> ```bash
> # Check controller logs
> kubectl logs -n kube-system deployment/aws-load-balancer-controller
>
> # Verify ingress annotations
> kubectl get ingress -n robot-shop -o yaml
> ```
>
> **Learning:** Always check controller logs when Ingress doesn't work. The controller watches for specific annotations.
---
### Challenge 4: Inter-Service Communication Issues
> **Problem:** Cart service couldn't connect to Redis on startup.
>
> **Root Cause:** Cart pod started before Redis was ready.
>
> **Solution:**
>
> 1. Added `depends_on` in docker-compose for local development
> 2. Added readiness probes in Kubernetes
> 3. Used init containers to wait for dependencies in K8s
>
> **Learning:** Service dependencies need to be handled at orchestration layer.
---
### Challenge 5: ALB Target Health Check Failures
> **Problem:** ALB showed targets as unhealthy, no traffic reaching pods.
>
> **Root Cause:** ALB health check was hitting `/` but application expected `/health`.
>
> **Solution:**
>
> ```yaml
> annotations:
>   alb.ingress.kubernetes.io/healthcheck-path: /health
> ```
>
> **Learning:** ALB has its own health checks separate from Kubernetes probes.
---
## 💡 Why These Tool Choices?
| Tool               | Why I Used It                              | Alternative Considered                         |
| ------------------ | ------------------------------------------ | ---------------------------------------------- |
| **EKS**            | Managed control plane, AWS integration     | Self-managed K8s (too much overhead)           |
| **Helm**           | Templating, version control, easy upgrades | Kustomize (less powerful for parameterization) |
| **ALB Controller** | Native AWS ALB with WAF, SSL support       | Nginx Ingress (no native AWS integration)      |
| **EBS CSI**        | Durable storage for databases              | EFS (unnecessary for block storage needs)      |
| **eksctl**         | Simple EKS cluster creation                | Terraform (more complex for simple clusters)   |
---
## 🎯 Key Metrics to Mention
- **12 microservices** deployed across the application
- **4 different databases/stores** (MongoDB, MySQL, Redis, RabbitMQ)
- **5 programming languages** (NodeJS, Java, Python, Go, PHP)
- **Single Helm command** deploys entire stack
- **0 hardcoded credentials** using IRSA
---
## 📝 Quick Reference Commands
```bash
# Create cluster
eksctl create cluster --name demo-cluster --region us-east-1
# Deploy with Helm
kubectl create ns robot-shop
helm install robot-shop --namespace robot-shop ./EKS/helm
# Apply ingress
kubectl apply -f EKS/helm/ingress.yaml
# Check deployment
kubectl get pods,svc,ingress -n robot-shop
# Get ALB URL
kubectl get ingress -n robot-shop -o jsonpath='{.items[0].status.loadBalancer.ingress[0].hostname}'
# Cleanup
helm uninstall robot-shop -n robot-shop
eksctl delete cluster --name demo-cluster
```
---
## 🔑 Keywords to Use in Interview
- Three-tier architecture
- Microservices
- Container orchestration
- Kubernetes / EKS
- Helm charts
- Infrastructure as Code
- OIDC / IRSA
- Load Balancer Controller
- Ingress / ALB
- Persistent Volumes / EBS CSI
- Service Discovery
- Polyglot architecture
- Horizontal Pod Autoscaling
- Rolling deployments
- Observability (monitoring, logging, tracing)
