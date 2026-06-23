# OSAC Project Context (Design Phase)

**Last Updated:** 2026-06-23

## Overview

OSAC (Open Sovereign AI Cloud) is a fulfillment system for provisioning Kubernetes clusters, virtual machines, and bare metal instances with networking capabilities. It is a multi-tenant cloud platform built on Kubernetes/OpenShift.

## Component Repositories

| Repository | Description | Key Areas |
|------------|-------------|-----------|
| [fulfillment-service](https://github.com/osac-project/fulfillment-service) | gRPC/REST API server, PostgreSQL backend | Proto definitions, servers, database, controllers |
| [osac-operator](https://github.com/osac-project/osac-operator) | Kubernetes operator for resource provisioning | CRDs, controllers, provisioning providers |
| [osac-aap](https://github.com/osac-project/osac-aap) | Ansible roles for VM and network provisioning | Roles for compute, networking, storage |
| [osac-installer](https://github.com/osac-project/osac-installer) | Installation manifests and setup scripts | Helm charts, kustomize, version pinning |
| [osac-test-infra](https://github.com/osac-project/osac-test-infra) | Integration testing infrastructure | pytest E2E tests |
| [osac-ui](https://github.com/osac-project/osac-ui) | Web console | React UI |
| [enhancement-proposals](https://github.com/osac-project/enhancement-proposals) | Design documents and RFCs | Enhancement proposals |
| [docs](https://github.com/osac-project/docs) | Architecture docs | Diagrams, design docs |
| [host-management-openstack](https://github.com/osac-project/host-management-openstack) | Bare metal host management | Host discovery, ESI |
| [bare-metal-fulfillment-operator](https://github.com/osac-project/bare-metal-fulfillment-operator) | Bare metal fulfillment operator | HostPool, Host CRDs |

## Architecture

### Layers

- **API Layer (Proto-based)**: Resource schemas and service contracts in `fulfillment-service/proto/`. Split into public (user-facing, read-heavy) and private (admin, full CRUD + Signal RPC).
- **Server Layer (gRPC)**: Resource-specific servers in `fulfillment-service/internal/servers/`. Public servers wrap private servers and add tenant/auth logic. Builder pattern for configuration.
- **Database Layer**: PostgreSQL with generic DAO in `fulfillment-service/internal/database/dao/`. CEL filtering, type-safe CRUD via Go generics.
- **Auth Layer**: JWT + OPA in `fulfillment-service/internal/auth/`. Attribution (creator tracking), tenancy (tenant identification), multi-tenancy enforcement.
- **Controller Layer**: Kubernetes controllers in `osac-operator/internal/controller/` and `fulfillment-service/internal/controllers/`. Reconciliation pattern: finalizer → status update → provisioning lifecycle.
- **Provisioning Providers**: Pluggable backends in `osac-operator/internal/provisioning/`. AAP provider, EDA webhook provider.

### Data Flow

1. Client → Public Server → Private Server → Database (create resource)
2. Controller polls fulfillment service → checks spec vs status → triggers provisioning provider
3. Provider completes → feedback controller → Signal RPC → database update
4. Desired state in resource Spec; observed state in resource Status; database is source of truth

### Key Abstractions

| Abstraction | Purpose | Location |
|-------------|---------|----------|
| Generic DAO[O] | Type-safe database CRUD for any protobuf message | `fulfillment-service/internal/database/dao/generic_dao.go` |
| Generic Server[O] | CRUD gRPC operations for any resource | `fulfillment-service/internal/servers/generic_server.go` |
| Server Builder | Builder pattern for server configuration | `NewClustersServer().SetLogger(...).Build()` |
| Reconciler[O] | Polls/watches resource changes, executes reconciliation | `fulfillment-service/internal/controllers/reconciler.go` |
| Provisioning Provider | Interface for provision/deprovision backends | `osac-operator/internal/provisioning/provider.go` |

## Proto Conventions

- **Files**: `snake_case.proto` (e.g., `virtual_network_type.proto`)
- **Messages**: `PascalCase` (e.g., `VirtualNetwork`, `Cluster`)
- **Fields**: `snake_case` (e.g., `creation_timestamp`, `ipv4_cidr`)
- **Enums**: `SCREAMING_SNAKE_CASE` (e.g., `STATE_PENDING`, `STATE_READY`)
- **Services**: `PascalCase` (e.g., `Clusters`, `VirtualNetworks`)
- **RPC Methods**: `PascalCase` (e.g., `CreateCluster`, `GetVirtualNetwork`)
- **Public vs Private**: Public API in `proto/public/` (read-heavy, limited writes), private in `proto/private/` (full CRUD + Signal RPC for controller feedback)

## Controller Patterns

All osac-operator controllers follow the same pattern:
1. **Finalizer management**: Add finalizer on creation, remove on deletion
2. **Status update**: Set conditions based on current state
3. **Provisioning lifecycle**: Use `provisioning.RunProvisioningLifecycle()` for provision/deprovision
4. **Management-state annotation**: Check `osac.openshift.io/management-state`, skip reconciliation when `Unmanaged`
5. **Multi-cluster support**: Use `multicluster-runtime` for management/workload cluster separation
6. **Namespace isolation**: Networking controllers filter to configured namespace via `NetworkingNamespacePredicate`

When adding features, check ALL controllers that follow the same pattern.

## Multi-Tenancy

- **Tenant ID**: `osac.openshift.io/tenant` annotation on every resource
- **Owner reference**: `osac.openshift.io/owner-reference` annotation
- **Enforcement**: OPA policies at authorization layer; all database queries filtered by tenant
- **Tenancy modes**: guest (no enforcement), default (annotation-based), serviceaccount (SA-based)

## Where to Add New Code

For a new resource type:
1. Proto definitions in `fulfillment-service/proto/{public,private}/osac/*/v1/`
2. Server implementation in `fulfillment-service/internal/servers/`
3. Database migration in `fulfillment-service/internal/database/migrations/`
4. CRD definition in `osac-operator/api/v1alpha1/` (if operator-managed)
5. Controller in `osac-operator/internal/controller/`
6. Tests: unit tests alongside code, integration tests in `fulfillment-service/it/`, E2E in `osac-test-infra`

## Cross-Repo Dependencies

| Change in | Also check | Why |
|-----------|-----------|-----|
| `fulfillment-service` proto fields | `osac-installer` CI overlays | Installer pins image versions |
| `osac-aap` collection roles | `osac-installer` submodule refs | Installer pins AAP collection versions |
| `osac-operator` CRD types | `fulfillment-service` reconciler | New CRDs must be registered |
| `osac-operator` CRD spec changes | `osac-aap` roles | Roles must extract and use new fields |
| `fulfillment-service` CLI flag changes | `osac-test-infra` helpers | Test helpers must match CLI interface |
| `osac-installer` submodule bumps | `base/kustomization.yaml` image tags | Run `scripts/sync-image-tags.sh --fix` |

## Test Patterns

- **Unit tests (Go)**: Ginkgo/Gomega in `*_test.go` files; mock with `go.uber.org/mock`
- **Integration tests (Go)**: Kind cluster in `fulfillment-service/it/`; `IT_KEEP_KIND=true` for debugging
- **E2E tests (Python)**: pytest in `osac-test-infra`; test against running cluster
- **Proto linting**: `buf lint` before committing proto changes; `buf generate` for codegen

## OSAC Services

| Service | Description |
|---------|-------------|
| **BMaaS** | Bare Metal as a Service — physical machine provisioning and lifecycle |
| **CaaS** | Cluster as a Service — Kubernetes cluster provisioning via Hosted Control Planes |
| **VMaaS** | Virtual Machines as a Service — KubeVirt-based compute instances |
| **MaaS** | Model as a Service — AI model serving and inference |
| **Enclave** | Day 1/Day 2 operations, installation, wizard UI |

## OSAC Personas

| Persona | Role |
|---------|------|
| **Cloud Provider Admin** | Tenant onboarding, quotas, global catalogs, super-user |
| **Cloud Infrastructure Admin** | Core infrastructure, network/firewall/compute/storage |
| **Tenant Admin** | Org config, users, IDP, quotas, org-specific catalogs |
| **Tenant User** | Self-service resource provisioning, lifecycle management |

## EP Reference Library

Existing EPs as quality benchmarks:

| Slug | Lines | Notable Patterns |
|------|-------|------------------|
| `networking` | 818 | Terminology section, dual-stack IPv4/IPv6, NetworkClass pluggable architecture |
| `bare-metal-fulfillment` | ~400 | ESI integration, serial console, network attachment at interface level |
| `vmaas` | ~300 | Template-based provisioning, GPU support, live migration |
| `organizations` | ~300 | Tenant isolation, RBAC patterns |
| `tenant-specific-storageclasses` | ~200 | Provider/tenant resource split pattern |
| `computeinstance-phase-condition-expansion` | ~200 | API evolution pattern for existing resources |
