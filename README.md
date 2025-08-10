# Manifests Applications

Kubernetes application manifests for the 40docs platform, managed via GitOps with Flux v2.

## Overview

This repository contains Kubernetes deployment manifests for all applications running on the 40docs platform's AKS cluster. Each application is structured with core manifests and dependency management for automated certificate provisioning and FortiWeb integration.

## Repository Structure

```
manifests-applications/
├── artifacts/              # Build artifacts management (Helm)
├── artifacts-dependencies/ # Certificate and FortiWeb setup for artifacts
├── docs/                   # Documentation hosting with nginx
├── docs-dependencies/      # Certificate and FortiWeb setup for docs
├── dvwa/                   # Damn Vulnerable Web Application
├── dvwa-dependencies/      # Certificate and FortiWeb setup for dvwa
├── extractor/              # Data processing application
├── extractor-dependencies/ # Certificate and FortiWeb setup for extractor
├── ollama/                 # AI/ML workloads with GPU support (Helm)
├── ollama-dependencies/    # Certificate and FortiWeb setup for ollama
├── pretix/                 # Event management platform (Helm)
├── pretix-dependencies/    # Certificate and FortiWeb setup for pretix
└── video/                  # Media streaming application
```

## Application Architecture

### Deployment Patterns

#### Standard Kubernetes Applications
- **Direct Manifests**: `docs`, `dvwa`, `extractor`, `video`
- **Components**: Deployment, Service, Ingress, ConfigMap, RBAC, HPA
- **Management**: Kustomize for resource aggregation

#### Helm-Managed Applications  
- **Helm Charts**: `artifacts`, `ollama`, `pretix`
- **Components**: HelmRelease, HelmRepository via Flux v2
- **Management**: GitOps-driven Helm deployments

#### Dependency Management
- **Certificate Provisioning**: Automated via cert-manager and Let's Encrypt
- **FortiWeb Integration**: Automated virtual server and policy configuration
- **Prerequisites**: Jobs that run before application deployment

### Security Integration

All applications integrate with FortiWeb for:
- **SSL Termination**: Automated certificate installation
- **Web Application Firewall**: Traffic inspection and protection  
- **Load Balancing**: High availability and health monitoring
- **Virtual Server Management**: Dedicated IP assignment per application

## Quick Start

### Prerequisites
- Azure AKS cluster with Flux v2 configured
- FortiWeb ingress controller installed
- cert-manager for TLS certificate automation
- DNS zone configured for automatic certificate validation

### Deployment
Applications are deployed automatically via GitOps when changes are pushed to the main branch.

### Manual Operations

```bash
# Validate manifests
kubectl apply --dry-run=client -k docs/
kubectl apply --dry-run=client -k dvwa/

# Check application status
kubectl get pods -n docs
kubectl get certificates -A
kubectl get ingress -A

# Monitor dependency jobs
kubectl get jobs -A
kubectl logs job/docs-dependencies -n docs

# Force Flux reconciliation (if needed)
flux reconcile source git infrastructure
```

## Applications

| Application | Type | Purpose | Status | Public URL |
|-------------|------|---------|--------|------------|
| docs | Direct | Documentation hosting | ✅ Active | `https://docs.{domain}` |
| dvwa | Direct | Security testing lab | ✅ Active | `https://dvwa.{domain}` |
| extractor | Direct | Data processing | ✅ Active | `https://extractor.{domain}` |
| ollama | Helm | AI/ML inference | 🔄 Optional | `https://ollama.{domain}` |
| artifacts | Helm | Build artifacts | 🔄 Optional | `https://artifacts.{domain}` |
| pretix | Helm | Event management | 🔄 Optional | `https://pretix.{domain}` |
| video | Direct | Media streaming | 🔄 Optional | `https://video.{domain}` |

## Configuration

### Resource Management
- **CPU/Memory**: Resource requests and limits defined for all applications
- **Storage**: Premium SSD storage class for persistent workloads
- **Scaling**: Horizontal Pod Autoscaler configured where appropriate
- **Anti-Affinity**: Pod distribution rules for high availability

### Networking
- **Ingress**: FortiWeb ingress controller with security policies
- **TLS**: Automated Let's Encrypt certificates via cert-manager
- **DNS**: Automatic CNAME records for application access
- **Network Policies**: Pod-to-pod communication controls

### Security
- **RBAC**: Minimal required permissions via service accounts
- **Security Context**: Non-root containers with security constraints
- **Secrets**: External secret management (no secrets in Git)
- **Monitoring**: Lacework security monitoring integration

## Development

### Adding New Applications

1. **Create Application Directory**
   ```bash
   mkdir new-app/
   mkdir new-app-dependencies/
   ```

2. **Application Manifests**
   - `Deployment.yaml` - Container orchestration
   - `Service.yaml` - Internal networking  
   - `Ingress.yaml` - External access with FortiWeb annotations
   - `kustomization.yaml` - Resource aggregation
   - `rbac.yaml` - Service account and permissions

3. **Dependencies**
   - `certificate.yaml` - TLS certificate resource
   - `Job.yaml` - FortiWeb configuration automation
   - `rbac.yaml` - Job service account permissions

4. **Testing**
   ```bash
   kubectl apply --dry-run=client -k new-app/
   kubectl apply --dry-run=client -k new-app-dependencies/
   ```

### FortiWeb Annotations

Required annotations for FortiWeb ingress integration:
```yaml
annotations:
  fortiweb-ip: "10.0.0.4"                    # FortiWeb appliance IP
  fortiweb-login: "fortiweb-login-secret"     # Login credentials secret
  virtual-server-ip: "10.0.0.x"              # Assigned VIP
  virtual-server-addr-type: "ipv4"
  virtual-server-interface: "port1"
  server-policy-web-protection-profile: "app-name"
  server-policy-https-service: "HTTPS"
  server-policy-http-service: "HTTP"
  server-policy-intermediate-certificate-group: "app-name_app-name_ca-group"
```

## Troubleshooting

### Common Issues

#### Certificate Problems
```bash
# Check certificate status
kubectl describe certificate app-tls -n app-namespace

# Check dependency job logs
kubectl logs job/app-dependencies -n app-namespace

# Force certificate renewal
kubectl delete certificate app-tls -n app-namespace
```

#### Application Deployment Issues
```bash
# Check pod status
kubectl get pods -n app-namespace
kubectl describe pod pod-name -n app-namespace

# Check ingress configuration
kubectl describe ingress -n app-namespace

# View application logs
kubectl logs deployment/app-name -n app-namespace
```

#### FortiWeb Integration Issues
```bash
# Check FortiWeb connectivity from job
kubectl exec -it job-pod -n app-namespace -- curl -k https://10.0.0.4

# Verify FortiWeb login secret
kubectl get secret fortiweb-login-secret -o yaml

# Check virtual server assignment
kubectl get ingress -o yaml | grep virtual-server-ip
```

## GitOps Integration

This repository is managed by Flux v2 GitOps:
- **Automatic Deployment**: Changes to main branch trigger deployments
- **Dependency Management**: Jobs ensure prerequisites before app deployment  
- **Rollback Support**: Git-based rollback capabilities
- **Multi-Environment**: Support for dev/staging/production configurations

## Security Considerations

- **Zero Trust**: All applications behind FortiWeb security inspection
- **Certificate Management**: Automated TLS with short-lived certificates
- **Secret Management**: External secrets, no credentials in Git
- **Network Segmentation**: Namespace isolation and network policies
- **Security Monitoring**: Lacework runtime protection

## Contributing

1. **Fork and Branch**: Create feature branch from main
2. **Validate Manifests**: Test with `kubectl apply --dry-run=client`
3. **Security Review**: Ensure no secrets or credentials in code
4. **Documentation**: Update README.md for new applications
5. **Pull Request**: Submit PR with clear description and testing notes

## Support

- **Documentation**: See [CLAUDE.md](CLAUDE.md) for detailed development guidance
- **Issues**: Report issues in the main 40docs repository
- **Security**: Follow responsible disclosure for security issues

## License

Part of the 40docs platform - Enterprise Documentation as Code ecosystem.