# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the **manifests-applications** repository - part of the 40docs platform's Kubernetes application deployment infrastructure. It contains Kubernetes manifests for deploying and managing applications in the AKS cluster, organized using Kustomize for configuration management.

## Architecture and Structure

### Application Organization
Each application follows a consistent pattern with two main directories:
- **`{app}/`**: Core application manifests (Deployment, Service, Ingress, etc.)
- **`{app}-dependencies/`**: Prerequisite jobs and certificates needed before app deployment

### Key Applications
- **docs**: Documentation hosting with nginx and authentication
- **dvwa**: Damn Vulnerable Web Application for security testing
- **extractor**: Data processing application with persistent storage
- **ollama**: AI/ML workloads with GPU support and Helm-based deployment
- **artifacts**: Build artifacts management (Helm-based)
- **pretix**: Event management platform (Helm-based)
- **video**: Media streaming application

### Deployment Patterns

#### Standard Kubernetes Applications
Applications like `docs`, `dvwa`, `extractor`, and `video` use direct Kubernetes manifests:
- `Deployment.yaml`: Container orchestration with resource limits and health checks
- `Service.yaml`: Internal networking configuration
- `Ingress.yaml`: External access through FortiWeb ingress controller
- `kustomization.yaml`: Resource aggregation and management
- `rbac.yaml`: Role-based access control

#### Helm-Managed Applications
Applications like `ollama`, `artifacts`, and `pretix` use Flux HelmRelease:
- `HelmRelease.yaml`: Flux v2 Helm deployment configuration
- `HelmRepository.yaml`: Helm chart source repository
- `kustomization.yaml`: Resource management

#### Dependency Management
All applications have corresponding `-dependencies` directories containing:
- `Job.yaml`: Kubernetes job for certificate management and FortiWeb configuration
- `certificate.yaml`: cert-manager certificate provisioning
- `rbac.yaml`: Service account and permissions for dependency jobs

## Common Development Tasks

### Validating Manifests
```bash
# Validate all Kubernetes YAML files
find . -name "*.yaml" -exec kubectl --dry-run=client apply -f {} \;

# Check kustomization builds
kubectl kustomize docs/
kubectl kustomize dvwa/
kubectl kustomize extractor/
```

### Working with Applications

#### Standard Applications (Direct Manifests)
```bash
# Apply application manifests
kubectl apply -k docs/
kubectl apply -k dvwa/
kubectl apply -k extractor/

# Check application status
kubectl get pods -n docs
kubectl get svc -n dvwa
kubectl describe ingress -n extractor
```

#### Helm Applications (via Flux)
```bash
# Check Helm release status
kubectl get helmrelease -n cluster-config
kubectl describe helmrelease ollama -n cluster-config

# Force reconciliation
flux reconcile helmrelease ollama
```

### Certificate and Dependency Management
```bash
# Check certificate status
kubectl get certificates -A
kubectl describe certificate docs-tls -n docs

# Monitor dependency jobs
kubectl get jobs -A
kubectl logs job/docs-dependencies -n docs
```

### FortiWeb Integration
Applications use the FortiWeb ingress controller with specific annotations for:
- SSL offloading and certificate management
- Web application firewall protection
- Load balancing and health checks
- Virtual server IP assignment

Key FortiWeb annotations:
- `fortiweb-ip`: FortiWeb appliance IP address
- `virtual-server-ip`: Assigned virtual server IP
- `server-policy-*`: Security and SSL policies

## Configuration Patterns

### Resource Management
- **CPU/Memory Limits**: All deployments include resource requests and limits
- **Health Checks**: ReadinessProbe and LivenessProbe configured for reliability
- **Anti-Affinity**: Pod anti-affinity rules prevent single points of failure
- **Security Context**: Non-root containers and security constraints

### Persistent Storage
- **Storage Class**: `managed-premium` for Azure Premium SSD
- **Volume Claims**: Dedicated PVCs for applications requiring persistence
- **Mount Paths**: Consistent volume mounting patterns

### Networking and Security
- **Namespace Isolation**: Each application runs in its own namespace
- **RBAC**: Minimal required permissions via service accounts
- **Network Policies**: Pod-to-pod communication controls
- **TLS Termination**: Let's Encrypt certificates via cert-manager

### Dependency Chain Management
The dependency jobs handle:
1. Certificate readiness verification
2. Intermediate certificate extraction and upload to FortiWeb
3. FortiWeb virtual server policy configuration
4. Certificate group creation and management

## Troubleshooting

### Certificate Issues
```bash
# Check certificate status
kubectl describe certificate {app}-tls -n {namespace}
kubectl get certificaterequest -n {namespace}

# Check cert-manager logs
kubectl logs -n cert-manager -l app=cert-manager
```

### FortiWeb Integration Issues
```bash
# Check dependency job logs
kubectl logs job/{app}-dependencies -n {namespace}

# Verify FortiWeb connectivity
kubectl exec -it {pod} -n {namespace} -- curl -k https://10.0.0.4
```

### Application Deployment Issues
```bash
# Check pod status and logs
kubectl get pods -n {namespace}
kubectl describe pod {pod-name} -n {namespace}
kubectl logs {pod-name} -n {namespace}

# Check ingress configuration
kubectl describe ingress -n {namespace}
```

### Resource Constraints
```bash
# Check resource usage
kubectl top pods -n {namespace}
kubectl describe node {node-name}

# Check persistent volume status
kubectl get pv,pvc -n {namespace}
```

## Security Considerations

- All applications run with non-root security contexts
- Resource limits prevent resource exhaustion attacks
- RBAC follows principle of least privilege
- Network segmentation through namespaces
- TLS encryption for all external traffic
- Regular security scanning via FortiWeb WAF policies

## Integration with 40docs Platform

This repository is part of the larger 40docs GitOps workflow:
- **Flux v2**: Automatic deployment via GitOps
- **cert-manager**: Automated TLS certificate management
- **FortiWeb**: Web application firewall and load balancing
- **Azure AKS**: Managed Kubernetes cluster deployment
- **Monitoring**: Lacework security monitoring integration