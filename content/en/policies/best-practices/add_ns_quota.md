---
type: "docs"
title: Add Ns Quota
linkTitle: Add Ns Quota
weight: 17
description: >
    To limit the number of objects, as well as the total amount of compute that may be consumed by a single namespace, create a default resource quota for each namespace.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//best-practices/add_ns_quota.yaml" target="-blank">/best-practices/add_ns_quota.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-ns-quota
  annotations:
    policies.kyverno.io/category: Workload Isolation
    policies.kyverno.io/description: To limit the number of objects, as well as the 
      total amount of compute that may be consumed by a single namespace, create 
      a default resource quota for each namespace.
spec:
  rules:
  - name: generate-resourcequota
    match:
      resources:
        kinds:
        - Namespace
    exclude:
      resources:
        namespaces:
          - "kube-system"
          - "default"
          - "kube-public"
          - "kyverno"
    generate:
      kind: ResourceQuota
      name: default-resourcequota
      synchronize : true
      namespace: "{{request.object.metadata.name}}"
      data:
        spec:
          hard:
            requests.cpu: '4'
            requests.memory: '16Gi'
            limits.cpu: '4'
            limits.memory: '16Gi'
  - name: generate-limitrange
    match:
      resources:
        kinds:
        - Namespace
    generate:
      kind: LimitRange
      name: default-limitrange
      synchronize : true
      namespace: "{{request.object.metadata.name}}"
      data:
        spec:
          limits:
          - default:
              cpu: 500m
              memory: 1Gi
            defaultRequest:
              cpu: 200m
              memory: 256Mi
            type: Container

```