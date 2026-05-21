# Platform Engineering Repository

## 1. Overview

This repository represents the **Enterprise Platform Engineering Foundation** for infrastructure provisioning, configuration management, and CI/CD standardization.

It provides a **centralized, governed, and reusable platform** for:

- Infrastructure provisioning using Terraform
- Configuration management using Ansible / AWX
- CI/CD standardization using GitLab shared pipeline libraries
- Runtime standardization using approved container images
- Secure secret management using GitLab CI/CD variables

---

## 2. Platform Architecture

```text
                     ┌──────────────────────────────┐
                     │        GitLab CI/CD          │
                     │  (Pipeline Orchestration)    │
                     └──────────────┬───────────────┘
                                    │
                ┌───────────────────┼───────────────────┐
                │                   │                   │
                ▼                   ▼                   ▼
     platform-pipelines     platform-images       Terraform Repos
   (CI/CD Templates)        (Execution Images)    (IaC Code Only)
                │                   │                   │
                └──────────────┬────┴────┬──────────────┘
                               │         │
                               ▼         ▼
                         Terraform       AWX
                     (Infra Creation) (Config Mgmt)
                               │
                               ▼
                           AWS Cloud


3. Core Components
3.1 platform-pipelines
infra_team/platform-pipelines

Contains reusable GitLab CI/CD pipeline templates.

Key Files
File	Purpose
terraform/aws.yml	Terraform pipeline (init, fmt, validate, plan, apply, destroy)
ansible/awx.yml	AWX integration (inventory sync, job launch, cleanup)
3.2 platform-images
infra_team/platform-images

Contains standardized container images used by pipelines.

Images
Image	Purpose
terraform-aws	Terraform + AWS CLI + jq
ansible-awx	Ansible + AWX CLI + boto3

These images ensure:

Consistent tooling
Version control
Security compliance
Reproducible pipelines
3.3 GitLab Runner

Runner executes pipelines.

Current Setup
Docker executor
Hosted on internal VM (vault node)
Tagged as:
platform

Used by all pipelines:

tags:
  - platform
3.4 GitLab Container Registry
registry.midhtech.local:5050

Stores platform images:

terraform-aws:1.0.0
ansible-awx:1.0.0
3.5 Terraform Repositories

Application teams maintain Terraform code.

Responsibilities
Define infrastructure
Use approved modules
Consume shared pipeline
Do NOT define pipeline logic
3.6 AWX (Ansible Automation Platform)
http://awx.midhtech.local

Used for:

Inventory management
Configuration execution
Playbook orchestration
4. Enterprise Design Principles
4.1 Separation of Concerns
Component	Responsibility
Terraform	Infrastructure creation
AWX	Configuration
GitLab	Orchestration
Pipelines	Execution logic
Images	Runtime tools
4.2 Centralized Governance
Pipelines are defined once
Images are version controlled
Apply is restricted
Destroy is controlled
Secrets are externalized
4.3 Idempotency
Terraform ensures infrastructure idempotency
AWX pipeline ensures inventory idempotency
4.4 Immutable Infrastructure + Declarative Config
Terraform → desired infrastructure
AWX → desired configuration
5. End-to-End Flow
Developer commits code
        ↓
GitLab pipeline triggered
        ↓
Terraform:
  fmt → validate → plan
        ↓
Manual approval
        ↓
Terraform apply
        ↓
AWS resources created
        ↓
Terraform outputs generated
        ↓
Pipeline converts outputs → AWX format
        ↓
AWX inventory sync (optional)
        ↓
AWX job execution (optional)
        ↓
Systems configured
6. Security Model
6.1 Secrets Management

Secrets are stored in:

GitLab CI/CD Variables

Never stored in:

Git repositories
Terraform files
Docker images
Pipeline templates
6.2 AWS Authentication

Current:

AWS_ACCESS_KEY + SECRET_KEY (CI variables)

Future (recommended):

GitLab OIDC → AWS IAM Role → Temporary credentials
6.3 AWX Authentication
Provided via CI variables
Uses service account
Password masked and protected
6.4 Execution Control
Operation	Control
Plan	automatic
Apply	manual, main branch only
Destroy	manual + change ticket
AWX Sync	optional/manual
Cleanup	optional/manual
7. CI/CD Governance
Terraform Rules
Non-main branches:
  fmt → validate → plan

Main branch:
  fmt → validate → plan → apply (manual)
Destroy Controls
Requires:
ALLOW_DESTROY=true
CHANGE_TICKET_ID
Manual approval
Main branch only
AWX Controls
Inventory sync → optional
Job execution → separate
Cleanup → optional + controlled
8. Image Versioning Strategy

Images are versioned:

terraform-aws:1.0.0
ansible-awx:1.0.0
Upgrade Flow
Build new image
Test internally
Update shared pipeline variable
Rollout to all repos

Optional override per repo:

TERRAFORM_IMAGE_TAG=1.0.1

9. Repository Usage Model
Terraform Repo Example
include:
  - project: infra_team/platform-pipelines
    ref: main
    file: terraform/aws.yml
Terraform + AWX Repo Example
include:
  - project: infra_team/platform-pipelines
    ref: main
    file: terraform/aws.yml
  - project: infra_team/platform-pipelines
    ref: main
    file: ansible/awx.yml

10. Responsibilities
Platform Team
Maintain shared pipelines
Maintain platform images
Manage runners
Enforce governance
Approve changes
Application / Infra Teams
Write Terraform code
Define infrastructure
Use shared pipelines
Configure variables
Raise merge requests

11. What This Platform Does NOT Do
Does not store secrets in code
Does not allow uncontrolled applies
Does not allow uncontrolled destroy
Does not embed credentials in images
Does not replace Terraform modules
Does not replace Ansible playbooks

12. Future Enhancements
OIDC integration with AWS
Multi-environment pipelines (dev/stage/prod)
Policy-as-Code (OPA / Sentinel)
Drift detection
Cost governance
Automated compliance scanning
RBAC enforcement for pipelines

13. Audit Statement

The platform enforces standardized, secure, and auditable infrastructure delivery by centralizing CI/CD logic, using version-controlled execution environments, externalizing secrets, and applying strict governance controls on infrastructure lifecycle operations.

14. Summary

This platform provides:

Consistency across all Terraform pipelines
Secure handling of credentials
Controlled infrastructure changes
Integration with AWX for configuration
Scalable enterprise architecture

It enables teams to focus on infrastructure definition, while the platform enforces execution standards and governance.