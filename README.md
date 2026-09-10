**jenkins-helm-automated-rollback**

```markdown
# Automated Canary Release & Rollback Pipeline (Jenkins & Helm)

## Overview
This repository demonstrates a resilient CI/CD pipeline that prioritizes system stability over deployment speed. Instead of blindly pushing code to production, this pipeline executes a deployment, actively monitors application health, and automatically rolls back the release if Service Level Indicators (SLIs) degrade.

This workflow eliminates the need for manual intervention during a bad release, directly reducing Mean Time to Resolution (MTTR) and protecting end-user experience from flawed code.

## Architecture & SRE Controls
* **Helm Package Management:** Microservices are packaged as Helm charts, allowing for versioned, atomic deployments and instant state reversions.
* **Progressive Delivery CI/CD:** A multi-stage Jenkins declarative pipeline that handles building, deploying, and post-deployment verification.
* **Automated Health Verification:** The pipeline executes automated scripts immediately after a Helm upgrade to verify HTTP status codes and application responsiveness.
* **Zero-Touch Rollbacks:** If the post-deployment script detects a spike in HTTP 5xx errors or a failed health check, Jenkins automatically executes a `helm rollback` to instantly restore the previous stable state before terminating the build.

## Usage Instructions
```bash
# The Jenkinsfile is designed to be executed by a Jenkins server. 
# To test the logic manually via CLI:

# 1. Install the Helm chart
helm upgrade --install sample-app ./helm-chart --namespace production --create-namespace

# 2. Trigger the health check script
curl -s -f http://<service-endpoint>/health || exit 1

# 3. Execute rollback if the health check fails
helm rollback sample-app 0 --namespace production
