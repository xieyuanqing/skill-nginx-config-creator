---
name: nginx-config-creator
description: "Creates a standard Nginx/OpenResty reverse proxy config file for a service and reloads the web server. Features safety checks and environment awareness. Takes service name, domain, and port as main arguments."
metadata:
  openclaw:
    requires:
      bins: ["bash", "docker"]
---

# Nginx Config Creator (Enterprise Grade)

This skill automates the creation of Nginx/OpenResty reverse proxy configurations. It is designed for both ease of use and safety, incorporating environment awareness and a critical safety-check mechanism.

## Features

- **Environment Awareness**: Simplifies commands by reading configuration from environment variables.
- **Safety Check**: Includes a '熔断' (fuse) mechanism. It tests the configuration before applying it and automatically rolls back if the test fails, preventing web server downtime.

## Pre-requisites (Recommended)

For maximum convenience, it is recommended to set the following environment variables on the host system:

- `NGINX_CONFIG_PATH`: The absolute path to the Nginx `conf.d` directory.
- `NGINX_CONTAINER_NAME`: The name of the running Nginx/OpenResty Docker container.

If these are not set, they **must** be provided as command-line arguments.

## Core Action: `scripts/create-and-reload.sh`

This script performs the entire operation.

### **Inputs (Command-Line Arguments)**

- `--service-name`: (Required) The short name for the service (e.g., `n8n`).
- `--domain`: (Required) The root domain name (e.g., `994431.xyz`).
- `--port`: (Required) The local port the service is running on (e.g., `5678`).
- `--config-path`: (Optional) The path to Nginx's `conf.d` directory. **Overrides** the `NGINX_CONFIG_PATH` environment variable.
- `--container-name`: (Optional) The name of the Nginx Docker container. **Overrides** the `NGINX_CONTAINER_NAME` environment variable.

### **Output**

- **On Success**: Prints a step-by-step log of its actions and a final success message.
- **On Failure**: Prints a descriptive error message to stderr and exits. If the failure occurs during the Nginx configuration test, the full error from `nginx -t` is displayed.

### **Execution Workflow**

1.  **Parse Arguments & Environment**: The script gathers all necessary paths and names from command-line arguments and environment variables.
2.  **Generate Config**: It creates the `.conf` file in the target directory.
3.  **Test Config (Safety Check)**: It executes `nginx -t` inside the specified container.
4.  **Decide & Act**:
    - If the test passes, it proceeds to reload Nginx via `nginx -s reload`.
    - If the test fails, it **automatically deletes the generated file (rolls back)** and reports the error.
5.  **Report Result**: Informs the user of the final outcome.

### **Example Usage**

**Scenario 1: Environment variables are pre-set**

```bash
# Set for future convenience
export NGINX_CONFIG_PATH="/opt/1panel/apps/openresty/openresty/conf/conf.d"
export NGINX_CONTAINER_NAME="1Panel-openresty-1Mjq"

# Now, the command is very simple:
bash skills/nginx-config-creator/scripts/create-and-reload.sh \
  --service-name "n8n" \
  --domain "994431.xyz" \
  --port "5678"
```

**Scenario 2: No environment variables (providing all info via arguments)**

```bash
bash skills/nginx-config-creator/scripts/create-and-reload.sh \
  --service-name "n8n" \
  --domain "994431.xyz" \
  --port "5678" \
  --config-path "/opt/1panel/apps/openresty/openresty/conf/conf.d" \
  --container-name "1Panel-openresty-1Mjq"
```

### **Failure Strategy**

- **Missing Arguments**: The script will exit with an error if required arguments/environment variables are missing.
- **`nginx -t` Fails**: The skill is designed to be safe. It will **not** attempt to reload a broken configuration. It will clean up after itself and show you the exact error, ensuring the live web server is never affected.
