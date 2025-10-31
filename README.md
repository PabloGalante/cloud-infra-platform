# Multi-Environment Cloud Infrastructure Platform

## Overview

This repository contains an example Infrastructure-as-Code (IaC) platform intended to demonstrate a complete set of cloud infrastructure engineering skills. It provides reusable Terraform modules, a multi-environment layout, CI/CD automation, documentation, and operational runbooks. The structure is opinionated to illustrate common patterns for networking, compute, storage, security, monitoring, and deployment across dev, staging, and production.

## Project structure

terraform-infrastructure-platform/

- modules/                    # Reusable Terraform modules
  - networking/               # VPC, subnets, routing, NAT, gateways
  - compute/                  # Compute instances, auto-scaling groups
  - storage/                  # Object storage configuration (S3, GCS)
  - security/                 # Security groups, IAM roles and policies
  - monitoring/               # Metrics, logs, alerts (CloudWatch / Stackdriver)
- environments/               # Environment-specific configs and state
  - dev/
  - staging/
  - production/
- docs/                       # Architecture diagrams, requirements, runbooks
  - architecture-diagram.md
  - requirements.md
  - runbooks.md
- .github/workflows/          # CI/CD pipeline definitions

## Key features

- Requirements documentation that translates fictional engineering needs into technical specs.
- Reusable Terraform modules: networking (multi-AZ VPC), application tier (load-balanced instances), database tier (RDS/Cloud SQL), storage, and monitoring.
- Multi-environment support: dev, staging, and production with environment-specific variables and remote state backends.
- CI/CD automation: GitHub Actions for format/validate/plan, security scanning (tfsec/checkov), and optional automated applies for non-prod.
- Documentation: ADRs, runbooks, architecture diagrams and cost-optimization guidelines.

## User manual

This section describes how to use the repository: how to bootstrap, work with modules, run Terraform per-environment, and interact with CI/CD.

### Prerequisites

- Terraform.
- Cloud CLI(s) for your provider (e.g., AWS CLI, gcloud) configured with appropriate credentials.
- Git and a GitHub account if you want to use the included GitHub Actions workflows.
- Optional tools: tfenv, direnv, or other environment helpers.

### Bootstrap (local testing)

1. Copy example environment files (if present) into `environments/<env>` or create a new environment folder.
1. Configure provider credentials and backend in `environments/<env>/backend.tf` and provider blocks.
1. Initialize Terraform in the environment directory:

```sh
terraform init
```

1. Create a plan:

```sh
terraform plan -out=tfplan
```

1. Apply locally (use caution and prefer dev for testing):

```sh
terraform apply tfplan
```

Notes:

- Keep secrets out of the repo: use CI secrets or a secrets manager. Do not commit `terraform.tfvars` with secrets.
- Protect production state with access controls and require approvals before apply.

### Working with modules

- Modules are located under `modules/`. Each module should include a `README.md` that documents inputs, outputs and examples.
- Example module usage from an environment `main.tf`:

```hcl
module "vpc" {
  source = "../../modules/networking"
  name   = var.project_name
  cidr   = var.vpc_cidr
}
```

- When changing a module, follow semantic versioning and test changes in `dev` before promoting to `staging` and `production`.

### Environments and remote state

- Each environment directory should define its own backend configuration (e.g., S3 + DynamoDB for AWS remote state locking).
- Files to keep per-environment: `backend.tf`, `providers.tf`, `variables.tf`, and (gitignored) `terraform.tfvars` for secrets.
- Consider using a shared state prefix per environment to avoid collisions.

### CI/CD pipeline

- Workflows (GitHub Actions) under `.github/workflows/` typically perform:
  - terraform fmt, validate
  - terraform init & plan (per-environment)
  - security scanning: tfsec and/or checkov
  - publish plan artifacts to PRs for review
  - optional: apply in non-prod or on protected workflows with manual approval for prod

- Store provider credentials and backend access tokens in GitHub Secrets or your CI's secret store.

### Security and policy guidance

- Use least-privilege service accounts for automation.
- Run static security checks (tfsec/checkov) in CI and block merges on critical findings.
- Do not store secrets or long-lived credentials in the repository.

### Monitoring and alerting

- The `monitoring` module should configure metrics, log groups, dashboards, and alerting rules.
- Configure notification channels (email, Slack, PagerDuty) via secure environment variables or secrets.

### Cost optimization

- Use smaller instance sizes in dev and reserve larger sizes for production.
- Prefer managed services when operational savings justify cost.
- Tag resources with `cost_center`, `environment`, and `owner` for reporting.

## Docs & runbooks

- `docs/requirements.md` — maps business requirements to infrastructure specs.
- `docs/architecture-diagram.md` — architecture diagrams and component responsibilities.
- `docs/runbooks.md` — operational runbooks for incident response, backup, and recovery.
- `docs/adr/` — keep Architecture Decision Records here.

## Contributing

- Open an issue to propose changes.
- Create a feature branch, add tests or verification steps, and open a pull request.
- Follow semantic versioning for module changes and document breaking changes in ADRs.

## Contact

Open an issue or contact the repository owner for questions or feature requests.

----

Last updated: 2025-10-31

*** End Patch
