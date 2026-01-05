# CAPI Ingress CRD

## Overview

`CAPIIngress` is a Kubernetes Custom Resource used to declaratively register services into one or more Consul environments.

It **does not route traffic** and **does not act as an ingress controller**.  
Its sole responsibility is to express *intent*, which is then reconciled by the **CAPI Agent** into Consul service registrations.

---

## API Version

```yaml
apiVersion: capi.io/v1
kind: CAPIIngress
```

---

## Scope

- Namespaced
- One `CAPIIngress` represents one logical service
- Can declare multiple Consul registrations

---

## Example

```yaml
apiVersion: capi.io/v1
kind: CAPIIngress
metadata:
  name: hello-world
  namespace: dev
spec:
  serviceName: hello-world
  serviceGroup: dev
  instances:
    - capiNamespace: public
      service: hello-world
      port: 8080
      path: /hello
      secured: false
      openApi: http://hello-world:8080/openapi

    - capiNamespace: private
      service: hello-world
      port: 8080
      path: /internal
      secured: true
      openApi: http://hello-world:8080/internal/openapi
```

---

## Spec Fields

### `spec.serviceName` (required)

Logical name of the service registered in Consul.

---

### `spec.serviceGroup` (required)

Logical grouping (environment, domain).  
Used to ensure uniqueness both in Consul and CAPI.

---

### `spec.instances` (required)

List of registration intents.

Each instance maps to one CAPI namespace. 2 instances can point to the same Consul.

---

## Instance Fields

| Field | Required | Description | Default |
|------|----------|-------------| ---------|
| `capiNamespace` | yes | CAPI instance target |
| `service` | yes | Kubernetes Service name |
| `port` | yes | Service port |
| `path` | yes | HTTP path prefix |
| `secured` | no | Security flag | false
| `openApi` | no | OpenAPI URL | null
| `routeGroupFirst` | no | group will come before in the context | false
| `subscriptionGroup` | no | If secured, CAPI will search for this group in the token subscriptions claim | null
| `rootContext` | no | If not null, CAPI will forward all requests to this path | null
| `scheme`| no | Service listening scheme | HTTP
| `version`| no | Free version field, good to trigger a new CAPI route update | null
| `ingress`| no | If not null, CAPI will forward all requests to this ingress | null
| `keepGroup` | no | If not null CAPI will preserve the `group` in a dedicated header | null
| `opaRego` | no | If not null CAPI will call this opa rego to authorize requests. | null
| `type` | no | Type of service. Enum (rest, websocker, sse) | rest

---

## Consul Registration Rules

- Exactly **one Consul service** per:
  `(serviceName + serviceGroup + consulHost)`
- Multiple CAPI instances targeting the same Consul are flattened into metadata

### Example Metadata

```yaml
capi-instance-public-open-api: {...}
capi-instance-private-open-api: {...}
```

---

## Lifecycle Mapping

| Kubernetes Event | Consul Action |
|------------------|---------------|
| ADDED | Register service |
| MODIFIED | Update metadata |
| DELETED | Deregister service |

---

## Responsibility Split

### Developers
- Declare `CAPIIngress`
- Express exposure intent

### Operations
- Deploy CAPI Agent
- Configure Consul environments
- Control allowed `capiNamespace` values

---

## Non-goals

CAPIIngress does **not**:
- route traffic
- terminate TLS
- replace Ingress controllers

---

## Design Philosophy

Declarative intent → deterministic reconciliation → external system consistency.
