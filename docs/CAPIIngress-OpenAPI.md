# CAPI Ingress – OpenAPI Specification

## Resource

**Kind:** `CAPIIngress`  
**Group:** `capi.io`  
**Version:** `v1`  
**Scope:** Namespaced

---

## Schema

### CAPIIngress

```yaml
CAPIIngress:
  type: object
  required:
    - spec
  properties:
    metadata:
      type: object
    spec:
      $ref: '#/components/schemas/CAPIIngressSpec'
```

---

### CAPIIngressSpec

```yaml
CAPIIngressSpec:
  type: object
  required:
    - serviceName
    - serviceGroup
    - instances
  properties:
    serviceName:
      type: string
    serviceGroup:
      type: string
    instances:
      type: array
      items:
        $ref: '#/components/schemas/CAPIInstance'
```

---

### CAPIInstance

```yaml
CAPIInstance:
  type: object
  required:
    - capiNamespace
    - service
    - port
    - path
    - secured
  properties:
    capiNamespace:
      type: string
      description: Logical Consul environment
    service:
      type: string
      description: Kubernetes Service name
    port:
      type: integer
      format: int32
    path:
      type: string
    secured:
      type: boolean
    openApi:
      type: string
      format: uri
```

---

## Validation Rules

- `capiNamespace` must be pre-configured by operations
- Duplicate `(capiNamespace + secured)` combinations are forbidden
- At least one instance must be declared

---

## Behavioral Contract

- Controller reacts to ADD / MODIFY / DELETE events
- State is reconciled idempotently
- Failures do not block reconciliation of other resources

---

## Compatibility

- No backward compatibility guaranteed before v1
- Fields may be extended in future versions

---

## Status

CAPIIngress does not currently expose a `.status` subresource.

---

## Security Considerations

- No secrets stored in CRD
- Credentials managed by the CAPI Agent via ConfigMaps

---

## Notes

This specification describes intent only.  
Execution is performed by the CAPI Agent.
