# AWS Workflow Suite for Online Book Store

This repository contains a comprehensive set of GitHub Actions workflows for deploying and managing an online book store application on AWS. The workflows provide end-to-end automation for infrastructure deployment, application deployment, monitoring, backup/disaster recovery, and security/compliance.

## 🏗️ Architecture Overview

The AWS workflows implement a robust cloud-native architecture including:

- **Containerized Application**: Deployed using Amazon ECS with Docker containers
- **Infrastructure as Code**: Managed with Terraform
- **Load Balancing**: Application Load Balancer (ALB) for high availability
- **Database**: Amazon RDS PostgreSQL with automated backups
- **Container Registry**: Amazon ECR for container image management
- **Monitoring**: CloudWatch metrics, logs, and alerting
- **Security**: Multi-layered security with GuardDuty, Security Hub, and compliance scanning
- **Disaster Recovery**: Cross-region backup and automated DR testing

## 📁 Workflow Files

### 1. Infrastructure Deployment (`aws-deploy-infrastructure.yml`)
**Purpose**: Deploys AWS infrastructure using Terraform

**Features**:
- Terraform plan and apply with state management
- Multi-environment support (dev, staging, production)
- Infrastructure validation and drift detection
- Secure backend state storage in S3

**Triggers**:
- Push to main branch (infrastructure changes)
- Pull requests (for planning)
- Manual dispatch with environment selection

### 2. Application Deployment (`aws-deploy-application.yml`)
**Purpose**: Builds, tests, and deploys the book store application

**Features**:
- Comprehensive testing (unit, integration, coverage)
- Security scanning with Trivy
- Docker image building and ECR push
- ECS service deployment with zero-downtime
- Database migrations
- Health checks and notifications

**Triggers**:
- Push to main branch (application changes)
- Pull requests (for testing)
- Manual dispatch with environment selection

### 3. Monitoring & Alerting (`aws-monitoring-alerts.yml`)
**Purpose**: Continuous monitoring and alerting for the application

**Features**:
- Application health checks every 15 minutes
- Performance monitoring (CPU, memory, response times)
- Cost monitoring with budget alerts
- Security monitoring with GuardDuty integration
- Database connectivity checks
- Real-time SNS notifications

**Triggers**:
- Scheduled (every 15 minutes)
- Push to main branch (monitoring configuration changes)
- Manual dispatch for immediate checks

### 4. Backup & Disaster Recovery (`aws-backup-disaster-recovery.yml`)
**Purpose**: Automated backup and disaster recovery operations

**Features**:
- Daily automated RDS snapshots
- S3 data backups with lifecycle management
- Application configuration backups
- Cross-region DR replication
- Weekly DR testing with automated cleanup
- Manual restore operations

**Triggers**:
- Scheduled daily backups (2 AM UTC)
- Scheduled weekly DR tests (Sunday 3 AM UTC)
- Manual dispatch for backup/restore operations

### 5. Security & Compliance (`aws-security-compliance.yml`)
**Purpose**: Comprehensive security scanning and compliance checking

**Features**:
- Vulnerability scanning with Trivy and AWS Inspector
- Secret detection with GitLeaks and TruffleHog
- Compliance checking (AWS Config, security groups, IAM)
- Automated penetration testing with OWASP ZAP
- Security monitoring setup (GuardDuty, Security Hub)
- Compliance reporting

**Triggers**:
- Daily security scans (1 AM UTC)
- Weekly compliance checks (Monday 2 AM UTC)
- Push/PR for code security scanning
- Manual dispatch with scan type selection

## 🔧 Setup Instructions

### Prerequisites

1. **AWS Account** with appropriate permissions
2. **GitHub Repository** with Actions enabled
3. **Terraform** state backend (S3 bucket)
4. **Domain** for the application (optional)

### Required GitHub Secrets

Add the following secrets to your GitHub repository:

```bash
# AWS Credentials
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_ACCOUNT_ID

# Optional Integrations
CODECOV_TOKEN          # For test coverage reporting
SLACK_WEBHOOK          # For deployment notifications
GITLEAKS_LICENSE       # For GitLeaks pro features
```

### Environment Setup

1. **Create GitHub Environments**:
   - `dev`
   - `staging` 
   - `production`
   - `dr-test`
   - `security-testing`

2. **Configure Environment Protection Rules**:
   - Require approval for production deployments
   - Restrict environment access to specific branches

### AWS Infrastructure Prerequisites

Before running the workflows, ensure the following AWS resources exist:

1. **S3 Buckets for Terraform State**:
   ```bash
   bookstore-terraform-state-dev
   bookstore-terraform-state-staging
   bookstore-terraform-state-production
   ```

2. **SNS Topics for Alerts**:
   ```bash
   bookstore-alerts-dev
   bookstore-alerts-staging
   bookstore-alerts-production
   bookstore-security-alerts
   ```

3. **ECR Repository**:
   ```bash
   bookstore-app
   ```

## 🚀 Usage Guide

### Infrastructure Deployment

1. **Initial Setup**:
   ```bash
   # Create infrastructure directory with Terraform files
   mkdir infrastructure
   # Add your Terraform configuration files
   ```

2. **Deploy Infrastructure**:
   - Push changes to `infrastructure/` directory
   - Workflow automatically plans and applies changes
   - Manual approval required for production

### Application Deployment

1. **Prepare Application**:
   ```bash
   # Ensure Dockerfile exists in root directory
   # Add package.json with required scripts:
   # - npm run lint
   # - npm test
   # - npm run test:integration
   # - npm run test:coverage
   # - npm run migrate
   ```

2. **Deploy Application**:
   - Push changes to `src/` directory
   - Workflow automatically builds, tests, and deploys
   - Health checks validate deployment success

### Monitoring Setup

1. **Configure Alerting**:
   - Update SNS topic subscriptions
   - Customize alert thresholds in workflow
   - Set up Slack notifications (optional)

2. **Monitor Application**:
   - Check GitHub Actions for monitoring status
   - Review CloudWatch dashboards
   - Respond to SNS alerts

### Backup & Recovery

1. **Automatic Backups**:
   - Daily backups run automatically
   - Cross-region replication for DR
   - 30-day retention policy

2. **Manual Restore**:
   ```yaml
   # Trigger manual restore workflow
   workflow_dispatch:
     inputs:
       operation: 'restore'
       environment: 'production'
       restore_point: 'snapshot-id-here'
   ```

### Security & Compliance

1. **Daily Security Scans**:
   - Automatic vulnerability scanning
   - Results posted to GitHub Security tab
   - Alerts sent for critical issues

2. **Manual Security Review**:
   ```yaml
   # Run specific security scans
   workflow_dispatch:
     inputs:
       scan_type: 'vulnerability' # or 'secrets', 'compliance'
   ```

## 📊 Workflow Outputs

### Infrastructure Deployment
- Terraform plan summaries
- Infrastructure resource details
- Cost estimates

### Application Deployment  
- Test coverage reports
- Security scan results
- Deployment status and URLs

### Monitoring
- Health check status
- Performance metrics
- Cost analysis

### Backup & Recovery
- Backup completion reports
- DR test results
- Recovery time metrics

### Security & Compliance
- Vulnerability reports
- Compliance status
- Security recommendations

## 🛡️ Security Best Practices

The workflows implement several security best practices:

1. **Least Privilege Access**: IAM roles with minimal required permissions
2. **Secret Management**: All sensitive data stored in GitHub Secrets
3. **Network Security**: Security groups with restrictive rules
4. **Encryption**: Data encrypted at rest and in transit
5. **Monitoring**: Continuous security monitoring with GuardDuty
6. **Compliance**: Regular compliance checks and reporting
7. **Vulnerability Management**: Automated scanning and alerting

## 🔄 Disaster Recovery

The DR strategy includes:

1. **RTO (Recovery Time Objective)**: ~20 minutes
2. **RPO (Recovery Point Objective)**: Last backup (daily)
3. **Cross-Region Replication**: Primary: us-east-1, DR: us-west-2
4. **Automated Testing**: Weekly DR tests with cleanup
5. **Runbook**: Documented recovery procedures

## 📈 Cost Optimization

Cost management features:

1. **Resource Tagging**: All resources tagged for cost allocation
2. **Cost Monitoring**: Daily cost alerts above thresholds
3. **Right-Sizing**: Automated scaling based on usage
4. **Storage Optimization**: Lifecycle policies for backups
5. **Reserved Instances**: Recommendations for production workloads

## 🐛 Troubleshooting

### Common Issues

1. **Terraform State Lock**:
   ```bash
   # Force unlock if workflow fails
   terraform force-unlock LOCK_ID
   ```

2. **ECR Authentication**:
   ```bash
   # Verify ECR repository exists and permissions are correct
   aws ecr describe-repositories --repository-names bookstore-app
   ```

3. **ECS Service Deployment**:
   ```bash
   # Check ECS service events for deployment issues
   aws ecs describe-services --cluster bookstore-cluster-env --services bookstore-service-env
   ```

4. **Health Check Failures**:
   - Verify load balancer target group health
   - Check application logs in CloudWatch
   - Ensure security groups allow traffic

### Workflow Debugging

1. **Enable Debug Logging**:
   ```yaml
   env:
     ACTIONS_RUNNER_DEBUG: true
     ACTIONS_STEP_DEBUG: true
   ```

2. **Check AWS Credentials**:
   - Verify secrets are properly configured
   - Ensure IAM permissions are sufficient
   - Check for expired credentials

## 📞 Support

For issues with the workflows:

1. Check GitHub Actions logs for detailed error messages
2. Review AWS CloudTrail for API call failures
3. Verify all prerequisites are met
4. Check AWS service status for any outages

## 🔄 Updates & Maintenance

Regular maintenance tasks:

1. **Monthly**: Review and update workflow dependencies
2. **Quarterly**: Update Terraform and AWS provider versions
3. **Annually**: Review and update security scanning tools
4. **As needed**: Update application dependencies and base images

## 📝 Contributing

To contribute to the workflows:

1. Fork the repository
2. Create a feature branch
3. Test changes in dev environment
4. Submit pull request with detailed description
5. Ensure all security scans pass

---

*This documentation provides a comprehensive guide to the AWS workflow suite. For specific implementation details, refer to the individual workflow files.*