# GitLab Proof of Concept (PoC) – Production-Grade Deployment

## Executive Summary

This document outlines the successful implementation of a modular, production-grade GitLab Proof of Concept (PoC). The primary objective was to validate the architecture, network design, security model, and component integration before scaling to the full enterprise GitLab rollout. The PoC demonstrates that the platform is operationally ready, secure, and adheres to best practices for high availability and modular separation of concerns.

---
![image](https://github.com/user-attachments/assets/a5555183-d267-44f9-b3ce-d8a0fb640231)





## Project Objectives

- Deploy GitLab Core in a dedicated environment accessible via reverse proxy.
- Isolate GitLab Runner infrastructure to ensure secure CI/CD execution.
- Secure GitLab with HTTPS via NGINX, preparing for TLS certificates from a public CA.
- Validate public/private routing, port exposure, and DNS-based access models.
- Lay the foundation for a scalable and production-hardened GitLab deployment.

---

## Architecture Design

The GitLab system was deployed using a three-tier modular architecture to mirror an enterprise-ready topology. Each component was provisioned on its own virtual machine to ensure isolation, security, and scalability:

### GitLab Core

- Hosted on a dedicated Ubuntu EC2 instance.
- Installed using the official GitLab Omnibus package.
- Exposed locally on port `8181`.
- External access is routed through the NGINX reverse proxy.
- Key endpoint validated: `http://18.168.148.18:8181`.

### GitLab Runner

- Hosted on a separate Amazon Linux instance.
- Designed to handle CI/CD jobs in a controlled and secure manner.
- Docker support is enabled to allow containerized builds.
- Connectivity and registration with GitLab Core are deferred to the production phase.

### NGINX Reverse Proxy

- Deployed on Amazon Linux 2023.
- Acts as the public-facing entry point for GitLab.
- Routes all traffic from port `80/443` to the internal GitLab Core on port `8181`.
- SSL placeholders configured for public CA integration (Let's Encrypt via Certbot).
- Validated HTTP access via `curl`, browser, and internal loopback.

---

## Implementation Summary

### GitLab Core Setup

- Installed via package manager (`curl` and `.deb` source).
- Configured `external_url` to match internal access address.
- Verified service via `curl`, systemd, and browser redirect tests.

### NGINX Proxy Configuration

- Created custom `gitlab.conf` in `/etc/nginx/conf.d/` with both HTTP and HTTPS server blocks.
- Verified NGINX configuration using `nginx -t` and `systemctl status`.
- Routes requests to the GitLab backend while preserving key headers (`Host`, `X-Forwarded-*`).
- Prepped SSL paths for `gitlab.crt` and `gitlab.key`.

### DNS and IP Configuration

- Verified all public/private IPs.
- Ensured connectivity between instances using `curl`, `ss`, and port checks.
- Prepared DNS routing plan for future domains like `gitlab.greenlnk.net`.

---

## TLS Integration Plan (Next Step)

To finalize the production readiness, we will acquire a publicly trusted TLS certificate using Let’s Encrypt and integrate it with the NGINX proxy. The steps are:

1. Install `certbot` on the NGINX server.
2. Obtain a certificate for a real domain name (e.g., `gitlab.greenlnk.net`).
3. Replace existing SSL placeholder paths with Certbot-managed certificates.
4. Reload NGINX and test HTTPS access.

---

## Validation & Testing

- Verified NGINX proxy routing and GitLab UI response.
- Conducted TCP and HTTP checks across all relevant ports.
- Reviewed log output from systemd services and browser network tools.
- Confirmed public-facing access is functional and secure for the PoC stage.

---

## Conclusion

This PoC confirms that the modular GitLab architecture is:

- Correctly designed and deployed according to enterprise standards.
- Securely proxied using NGINX with a clear path to HTTPS.
- Ready for integration with CI/CD pipelines and user authentication.
- Scalable for production deployment following Certbot TLS setup and GitLab Runner integration.

This PoC now serves as a validated reference for the main GitLab project rollout planned for the upcoming week.
