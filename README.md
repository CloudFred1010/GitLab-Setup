# GitLab Modular Architecture

## Overview

This repository documents the architecture and implementation of a modular GitLab Proof of Concept (PoC) intended to validate design decisions, networking, security, and deployment strategies before enterprise-wide rollout.

The PoC demonstrates how GitLab can be deployed in a production-aligned configuration using isolated components, reverse proxy access, and infrastructure-as-code readiness.

---
![image](https://github.com/user-attachments/assets/e5b90a11-6738-4ad5-9451-0befb1bf1f85)



## Objectives

- Deploy GitLab Core in a controlled, private environment.
- Isolate GitLab Runner on a separate node to decouple job execution from the GitLab application layer.
- Secure external access via NGINX acting as a reverse proxy.
- Prepare for TLS certificate integration using a public Certificate Authority (CA).
- Validate the entire deployment pipeline for future scale-out.

---

## Architecture

The architecture follows a clean separation-of-concerns design:

### 1. GitLab Core (Private Node)

- Installed using the official GitLab Omnibus package.
- Runs on a dedicated host with restricted access.
- Exposes the GitLab UI and API locally on a non-standard port.
- Only accessible externally via the proxy.

### 2. GitLab Runner (Worker Node)

- Configured as a separate compute instance.
- Prepares for secure CI/CD workloads in Docker or shell executor modes.
- Will be registered with GitLab Core in the next deployment phase.

### 3. NGINX Reverse Proxy (Public Node)

- Hosts public entry points for HTTP/HTTPS.
- Forwards requests to the GitLab Core over internal networking.
- Configured with SSL placeholders in preparation for Let’s Encrypt integration.
- Protects backend infrastructure by exposing only controlled endpoints.

---

## Implementation Details

### GitLab Core

- Installed via package manager with required dependencies.
- Verified application health using CLI tools, internal curl checks, and systemd.
- Configured `external_url` pointing to the reverse proxy address.

### NGINX Proxy

- Config includes both HTTP (port 80) and HTTPS (port 443) blocks.
- Default HTTPS paths reference placeholder certificate files:
  - `/etc/nginx/ssl/gitlab.crt`
  - `/etc/nginx/ssl/gitlab.key`
- Verified NGINX health using `nginx -t`, `systemctl`, and live curl tests.

---

## TLS Integration Plan

To secure GitLab with HTTPS, the next stage is to issue a TLS certificate from Let’s Encrypt:

1. Install `certbot` on the NGINX proxy server.
2. Point DNS (e.g., `gitlab.example.com`) to the public proxy IP.
3. Generate certificates using Certbot (webroot or standalone).
4. Update NGINX SSL paths to reference Certbot-generated keys.
5. Reload NGINX and verify HTTPS access.

---

## Verification Performed

- End-to-end routing from browser to GitLab UI confirmed.
- Port bindings (`ss -tuln`) validated across all nodes.
- NGINX reverse proxy tested using both `curl` and browser clients.
- Internal network connectivity tested between GitLab Core and the proxy.

---

## Conclusion

This PoC validates that the proposed modular GitLab deployment strategy is:

- Secure by default with strict public/private boundary controls.
- Operationally sound with clear role separation between nodes.
- Ready for further enhancements including TLS, GitLab Runner integration, and CI/CD pipeline onboarding.

The design and deployment approach proven in this PoC will now serve as the foundation for the full-scale production implementation.
