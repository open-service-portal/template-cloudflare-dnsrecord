# CloudflareDNSRecord Crossplane Template

Real DNS record management for Cloudflare using Crossplane.

## Overview

This template provides real DNS record creation in Cloudflare, moving beyond the mock ConfigMap approach to actual DNS management.

## Prerequisites

1. **provider-cloudflare** installed
2. **Cloudflare API Token** with Zone:Edit permissions
3. **Zone ID** from Cloudflare dashboard
4. **dns-config** EnvironmentConfig (provides zone name)

## Installation

### 1. Install Provider

```bash
kubectl apply -f scripts/cluster-manifests/crossplane-provider-cloudflare.yaml
```

### 2. Configure Credentials

```bash
# Get your API token from Cloudflare dashboard
# Update the secret with your token
kubectl edit secret cloudflare-credentials -n crossplane-system
```

### 3. Install Template

```bash
kubectl apply -k .
```

## Usage

### Create A Record

```yaml
apiVersion: platform.io/v1alpha1
kind: CloudflareDNSRecord
metadata:
  name: myapp-dns
spec:
  type: A
  name: myapp
  value: "50.56.157.82"
  proxied: true
```

### Create CNAME

```yaml
apiVersion: platform.io/v1alpha1
kind: CloudflareDNSRecord
metadata:
  name: www-dns
spec:
  type: CNAME
  name: www
  value: "openportal.dev"
```

## Configuration

The template requires:
- **dns-config** EnvironmentConfig with `zone` field
- **cloudflare-credentials** Secret with API token
- **cloudflare_zone_id** in the composition (currently hardcoded, TODO: make dynamic)

## Features

- ✅ Real DNS record creation in Cloudflare
- ✅ Support for A, AAAA, CNAME, TXT, MX records
- ✅ Cloudflare proxy (orange cloud) support
- ✅ TTL management
- ✅ Comment support for record documentation

## Restaurant Analogy

- **Menu (XRD)**: CloudflareDNSRecord - what DNS records can be ordered
- **Recipe (Composition)**: How to create records using Cloudflare API
- **Supplier (Provider)**: provider-cloudflare - connects to Cloudflare service
- **Order (XR)**: Your DNS record request
- **Kitchen (Cloudflare)**: Where the actual DNS records are created

## TODO

- [ ] Dynamic zone ID lookup from Cloudflare
- [ ] Support for MX priority
- [ ] Support for SRV records
- [ ] Integration with WhoAmIApp for automatic DNS