# OSAC Project Context

**Last Updated:** 2026-06-23

## Overview

OSAC (Open Sovereign AI Cloud) is a fulfillment system for provisioning Kubernetes clusters, virtual machines, and bare metal instances with networking capabilities. It is a multi-tenant cloud platform built on Kubernetes/OpenShift.

## Technology Stack

- **Primary languages**: Go, Python
- **API framework**: gRPC with REST gateway, Protocol Buffers
- **Database**: PostgreSQL with generic DAO pattern
- **Orchestration**: Kubernetes operators (controller-runtime)
- **Provisioning**: Ansible Automation Platform (AAP), Event-Driven Ansible (EDA)
- **Auth**: Keycloak + OPA, JWT tokens, multi-tenancy via annotations
- **Testing**: Ginkgo/Gomega (Go), pytest (Python), Kind for integration tests

## Component Repositories

| Repository | Description | Key Areas |
|------------|-------------|-----------|
| [fulfillment-service](https://github.com/osac-project/fulfillment-service) | gRPC/REST API server, PostgreSQL backend | Proto definitions, servers, database, controllers |
| [osac-operator](https://github.com/osac-project/osac-operator) | Kubernetes operator for resource provisioning | CRDs, controllers, provisioning providers (AAP, EDA) |
| [osac-aap](https://github.com/osac-project/osac-aap) | Ansible roles for VM and network provisioning | Roles for compute, networking, storage |
| [osac-installer](https://github.com/osac-project/osac-installer) | Installation manifests and setup scripts | Helm charts, kustomize, version pinning |
| [osac-test-infra](https://github.com/osac-project/osac-test-infra) | Integration testing infrastructure | pytest E2E tests, CI utilities |
| [osac-ui](https://github.com/osac-project/osac-ui) | Web console for managing cloud resources | React UI |
| [enhancement-proposals](https://github.com/osac-project/enhancement-proposals) | Design documents and RFCs | Enhancement proposals (EPs) |
| [docs](https://github.com/osac-project/docs) | Architecture documentation and guides | Architecture diagrams, design docs |
| [host-management-openstack](https://github.com/osac-project/host-management-openstack) | Bare metal host management via OpenStack | Host discovery, ESI integration |
| [bare-metal-fulfillment-operator](https://github.com/osac-project/bare-metal-fulfillment-operator) | Operator for bare metal fulfillment | HostPool, Host CRDs |

## OSAC Services

| Service | Description |
|---------|-------------|
| **BMaaS** | Bare Metal as a Service — provisioning and lifecycle of physical machines |
| **CaaS** | Cluster as a Service — Kubernetes cluster provisioning via Hosted Control Planes |
| **VMaaS** | Virtual Machines as a Service — KubeVirt-based compute instances |
| **MaaS** | Model as a Service — AI model serving and inference platform |
| **Enclave** | Day 1/Day 2 operations, installation monitoring, wizard UI |

## OSAC Personas

| Persona | Role | Examples |
|---------|------|----------|
| **Cloud Provider Admin** | Works for the cloud provider. Handles tenant onboarding, sets quotas, manages global catalogs, is a super-user who can see all tenants. | Tenant onboarding, quota management, global template catalogs, resource allocation |
| **Cloud Infrastructure Admin** | Works for the cloud provider. Manages core infrastructure (network, firewall, compute, storage). Integrates control plane with local infrastructure. | Specify inventory backends, network classes, IP pools, storage tiers, DNS, integrate with Netris/VAST/ESI |
| **Tenant Admin** | Works for the tenant organization. Manages their org's config, users, IDP, quotas, and org-specific catalogs. Can only see their own organization. | Create networking objects, manage tenant resources, onboard users, control template visibility |
| **Tenant User** | Works for the tenant organization. Self-service provisions cloud resources, manages full lifecycle. Prefers click-ops but wants API/CLI for automation. | Order machines/clusters/VMs via catalog, manage instance lifecycle, view quota utilization |

## Cross-Cutting Dimensions

Every OSAC PRD must address each of these dimensions (even if just "Not applicable"):

### Tenant Onboarding
- RBAC requirements (new roles, permissions, policy changes)
- IDP integration (authentication flows, identity provider considerations)
- Auto-provisioned resources during tenant creation
- Tenant isolation implications

### Inventory
- Which inventory backend(s) does the feature use or affect?
- Does the feature add new inventory backends or extend existing ones?
- Which services consume the inventory data?

### Provisioning
- Which provisioning backend(s) are involved?
- Lifecycle stages affected (create, start, stop, restart, delete)
- Power management considerations (BMaaS)
- Cluster vs. ComputeInstance vs. bare metal provisioning differences

### Networking
- Which networking backend(s) are involved?
- Is the integration through the OSAC networking API or a side-channel?
- Does the feature add or modify networking API resources?
- NetworkClass configuration requirements (Cloud Infrastructure Admin)
- PublicIP pool management

### Storage
- Prerequisites (e.g., VAST storage accessible from hub cluster)
- StorageTier API resources
- Automated provisioning during tenant onboarding
- Per-cluster / per-tenant VAST view creation
- Disk attachment to compute instances

### Installation
- Changes to Helm charts or kustomize manifests
- CI pipeline implications
- New prerequisites or dependencies
- `osac-installer/setup.sh` updates needed

## Cross-Repo Dependencies

When a feature touches multiple repos, note all dependent repos:

| Change in | Also check | Why |
|-----------|-----------|-----|
| `fulfillment-service` proto fields | `osac-installer` CI overlays | Installer pins image versions |
| `osac-aap` collection roles | `osac-installer` submodule refs | Installer pins AAP collection versions |
| `osac-operator` CRD types | `fulfillment-service` reconciler | New CRDs must be registered in reconciler |
| `osac-operator` CRD spec changes | `osac-aap` roles | Roles must extract and use new fields |
| `fulfillment-service` CLI flag changes | `osac-test-infra` helpers | Test helpers must match CLI interface |
| `osac-installer` submodule bumps (`base/`) | `base/kustomization.yaml` image tags | Run `scripts/sync-image-tags.sh --fix` to update; CI fails if tags don't match submodule SHAs |

## Review Expectations

OSAC PRD reviewers expect:
- All template sections present, even if "N/A"
- All four personas considered where applicable
- Cross-cutting dimensions addressed explicitly
- Terminology defined and used consistently
- Functional requirements with testable acceptance criteria
- Specific non-goals ("Auto-scaling and multi-region placement are out of scope" not "Advanced features are out of scope")
- Risks with concrete failure modes, not vague categories
- Cross-repo dependencies identified

### Common Anti-Patterns to Avoid
- Implementation-focused user stories ("I want the CRD to have a field" vs "I want to define an isolated network")
- Placeholder test plans ("Tests will be added" vs specific test strategy)
- Generic risks ("Implementation might have bugs" vs "IPv6 dual-stack adds testing complexity")
- Inconsistent terminology (e.g., "Floating IP" / "PublicIP" / "External IP" interchangeably)
- Workflow gaps (jumping from creation to deletion, missing lifecycle operations)

## Milestone Scoping

When writing a PRD, explicitly declare:
- **Target milestone** (e.g., 0.1, 0.2)
- **What's NOT covered** — capabilities deferred to a later milestone
- **Known risks and gaps** — dependencies, DNS requirements, third-party onboarding
- **Upgrades** — OSAC does not currently support upgrades, so data migration and backward compatibility are not concerns. State this explicitly if applicable.

## Publishing Convention

PRDs are published to the `enhancement-proposals` repo:
- Skip the "release" question — use `enhancements` as the fixed directory prefix
- Directory: `enhancements/<feature-slug>/`
- PRD filename: `prd.md`
- Design (EP) filename: `README.md` (not `design.md` — this is the main EP file)
- Both files live in the same directory
- Fork-based workflow: push to `fork` remote, never `origin`. PRs go from `fork/<branch>` to `origin/main`.

## EP Reference Library

Existing EPs as quality benchmarks (in `enhancement-proposals/enhancements/`):

| Slug | Lines | Notable Patterns |
|------|-------|------------------|
| `networking` | 818 | Terminology section, dual-stack IPv4/IPv6, NetworkClass pluggable architecture |
| `bare-metal-fulfillment` | ~400 | ESI integration, serial console, network attachment at interface level |
| `vmaas` | ~300 | Template-based provisioning, GPU support, live migration |
| `organizations` | ~300 | Tenant isolation, RBAC patterns |
| `tenant-specific-storageclasses` | ~200 | Provider/tenant resource split pattern |
