# NGINX Configuration for HashiCorp Vault

This document explains how to set up NGINX as a reverse proxy for HashiCorp Vault, ensuring proper routing and resource loading for the Vault UI and API.

## Overview

The NGINX configuration handles the following:
1. Proxying requests to Vault running on `http://localhost:8200`.
2. Exposing Vault's UI and API under a `/vault/` prefix.
3. Ensuring correct paths for static resources like CSS and JS.

## Prerequisites

- HashiCorp Vault running on `http://localhost:8200`.
- NGINX installed and configured to listen on port 8088.

## Configuration

Below is the NGINX configuration file:

```nginx
server {
    listen 8088;
    server_name localhost;

    location /vault/ {
        rewrite ^/vault(/.*)$ $1 break;  # Rewrites /vault/ path to root path when forwarding
        proxy_pass http://localhost:8200/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /ui/ {
        proxy_pass http://localhost:8200/ui/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /v1/ {
        proxy_pass http://localhost:8200/v1/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Key Points

1. **`/vault/` Prefix**:
   - Requests to `http://localhost:8088/vault/` are rewritten to remove the `/vault/` prefix before being forwarded to Vault.
   - This ensures compatibility with Vault's routing logic.

2. **UI and API Paths**:
   - Specific location blocks for `/ui/` and `/v1/` ensure that Vault's static resources and API paths are properly proxied.
   - This avoids issues with missing CSS or JS files.

## Testing

1. Start Vault on `http://localhost:8200`.
2. Restart NGINX:
   ```bash
   sudo systemctl restart nginx
   ```
   or
   ```bash
   sudo nginx -s reload
   ```
3. Access Vault at:
   - **Vault UI**: `http://localhost:8088/vault/ui`
   - **Vault API**: `http://localhost:8088/vault/v1` (e.g., `http://localhost:8088/vault/v1/sys/health`)

## Troubleshooting

1. **Blank Page in UI**:
   - Ensure the `/ui/` location block correctly forwards requests to `http://localhost:8200/ui/`.
2. **Missing Resources**:
   - Verify that static resources (like JS or CSS) are loaded from the correct paths under `/vault/ui/`.
3. **Connection Issues**:
   - Check that Vault is running and accessible on `http://localhost:8200`.
   - Verify NGINX logs for errors:
     ```bash
     sudo tail -f /var/log/nginx/error.log
     ```

## Conclusion

This configuration allows NGINX to act as a reverse proxy for HashiCorp Vault, ensuring proper routing and resource loading for the UI and API. Adjust the configuration as needed for your specific environment.

