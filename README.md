# CloudflareDNSRecord Crossplane Template

Real DNS record management for Cloudflare using External-DNS.

## Overview

This template provides real DNS record creation in Cloudflare via External-DNS, enabling namespaced DNS management without requiring cluster-scoped resources.

## Migration from Provider-Based Approach

**Previous:** Used Crossplane's provider-cloudflare which required cluster-scoped resources
**Current:** Uses External-DNS with DNSEndpoint CRDs for namespace-isolated DNS management

## Prerequisites

1. **External-DNS** installed and configured
2. **Cloudflare API Token** configured in External-DNS
3. **dns-config** EnvironmentConfig (provides zone name)

## Installation

### 1. Ensure External-DNS is Running

```bash
# Check External-DNS status
kubectl get deployment -n external-dns external-dns

# Verify DNSEndpoint CRD exists
kubectl get crd dnsendpoints.externaldns.openportal.dev
```

### 2. Configure DNS Zone

```bash
# Edit dns-config EnvironmentConfig with your zone
kubectl edit environmentconfig dns-config

# Set the zone field (e.g., "openportal.dev")
```

### 3. Install Template

```bash
kubectl apply -k .
```

## Usage

### Create A Record

```yaml
apiVersion: openportal.dev/v1alpha1
kind: CloudflareDNSRecord
metadata:
  name: myapp-dns
  namespace: my-team
spec:
  type: A
  name: myapp
  value: "50.56.157.82"
  proxied: true
  comment: "My application DNS"
```

### Create CNAME

```yaml
apiVersion: openportal.dev/v1alpha1
kind: CloudflareDNSRecord
metadata:
  name: www-dns
  namespace: my-team
spec:
  type: CNAME
  name: www
  value: "openportal.dev"
```

### Create MX Record

```yaml
apiVersion: openportal.dev/v1alpha1
kind: CloudflareDNSRecord
metadata:
  name: mail-dns
  namespace: my-team
spec:
  type: MX
  name: "@"
  value: "mail.openportal.dev"
  priority: 10
```

## How It Works

1. **You create** a CloudflareDNSRecord XR in your namespace
2. **Crossplane Composition** transforms it into a DNSEndpoint resource
3. **External-DNS** watches DNSEndpoint resources and syncs them to Cloudflare
4. **Cloudflare** creates/updates the actual DNS records

## Configuration

The template requires:
- **dns-config** EnvironmentConfig with `zone` field
- **External-DNS** with Cloudflare credentials configured

## Features

- ✅ Real DNS record creation in Cloudflare
- ✅ Support for A, AAAA, CNAME, TXT, MX records
- ✅ Cloudflare proxy (orange cloud) support
- ✅ TTL management
- ✅ Comment support for record documentation
- ✅ **Namespace isolation** - teams manage their own DNS records
- ✅ No cluster-scoped resources required

## Restaurant Analogy

- **Menu (XRD)**: CloudflareDNSRecord - what DNS records can be ordered
- **Recipe (Composition)**: How to create DNSEndpoint resources
- **Waiter (External-DNS)**: Takes DNSEndpoint orders to Cloudflare
- **Order (XR)**: Your DNS record request
- **Kitchen (Cloudflare)**: Where the actual DNS records are created

## Architecture Benefits

### Previous (Provider-Based)
- Required cluster-scoped Cloudflare provider
- Namespaced XRs couldn't create cluster-scoped resources
- Limited multi-tenancy support

### Current (External-DNS)
- Namespaced DNSEndpoint resources
- External-DNS handles provider interaction
- Full namespace isolation for teams
- Better security and multi-tenancy

## Troubleshooting

### DNS Record Not Created

1. Check XR status:
```bash
kubectl get cloudflarednsrecord -n your-namespace
kubectl describe cloudflarednsrecord <name> -n your-namespace
```

2. Check DNSEndpoint:
```bash
kubectl get dnsendpoint -n your-namespace
kubectl describe dnsendpoint <name> -n your-namespace
```

3. Check External-DNS logs:
```bash
kubectl logs -n external-dns deployment/external-dns
```

### External-DNS Not Processing Records

1. Verify External-DNS is running:
```bash
kubectl get pods -n external-dns
```

2. Check External-DNS configuration:
```bash
kubectl get deployment -n external-dns external-dns -o yaml | grep args -A 10
```

3. Ensure it's watching the correct CRD:
- Should include: `--crd-source-apiversion=externaldns.openportal.dev/v1alpha1`

## TODO

- [ ] Support for SRV records
- [ ] Integration with WhoAmIApp for automatic DNS
- [ ] Support for wildcard records
- [ ] DNS record validation webhook