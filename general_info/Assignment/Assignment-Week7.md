# Assignment Week 7 - End-to-End Kubernetes Project with Security and Observability

## Deadline
**Presentation Date:** December 18th or 19th, 2025 at 8:00 PM

---

## Question 1: Enable Logs within OpenTelemetry on Datadog

Configure and enable log collection within the OpenTelemetry collector to send application logs to Datadog alongside your existing traces and metrics.

**Requirements:**
- Configure the OpenTelemetry collector to receive logs
- Set up appropriate processors for log data
- Export logs to Datadog
- Verify logs are visible in the Datadog Logs dashboard

---

## Question 2: Implement Security Best Practices

Implement comprehensive security standards across the following components:

### Components to Secure:
1. **Government API Application**
2. **Loan Validator Portal**
3. **Shared Kubernetes Manifests**

### Security Requirements:
- Implement proper RBAC (Role-Based Access Control)
- Use Network Policies to restrict pod-to-pod communication
- Apply Pod Security Standards/Policies
- Implement secrets management (avoid hardcoded credentials)
- Configure resource limits and requests
- Use non-root containers where possible
- Implement image scanning and use trusted registries
- Apply security contexts to pods
- Use read-only root filesystems where applicable
- Implement proper health checks and liveness/readiness probes

---

## Question 3: Deploy a Managed Kubernetes Cluster with Security Best Practices

Deploy a production-ready managed Kubernetes cluster on your cloud provider of choice.

### Choose ONE of the following:
- **AWS EKS (Elastic Kubernetes Service)**
- **GCP GKE (Google Kubernetes Engine)**
- **Azure AKS (Azure Kubernetes Service)**

### Requirements:
- Build the cluster using security best practices
- Implement proper networking (VPC, subnets, security groups)
- Enable cluster logging and monitoring
- Configure node security (encryption, IAM roles, security groups)
- Implement cluster autoscaling

### Bonus Points:
Use Infrastructure as Code (IaC) for cluster provisioning:
- **eksctl** (for AWS EKS)
- **Terraform** (any cloud provider)
- **Crossplane** (any cloud provider)
- Other IaC tools

---

## Question 4: Domain, Ingress, and TLS Configuration

Set up a complete production-ready ingress configuration with a custom domain.

### Requirements:

#### 4.1 Domain Registration
- Register a domain name (can be done on Namecheap or any domain registrar)

#### 4.2 Ingress Controller Setup
- Deploy an NGINX Ingress Controller
- Ensure the ingress controller service creates a LoadBalancer with an A record

#### 4.3 DNS Configuration
- Create a CNAME record that maps your domain to the LoadBalancer A record
- Configure DNS so that `www.your-domain.com` routes to your cluster

#### 4.4 HTTPS Redirect
- Configure automatic HTTP to HTTPS redirect
- Set up TLS/SSL certificates (use cert-manager with Let's Encrypt for bonus points)

#### 4.5 Ingress Resource
- Create an Ingress resource with proper routing rules
- Configure the root path (`/`) to serve the Loan Validator Portal
- Verify that visiting `https://www.your-domain.com` returns the Loan Validator Portal

#### 4.6 Government API Access
The Government API needs to be accessible for POST requests to add customer records.

**Challenge:** 
Currently, there is no way to reach the Government API to invoke a POST command. How can you configure the ingress and routing so that you can successfully execute the following command?

```bash
curl -X POST http://your-domain:8082/api/customer \
  -H "Content-Type: application/json" \
  -d '{
    "first_name": "Bob",
    "last_name": "Williams",
    "date_of_birth": "1988-03-20",
    "loan_amount_requested": 75000.00,
    "loan_status": "Pending"
  }'
```

**Tasks:**
- Design and implement the appropriate ingress path routing
- Ensure the Government API endpoint is accessible
- Test the POST request successfully creates a customer record
- Document your solution and reasoning

---

## Question 5: ArgoCD Implementation

Deploy and configure ArgoCD for GitOps-based continuous deployment.

### Requirements:
- Install ArgoCD in your Kubernetes cluster
- Create ArgoCD applications for:
  - Government API
  - Loan Validator Portal
  - Shared infrastructure components
- Configure automated sync policies
- Implement proper Git repository structure for ArgoCD
- Demonstrate self-healing capabilities
- Show how changes in Git trigger automatic deployments

---

## Project Deliverables

### 1. Documentation
- Architecture diagram showing all components
- Step-by-step setup instructions
- Security measures implemented
- DNS and ingress configuration details
- ArgoCD setup and workflow

### 2. Code Repository
- All Kubernetes manifests
- IaC code (Terraform/eksctl/etc.)
- ArgoCD application definitions
- Configuration files
- Scripts for automation

### 3. Live Demonstration
- Working application accessible via custom domain
- HTTPS working correctly
- Successful POST request to Government API
- Logs visible in Datadog
- ArgoCD dashboard showing synchronized applications

### 4. Presentation (December 18th or 19th at 8:00 PM)
- Overview of your architecture
- Security implementations
- Demo of the working application
- Challenges faced and solutions
- Q&A session

---

## Evaluation Criteria

| Criteria | Points |
|----------|--------|
| OpenTelemetry Logs Configuration | 15% |
| Security Implementation | 25% |
| Managed Kubernetes Cluster Setup | 20% |
| Domain, Ingress, and TLS Setup | 20% |
| ArgoCD Implementation | 15% |
| Documentation Quality | 5% |
| Bonus: IaC Implementation | +10% |

---

## Tips for Success

1. **Start Early** - This is a comprehensive project covering multiple topics
2. **Test Incrementally** - Verify each component works before moving to the next
3. **Document as You Go** - Keep notes of commands and configurations
4. **Use Version Control** - Commit your changes frequently
5. **Security First** - Don't compromise on security for convenience
6. **Ask Questions** - Reach out if you're stuck on any component

---

## Resources

- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)
- [Datadog OpenTelemetry Integration](https://docs.datadoghq.com/opentelemetry/)
- [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/)
- [NGINX Ingress Controller Documentation](https://kubernetes.github.io/ingress-nginx/)
- [cert-manager Documentation](https://cert-manager.io/docs/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [eksctl Documentation](https://eksctl.io/)
- [Terraform Kubernetes Provider](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs)

---

**Good Luck! 🚀**







---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: government-api-role
  namespace: application
  labels:
    app: government-api
rules:
# Allow reading own ConfigMap and Secret
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  resourceNames: ["government-api-config", "government-api-secret"]
  verbs: ["get", "list"]
# Allow reading endpoints for service discovery
- apiGroups: [""]
  resources: ["endpoints"]
  verbs: ["get", "list"]
# Allow reading services for service discovery
- apiGroups: [""]
  resources: ["services"]
  resourceNames: ["government-api-service", "postgres-service"]
  verbs: ["get", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: loan-validator-portal-role
  namespace: application
  labels:
    app: loan-validator-portal
rules:
# Allow reading own ConfigMap and Secret
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  resourceNames: ["loan-validator-portal-config", "loan-validator-portal-secret"]
  verbs: ["get", "list"]
# Allow reading endpoints for service discovery
- apiGroups: [""]
  resources: ["endpoints"]
  verbs: ["get", "list"]
# Allow reading services for service discovery
- apiGroups: [""]
  resources: ["services"]
  resourceNames: ["loan-validator-portal-service", "government-api-service"]
  verbs: ["get", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: postgres-role
  namespace: application
  labels:
    app: postgres
rules:
# Allow reading own ConfigMap and Secret
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  resourceNames: ["postgres-init-scripts", "postgres-secret"]
  verbs: ["get", "list"]
# Allow reading persistent volume claims
- apiGroups: [""]
  resources: ["persistentvolumeclaims"]
  resourceNames: ["postgres-pvc"]
  verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: government-api-binding
  namespace: application
  labels:
    app: government-api
subjects:
- kind: ServiceAccount
  name: government-api-sa
  namespace: application
roleRef:
  kind: Role
  name: government-api-role
  apiGroup: rbac.authorization.k8s.io

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: loan-validator-portal-binding
  namespace: application
  labels:
    app: loan-validator-portal
subjects:
- kind: ServiceAccount
  name: loan-validator-portal-sa
  namespace: application
roleRef:
  kind: Role
  name: loan-validator-portal-role
  apiGroup: rbac.authorization.k8s.io

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: postgres-binding
  namespace: application
  labels:
    app: postgres
subjects:
- kind: ServiceAccount
  name: postgres-sa
  namespace: application
roleRef:
  kind: Role
  name: postgres-role
  apiGroup: rbac.authorization.k8s.io

---
# Allow Loan Validator Portal to access Government API
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-government-api
  namespace: application
spec:
  podSelector:
    matchLabels:
      app: government-api
  policyTypes:
  - Ingress
  ingress:
  # Allow Loan Validator Portal
  - from:
    - podSelector:
        matchLabels:
          app: loan-validator-portal
    ports:
    - protocol: TCP
      port: 8081
  # Allow health checks from kubelet
  - from:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 8081

---
# Allow external access to Loan Validator Portal
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-loan-validator-portal
  namespace: application
spec:
  podSelector:
    matchLabels:
      app: loan-validator-portal
  policyTypes:
  - Ingress
  - Egress
  ingress:
  # Allow all ingress (public-facing service)
  - {}
  egress:
  # Allow egress to Government API
  - to:
    - podSelector:
        matchLabels:
          app: government-api
    ports:
    - protocol: TCP
      port: 8081
  # Allow egress to PostgreSQL
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    ports:
    - protocol: TCP
      port: 5432
  # Allow DNS
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53

---
# Allow only Government API and Loan Validator to access PostgreSQL
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-postgres
  namespace: application
spec:
  podSelector:
    matchLabels:
      app: postgres
  policyTypes:
  - Ingress
  ingress:
  # Allow Government API
  - from:
    - podSelector:
        matchLabels:
          app: government-api
    ports:
    - protocol: TCP
      port: 5432
  # Allow Loan Validator Portal
  - from:
    - podSelector:
        matchLabels:
          app: loan-validator-portal
    ports:
    - protocol: TCP
      port: 5432

---
# Default deny-all network policy
# Apply this first, then add allow policies
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: application
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

