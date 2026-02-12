---
date: "2026-02-11"
org: sreniatnoc
repo: rusd
goal: Build comprehensive K8s validation test suite
type: session
model: claude-opus-4-6
cost_note: "shared session with multi-node-raft; see that memory for cost data"
---

# Session: K8s Test Suite

**Date**: 2026-02-11
**Org**: sreniatnoc
**Repo**: rusd
**Goal**: Automated K8s validation with 34 test points in CI

## Summary

Built a comprehensive Kubernetes validation script that runs rusd as an external etcd backend for a Kind cluster. The script tests 34 operations covering namespace lifecycle, RBAC, deployments, scaling, rolling updates, services, and pod exec. Runs in CI on every push.

## Test Coverage

34 test points organized into categories:
- **Namespace**: create, verify, delete (3 tests)
- **RBAC**: role, rolebinding, serviceaccount CRUD (6 tests)
- **ConfigMap/Secret**: create, read, update, delete (8 tests)
- **Deployment**: create, verify pods, scale up/down (6 tests)
- **Rolling Update**: image update, rollout status, verify (4 tests)
- **Service**: expose, verify endpoints (3 tests)
- **Pod Operations**: exec, logs, port-forward (4 tests)

## Key Gotchas

- Kind host.docker.internal works from Docker containers on macOS Docker Desktop
- etcdctl --count-only is broken in simple/JSON mode, use --keys-only with grep -c
- CI multi-node tests need wait_for_leader, not just endpoint health
- grep -c pattern || echo "0" causes double output, use || true instead
