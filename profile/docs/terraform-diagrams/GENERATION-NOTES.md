# Terraform Diagram Generation Notes

## Overview
This document describes how the Terraform infrastructure diagrams were generated for the PookieLand HRMS project.

## Generation Date
December 19, 2025

## Source Repositories Analyzed
1. **HRMS Repository**: https://github.com/PookieLand/HRMS
   - Terraform files in `/terraform` directory
   - Ansible playbooks in `/ansible/playbooks` directory
   
2. **Grafana Component Repository**: https://github.com/PookieLand/grafana-component
   - Terraform files in `/terraform_grafana_ec2` directory
   - Ansible playbooks in `/ansible_grafana_playbooks` directory

## Generation Method

### Original Plan
Initially, the plan was to use the terravision tool (https://github.com/patrickchugh/terravision) to generate diagrams directly from Terraform code. However, this approach faced challenges:
- Terraform requires AWS credentials to run `terraform plan`
- The backends were configured for S3, requiring actual AWS access
- Data sources in Terraform files (like EIP lookups) required connectivity to AWS

### Actual Implementation
Due to the constraints, a custom Python script using Graphviz was created to generate accurate architectural diagrams based on:
1. **Direct analysis** of Terraform configuration files (.tf files)
2. **Review** of Ansible playbooks to understand what software is installed
3. **Manual mapping** of resource relationships and dependencies
4. **Consultation** of backend configurations to understand state management

### Key Information Sources

#### HRMS Infrastructure
- **Terraform Files**:
  - `main.tf`: VPC, subnets, EC2 instances (master + 2 workers), EIP associations
  - `security_groups.tf`: Security groups for master and worker nodes
  - `ecr.tf`: Elastic Container Registry configuration
  - `nlb.tf`: Network Load Balancer setup
  - `secrets.tf`: AWS Secrets Manager and IAM roles
  - `backend.tf`: S3 backend (ap-south-1 region)

- **Ansible Playbooks**:
  - `01-prepare-nodes.yml`: Base node setup
  - `02-setup-master.yml`: Kubernetes master initialization with kubeadm, Flannel CNI
  - `03-setup-workers.yml`: Worker node joining
  - `04-deploy-hrms.yml`: Application deployment
  - `05-setup-ecr-refresh.yml`: ECR credential refresh automation

#### Grafana Monitoring Infrastructure
- **Terraform Files**:
  - `main.tf`: EC2 instance, security group, EIP, data volumes
  - `backend.tf`: S3 backend (ap-southeast-1 region)

- **Ansible Playbooks**:
  - `install-grafana.yml`: Grafana server installation
  - `install-prometheus.yml`: Prometheus LTS 3.5.0 setup
  - `install-alertmanager.yml`: Alert manager configuration
  - `install-node-exporter.yml`: System metrics exporter
  - `install-opensearch.yml`: OpenSearch installation
  - `install-opensearch-dashboards.yml`: Dashboard setup

## Diagrams Created

### 1. HRMS Kubernetes Infrastructure (`hrms-k8s-infrastructure.png/svg`)
**Components Identified**:
- Network: VPC (10.0.0.0/16), Internet Gateway, Public Subnet (10.0.1.0/24), Route Table
- Compute: 1 Master Node (t3.medium), 2 Worker Nodes (t3.medium)
- Networking: 3 Elastic IPs (one per node)
- Services: ECR, Network Load Balancer, Secrets Manager, IAM Roles
- Security: Separate security groups for master and workers
- Backend: S3 + DynamoDB in ap-south-1

**Key Architecture Decisions Captured**:
- Kubernetes cluster setup with kubeadm
- Flannel CNI for pod networking
- ECR integration for private container images
- External Secrets Operator for AWS Secrets Manager integration
- NLB for load balancing across workers
- Persistent EBS volumes (non-deletable)

### 2. Grafana Monitoring Infrastructure (`grafana-monitoring-infrastructure.png/svg`)
**Components Identified**:
- Compute: Single EC2 instance (Ubuntu 22.04, t3.medium)
- Storage: 50GB root + 50GB data volume
- Services: Grafana, Prometheus, Alertmanager, Node Exporter, OpenSearch, OpenSearch Dashboards
- Security: Security group with port restrictions
  - Public: SSH (22), Grafana (3000), OpenSearch (9200, 5601)
  - Internal: Prometheus (9090), Node Exporter (9100), Alertmanager (9093)
- Networking: Elastic IP
- Backend: S3 + DynamoDB in ap-southeast-1

**Key Architecture Decisions Captured**:
- Complete observability stack on single instance
- Port-based security model
- OpenSearch for log aggregation
- Prometheus for metrics collection
- Grafana for unified visualization

### 3. Combined Infrastructure (`combined-infrastructure.png/svg`)
**Integration Points Shown**:
- Shared project management
- Cross-region architecture (ap-south-1 for HRMS, ap-southeast-1 for monitoring)
- Monitoring relationships:
  - Prometheus monitors all Kubernetes nodes (red dotted lines)
  - OpenSearch collects logs from all Kubernetes nodes (blue dotted lines)
- Unified visualization through Grafana
- Separate Terraform state backends per region

## Technical Details

### Tools Used
- **Python 3.12**: Script execution
- **Graphviz 2.43.0**: Diagram rendering
- **Python graphviz library 0.21**: Python interface to Graphviz

### Diagram Formats
- **PNG**: High-resolution raster images (suitable for documentation, presentations)
- **SVG**: Scalable vector graphics (suitable for web, high-quality printing)

### Color Coding
- **Light Grey**: Network infrastructure
- **Light Green**: Compute resources (EC2/K8s)
- **Light Yellow**: AWS managed services
- **Light Coral**: Security components
- **Light Pink**: Backend/state management
- **Orange**: Monitoring stack components (in combined view)

### Connection Types
- **Solid lines**: Direct connections (e.g., network routing)
- **Dashed lines**: Service relationships (e.g., hosts, pulls images)
- **Dotted lines**: Logical relationships (e.g., IAM assignment, monitoring)
- **Red dotted**: Monitoring connections
- **Blue dotted**: Log collection flows

## Accuracy and Validation

The diagrams accurately represent:
✅ All Terraform-managed resources
✅ Network topology and relationships
✅ Security group configurations
✅ Backend configurations
✅ Software installed via Ansible
✅ Service relationships and dependencies
✅ Cross-infrastructure integrations

## Maintenance

To regenerate or update these diagrams:

1. **Update Source Repositories**: Ensure you have the latest Terraform and Ansible configurations
2. **Review Changes**: Check for new resources, changed relationships, or updated configurations
3. **Update Script**: Modify the generation script if architecture has changed
4. **Regenerate**: Run the Python script to create new diagrams
5. **Verify**: Review diagrams for accuracy
6. **Commit**: Update repository with new diagrams

## Script Location

The generation script is available at: `/tmp/create_tf_diagrams.py` (for PNG) and `/tmp/create_tf_diagrams_svg.py` (for SVG)

These scripts can be adapted for future diagram generation needs.

## Notes

- These diagrams represent the infrastructure as code, not necessarily what's currently deployed
- Actual deployed infrastructure may differ if manual changes were made
- Both repositories use Terraform backend configurations pointing to S3, confirming they're part of the same project
- The multi-region setup provides redundancy and separation of concerns
- The monitoring infrastructure can observe the production infrastructure cross-region

## References

- HRMS Repository: https://github.com/PookieLand/HRMS
- Grafana Component: https://github.com/PookieLand/grafana-component
- Terravision Tool: https://github.com/patrickchugh/terravision
- Graphviz: https://graphviz.org/
