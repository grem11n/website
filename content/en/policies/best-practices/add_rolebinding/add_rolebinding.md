---
title: "Add RoleBinding"
category: Multi-Tenancy
version: 1.6.0
subject: RoleBinding
policyType: "generate"
description: >
    Typically in multi-tenancy and other use cases, when a new Namespace is created, users and other principals must be given some permissions to create and interact with resources in the Namespace. Very commonly, Roles and RoleBindings are used to grant permissions at the Namespace level. This policy generates a RoleBinding called `<userName>-admin-binding` in the new Namespace which binds to the ClusterRole `admin` as long as a `cluster-admin` did not create the Namespace. Additionally, an annotation named `kyverno.io/user` is added to the RoleBinding recording the name of the user responsible for the Namespace's creation.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//best-practices/add_rolebinding/add_rolebinding.yaml" target="-blank">/best-practices/add_rolebinding/add_rolebinding.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-rolebinding
  annotations:
    policies.kyverno.io/title: Add RoleBinding
    policies.kyverno.io/category: Multi-Tenancy
    policies.kyverno.io/subject: RoleBinding
    policies.kyverno.io/minversion: 1.6.0
    policies.kyverno.io/description: >-
      Typically in multi-tenancy and other use cases, when a new Namespace is created,
      users and other principals must be given some permissions to create and interact
      with resources in the Namespace. Very commonly, Roles and RoleBindings are used to
      grant permissions at the Namespace level. This policy generates a RoleBinding
      called `<userName>-admin-binding` in the new Namespace which binds to the ClusterRole
      `admin` as long as a `cluster-admin` did not create the Namespace. Additionally, an annotation
      named `kyverno.io/user` is added to the RoleBinding recording the name of the user responsible
      for the Namespace's creation.
spec:
  background: false
  rules:
    - name: generate-admin-binding
      match:
        any:
        - resources:
            kinds:
              - Namespace
      exclude:
        any:
        - clusterRoles:
          - cluster-admin
      generate:
        synchronize: true
        kind: RoleBinding
        name: "{{request.userInfo.username}}-admin-binding"
        namespace: "{{request.object.metadata.name}}"
        data:
          metadata:
            annotations:
              kyverno.io/user: "{{request.userInfo.username}}"
          roleRef:
            apiGroup: rbac.authorization.k8s.io
            kind: ClusterRole
            name: admin
          subjects:
            - kind: User
              name: "{{request.userInfo.username}}"
```
