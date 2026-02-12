---
date: "2026-02-09"
org: sreniatnoc
repo: rusd
goal: Validate rusd as etcd replacement in Kind K8s cluster
type: session
model: claude-opus-4-6
---

# Session: rusd Kind Cluster Validation

**Date**: 2026-02-09 to 2026-02-10
**Org**: sreniatnoc
**Repo**: rusd
**Goal**: Implement Watch event delivery, fix txn CAS, validate rusd as etcd replacement in Kind K8s cluster

## Summary

Implemented Watch event delivery, txn CAS operations, prev_kv in Watch events, graceful shutdown with serve_with_shutdown(), and HTTP/2 keepalive. Successfully validated rusd as an external etcd backend for a Kind cluster running Kubernetes v1.35. All K8s CRUD operations work including deployments, scaling, rolling updates, pod exec, and namespace lifecycle.

## Phase 1: Initial Validation Attempt

Attempted full validation of rusd for Kubernetes compatibility. Native build and unit tests pass. etcdctl compatibility partial. Kind cluster creation failed due to unimplemented Watch event delivery.

### What Worked
- cargo build --release succeeds (44 warnings, 0 errors)
- Docker build succeeds after Dockerfile fixes (167MB image)
- 40/40 unit tests PASS
- etcdctl KV (put/get/del), Lease (grant/list/timetolive/revoke), Endpoint health

### What Failed
- Watch API: acknowledged but never delivered events
- Kind cluster: kube-apiserver 403 Forbidden (RBAC can't bootstrap without Watch)

## Phase 2: Bugs Found and Fixed

1. **next_revision() returning old value** - returning pre-increment instead of post-increment
2. **Proto field numbers wrong** - events field was 7 but etcd expects 11, prev_kv missing
3. **Initial revision 0** - etcd starts at 1, K8s expects >= 1
4. **Transport error on shutdown** - switched to serve_with_shutdown() for graceful draining
5. **prev_kv missing in Watch events** - K8s informers need prev_kv for change detection
6. **Txn compares always succeeding** - implemented actual comparison logic for CAS

## Final State

Full K8s CRUD validated: namespace, configmap, secret, serviceaccount, role, rolebinding, deployment create/scale/rolling-update, service, pod exec. Kind cluster with K8s v1.35 bootstraps and runs all control plane components with rusd as external etcd.
