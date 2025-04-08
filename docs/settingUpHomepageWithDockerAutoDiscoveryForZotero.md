# Setting Up Homepage with Docker Auto-Discovery for Zotero

This guide outlines the steps to configure [Homepage](https://github.com/gethomepage/homepage) with auto-discovery for a [Zotero Docker container](https://docs.linuxserver.io/images/docker-zotero/). The [`tecnativa/docker-socket-proxy`](https://github.com/Tecnativa/docker-socket-proxy) is utilized to securely access the Docker API, enabling Homepage to detect and display the Zotero container based on its labels.

## Prerequisites

* **Docker** and **Docker Compose** are installed on the system.
* A **Zotero** instance is operational within a Docker container.
* Familiarity with Docker networks and volumes is assumed.

## Step 1: Create a Docker Network

A dedicated bridge network is established to facilitate communication between Homepage and the `docker-socket-proxy`.

```bash
sudo docker network create homepage-net
```

## Step 2: Configure the Docker Socket Proxy

The [docker-socket-proxy](https://github.com/Tecnativa/docker-socket-proxy) service is set up to provide secure, read-only access to the Docker API.

**Docker Compose Configuration for the Proxy**

```yaml
# docker-compose.yaml for docker-socket-proxy
# Purpose: Configures a proxy to securely expose the Docker socket with read-only access
services:
  docker-socket-proxy:
    image: tecnativa/docker-socket-proxy:latest
    container_name: docker-socket-proxy
    environment:
      - CONTAINERS=1 # Enables read-only access to container info
      - POST=0 # Disables POST operations for safety
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    ports:
      - 127.0.0.1:<host-port>:2375 # Expose only on localhost; replace <host-port> with desired port
    restart: unless-stopped
    networks:
      - homepage-net
networks:
  homepage-net:
    external: true
```

## Step 3: Configure Homepage

Homepage is configured to leverage the proxy for Docker API access, enabling [automatic service discovery](https://gethomepage.dev/configs/docker/#automatic-service-discovery) of labeled containers.

**Docker Compose Configuration for Homepage (`docker-composeyaml`)**

```yaml
# docker-compose.yaml for homepage
# Purpose: Sets up Homepage to connect to the proxy and display Docker containers
services:
  homepage:
    image: ghcr.io/gethomepage/homepage:v1.1.1
    container_name: homepage
    ports:
      - <host-port>:3000  # Replace <host-port> with desired external port
    volumes:
      - <config-path>:/app/config  # Replace <config-path> with local config directory
    environment:
      - LOG_LEVEL=info
      - PGID=<group-id>            # Replace with desired group ID
      - PUID=<user-id>             # Replace with desired user ID
      - TZ=<timezone>              # Replace with desired timezone (e.g., UTC)
      - HOMEPAGE_ALLOWED_HOSTS=<allowed-hosts>  # Replace with allowed hosts (e.g., localhost:<port>)
    restart: unless-stopped
    networks:
      - homepage-net
networks:
  homepage-net:
    external: true
```

**Homepage Settings (settings.yaml)**

In the Homepage configuration directory (e.g., `<config-path>/config/settings.yaml`), the following is added:

```yaml
# settings.yaml for homepage
# Purpose: Configures Homepage to use the Docker proxy for container discovery
providers:
  docker:
    my-docker:
      host: docker-socket-proxy
      port: 2375
```

## Step 4: Configure Zotero with Labels

The Zotero container is labeled to allow Homepage to detect and display it automatically.

**Docker Compose Configuration for Zotero**

```yaml
# docker-compose.yaml for zotero
# Purpose: Configures the Zotero container with labels for Homepage auto-discovery
services:
  zotero:
    image: linuxserver/zotero:7.0.15-ls71
    container_name: zotero
    ports:
      - <host-port>:3001  # Replace <host-port> with desired external port
    volumes:
      - <zotero-config-path>:/config  # Replace with local config directory
    environment:
      - PGID=<group-id>              # Replace with desired group ID
      - PUID=<user-id>               # Replace with desired user ID
      - TZ=<timezone>                # Replace with desired timezone (e.g., UTC)
      - CUSTOM_USER=<username>       # Replace with desired username
      - PASSWORD=<password>          # Replace with desired password
    restart: unless-stopped
    labels:
      - homepage.group=Productivity
      - homepage.name=Zotero
      - homepage.icon=https://example.com/zotero.png  # Replace with desired icon URL
      - homepage.href=http://<server-ip>:<port>       # Replace with Zotero's external URL
      - homepage.description=Zotero is a tool to help collect, organize, annotate, cite, and share research.
    networks:
      - default
```

## Step 5: Deploy and Verify the Setup

The following steps ensure all services are operational and properly configured.

### Deployment Steps

1. **Start the Services:**
    * Execute `docker-compose up -d` for the `docker-socket-proxy` and Homepage stacks.
    * Confirm the Zotero container is running.
2. **Verify Network Connectivity:**
	* Test if Homepage can reach the proxy:
        ```bash
        sudo docker exec -it homepage ping -c 3 docker-socket-proxy
        ```
3. **Inspect Zotero Labels:**
    * Verify the labels are correctly applied:
        ```bash
        sudo docker inspect zotero | grep homepage
        ```
4. **Restart Homepage:**
    * Apply any configuration changes:
        ```bash
        sudo docker restart homepage
        ```
5. **Check Logs:**
    * Review logs for errors or confirmation:
        ```bash
        sudo docker logs homepage
        ```
6. **Access Homepage:**
    * Navigate to `http://<server-ip>:<homepage-port>` in a browser (replace with actual server IP and port).
    * Check for Zotero under the "Productivity" group.

## Troubleshooting

* Zotero Not Appearing:  
    * Confirm labels are accurate and present using docker inspect zotero.  
    * Examine Homepage logs for Docker API-related errors.  
* Network Issues:  
    * Verify homepage-net includes both Homepage and the proxy with docker network inspect homepage-net.  
* Proxy Warnings:  
    * Disregard HAProxy warnings unless they affect functionality (see [Issue #123](https://github.com/Tecnativa/docker-socket-proxy/issues/123) as of April 2025).

This configuration enables Homepage to automatically detect and display the Zotero container using Docker labels. For persistent issues, consult the troubleshooting steps above.
