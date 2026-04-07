---
name: controller-script
description: Generate a controller.sh bash script for a service with stop, start, deploy, destroy, build, logs, and connect commands wired to Terraform, Ansible, Docker, ECS, or Lambda.
---

You are acting as the devops-aws agent. Generate a `controller.sh` for a deployable service.

**Step 1 — Gather service information (ask if not provided):**
- Service name? (e.g., `my-api`, `data-processor`, `eo-pipeline`)
- Deployment target? (ECS Fargate / EC2 + Docker / Lambda via Zappa / ECS + EC2)
- AWS region and profile?
- ECR repository name? (for Docker-based services)
- CloudWatch log group name?
- Does it use Terraform? (path to terraform directory)
- Does it use Ansible? (playbook path)
- SSM parameter path or instance tag for `connect` command?

**Step 2 — Read existing deployment config:**
Check for existing `Dockerfile`, `docker-compose.yml`, `zappa_settings.json`, `serverless.yaml`, or Terraform files.

**Step 3 — Generate `controller.sh`:**

```bash
#!/usr/bin/env bash
# =============================================================================
# controller.sh — <SERVICE_NAME> deployment controller
# Usage: ./controller.sh <command>
# Commands: build | deploy | start | stop | destroy | logs | connect | status
# =============================================================================
set -euo pipefail

SERVICE_NAME="<service-name>"
AWS_REGION="${AWS_REGION:-eu-central-1}"
AWS_PROFILE="${AWS_PROFILE:-default}"
ECR_REPO="<account-id>.dkr.ecr.${AWS_REGION}.amazonaws.com/<ecr-repo-name>"
TF_DIR="./infrastructure"
LOG_GROUP="/ecs/<service-name>"
CLUSTER_NAME="<ecs-cluster-name>"
SERVICE_ECS_NAME="<ecs-service-name>"

# Colors
RED='\033[0;31m'; GREEN='\033[0;32m'; YELLOW='\033[1;33m'; NC='\033[0m'

log()  { echo -e "${GREEN}[$(date +%H:%M:%S)] $*${NC}"; }
warn() { echo -e "${YELLOW}[$(date +%H:%M:%S)] $*${NC}"; }
err()  { echo -e "${RED}[$(date +%H:%M:%S)] ERROR: $*${NC}" >&2; }

# --- build ---
cmd_build() {
    log "Building Docker image..."
    docker build -t "${SERVICE_NAME}:latest" .
    log "Authenticating to ECR..."
    aws ecr get-login-password --region "${AWS_REGION}" --profile "${AWS_PROFILE}" \
        | docker login --username AWS --password-stdin "${ECR_REPO%%/*}"
    log "Tagging and pushing..."
    docker tag "${SERVICE_NAME}:latest" "${ECR_REPO}:latest"
    docker push "${ECR_REPO}:latest"
    log "Build complete: ${ECR_REPO}:latest"
}

# --- deploy ---
cmd_deploy() {
    log "Running terraform apply..."
    terraform -chdir="${TF_DIR}" apply -auto-approve
    log "Forcing ECS service redeployment..."
    aws ecs update-service \
        --cluster "${CLUSTER_NAME}" \
        --service "${SERVICE_ECS_NAME}" \
        --force-new-deployment \
        --region "${AWS_REGION}" \
        --profile "${AWS_PROFILE}" > /dev/null
    log "Deployment triggered. Monitor with: ./controller.sh logs"
}

# --- start ---
cmd_start() {
    log "Setting ECS service desired count to 1..."
    aws ecs update-service \
        --cluster "${CLUSTER_NAME}" \
        --service "${SERVICE_ECS_NAME}" \
        --desired-count 1 \
        --region "${AWS_REGION}" \
        --profile "${AWS_PROFILE}" > /dev/null
    log "Service starting. Use './controller.sh status' to check."
}

# --- stop ---
cmd_stop() {
    warn "Stopping service (desired count → 0). Data is preserved."
    aws ecs update-service \
        --cluster "${CLUSTER_NAME}" \
        --service "${SERVICE_ECS_NAME}" \
        --desired-count 0 \
        --region "${AWS_REGION}" \
        --profile "${AWS_PROFILE}" > /dev/null
    log "Service stopped."
}

# --- destroy ---
cmd_destroy() {
    err "WARNING: This will DESTROY all Terraform-managed infrastructure for ${SERVICE_NAME}!"
    warn "This includes: ECS service, RDS instances, S3 buckets (if not versioned), SQS queues."
    read -rp "Type DESTROY to confirm: " confirmation
    if [[ "${confirmation}" != "DESTROY" ]]; then
        log "Aborted."
        exit 0
    fi
    log "Running terraform destroy..."
    terraform -chdir="${TF_DIR}" destroy -auto-approve
    log "Infrastructure destroyed."
}

# --- logs ---
cmd_logs() {
    log "Tailing CloudWatch logs: ${LOG_GROUP}"
    aws logs tail "${LOG_GROUP}" \
        --follow \
        --format short \
        --region "${AWS_REGION}" \
        --profile "${AWS_PROFILE}"
}

# --- connect ---
cmd_connect() {
    log "Finding running ECS task..."
    TASK_ARN=$(aws ecs list-tasks \
        --cluster "${CLUSTER_NAME}" \
        --service-name "${SERVICE_ECS_NAME}" \
        --desired-status RUNNING \
        --region "${AWS_REGION}" \
        --profile "${AWS_PROFILE}" \
        --query "taskArns[0]" --output text)
    if [[ -z "${TASK_ARN}" || "${TASK_ARN}" == "None" ]]; then
        err "No running tasks found for service ${SERVICE_ECS_NAME}."
        exit 1
    fi
    log "Connecting to task: ${TASK_ARN}"
    aws ecs execute-command \
        --cluster "${CLUSTER_NAME}" \
        --task "${TASK_ARN}" \
        --container "${SERVICE_NAME}" \
        --command "/bin/bash" \
        --interactive \
        --region "${AWS_REGION}" \
        --profile "${AWS_PROFILE}"
}

# --- status ---
cmd_status() {
    log "Service status for ${SERVICE_ECS_NAME}:"
    aws ecs describe-services \
        --cluster "${CLUSTER_NAME}" \
        --services "${SERVICE_ECS_NAME}" \
        --region "${AWS_REGION}" \
        --profile "${AWS_PROFILE}" \
        --query "services[0].{Status:status,Running:runningCount,Desired:desiredCount,Pending:pendingCount,LastDeploy:deployments[0].updatedAt}" \
        --output table
}

# --- main ---
COMMAND="${1:-help}"
case "${COMMAND}" in
    build)   cmd_build   ;;
    deploy)  cmd_deploy  ;;
    start)   cmd_start   ;;
    stop)    cmd_stop    ;;
    destroy) cmd_destroy ;;
    logs)    cmd_logs    ;;
    connect) cmd_connect ;;
    status)  cmd_status  ;;
    *)
        echo "Usage: ./controller.sh <command>"
        echo "Commands:"
        echo "  build    — Docker build + ECR push"
        echo "  deploy   — terraform apply + ECS force redeploy"
        echo "  start    — Set ECS desired count to 1"
        echo "  stop     — Set ECS desired count to 0 (data preserved)"
        echo "  destroy  — terraform destroy (REQUIRES confirmation)"
        echo "  logs     — Tail CloudWatch logs"
        echo "  connect  — ECS Exec into running container"
        echo "  status   — Show ECS service status"
        exit 1
        ;;
esac
```

**Adapt the template** to the actual deployment target (Lambda/Zappa, EC2, ECS) based on the service information gathered in Step 1. Fill in all placeholder values from project config.

Make the script executable: `chmod +x controller.sh`
