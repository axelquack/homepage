# Setting Up JDownloader with Docker and Homepage

## Table of Contents

1. **[Prerequisites](#prerequisites)**
2. **[Initial Setup](#initial-setup)**
3. **[Configuring Docker Compose](#configuring-docker-compose)**
4. **[Handling Permissions](#handling-permissions)**
5. **[Integrating with Homepage](#integrating-with-homepage)**
6. **[Enabling Dark Mode](#enabling-dark-mode)**
7. **[Ad Removal within JDownloader](#ad-removal-within-jdownloader)**
8. **[Security Considerations](#security-considerations)**
9. **[Additional Improvements](#additional-improvements)**
10. **[Troubleshooting](#troubleshooting)**

## Prerequisites

Before starting, ensure the following requirements are met:

* **Operating System**: A Linux-based system with Docker support enabled.
* **Docker and Docker Compose**: These tools must be installed. This guide uses [Dockge](https://github.com/louislam/dockge) for managing Docker Compose, but commands are adapted for standalone use.
* **MyJDownloader Account**: Required for remote access and Homepage widget integration. Register at [my.jdownloader.org](https://my.jdownloader.org/).
* **Basic Knowledge**: Familiarity with Docker, Docker Compose, and Linux permissions is helpful.

## Initial Setup

The initial setup involves creating a directory for JDownloader and preparing an environment file.

1. **Create a Directory for JDownloader**:  
   On the host system, create a directory to store the Docker Compose file and configuration data:
```bash
   mkdir -p /path/to/appdata/jdownloader
   cd /path/to/appdata/jdownloader
```

Replace `/path/to/appdata` with a suitable location on your system (e.g., a data directory provided by your OS).

2. **Prepare the .env File:**  
    Create an environment file to store configuration variables: `touch .env`  
    This file will hold sensitive and customizable settings, detailed in the next section.

## Configuring Docker Compose

Docker Compose defines and runs the JDownloader container. Follow these steps to set it up.

* **Create the Docker Compose File:**  
    In the `/path/to/appdata/jdownloader directory`, create a file named `docker-compose.yaml` with the following content:
```yaml
    # Defines the project name for Docker Compose, used as a prefix for resources
    name: jdownloader
    services:
      # Defines the JDownloader service
      jdownloader:
        # Specifies the Docker image to use
        image: jlesage/jdownloader-2:latest
        # Sets a fixed name for the container
        container_name: jdownloader
        # Environment variables for container configuration
        environment:
          # Timezone setting, sourced from .env (e.g., Europe/Berlin)
          - TZ=${TZ}
          # User ID for container processes, sourced from .env (e.g., 1000)
          - USER_ID=${PUID}
          # Group ID for container processes, sourced from .env (e.g., 1000)
          - GROUP_ID=${PGID}
          # Sets locale to German for language formatting
          - LANG=de_DE.UTF-8
          # Web interface username, sourced from .env
          - WEB_USER=${WEB_USER}
          # Web interface password, sourced from .env
          - WEB_PASSWORD=${WEB_PASSWORD}
          # Enables dark mode in the web interface
          - DARK_MODE=1
        # Maps host directories to container paths
        volumes:
          # Configuration directory
          - ${JD_CONFIG_PATH}:/config
          # Directory for game downloads
          - ${DOWNLOAD_PATH}:/download
        # Maps the host port to the container's web interface port
        ports:
          - ${WEB_PORT}:5800
        # Limits memory usage to 4GB
        mem_limit: 4g
        # Restarts the container unless manually stopped
        restart: unless-stopped
        # Labels for Homepage dashboard integration
        labels:
          - homepage.group=${HOMEPAGE_GROUP}
          - homepage.name=${HOMEPAGE_NAME}
          - homepage.icon=${HOMEPAGE_ICON}
          - homepage.href=http://${SERVER_IP}:${WEB_PORT}
          - homepage.description=${HOMEPAGE_DESCRIPTION}
          - homepage.widget.type=jdownloader
          - homepage.widget.username=${MYJD_USERNAME}
          - homepage.widget.password=${MYJD_PASSWORD}
          - homepage.widget.client=${MYJD_CLIENT}
          - homepage.widget.fields=["downloadCount","downloadTotalBytes","downloadSpeed"]
```

* **Populate the `.env` File:** 
    Edit the `.env` file with generic placeholders for customization. Replace placeholders (e.g., `<your-ip>`, `<user-id>`) with values specific to your system:
```bash
## Server IP or Hostname
SERVER_IP=<your-ip>  # Replace with your IP or hostname (e.g., 192.168.x.x)

# User and Group IDs
PUID=<user-id>  # User ID matching your system (e.g., 1000)
PGID=<group-id>  # Group ID matching your system (e.g., 1000)

# Timezone
TZ=<your-timezone>  # Replace with your timezone (e.g., UTC, America/New_York)

# Web Authentication
WEB_USER=<web-username>  # Choose a username for the web interface
WEB_PASSWORD=<web-password>  # Choose a secure password for the web interface

# Ports
WEB_PORT=<port-number>  # Port for web access (e.g., 5800)

# Volume Paths
JD_CONFIG_PATH=/path/to/appdata/jdownloader/config  # Configuration storage path
DOWNLOAD_PATH=/path/to/data/download  # Path for downloads

# Homepage Labels
HOMEPAGE_GROUP=Tools  # Category for Homepage dashboard
HOMEPAGE_NAME=JDownloader  # Display name in Homepage
HOMEPAGE_ICON=https://cdn.jsdelivr.net/gh/IceWhaleTech/CasaOS-AppStore@main/Apps/JDownloader2/icon.png  # Icon URL
HOMEPAGE_HREF=http://${SERVER_IP}:${WEB_PORT}  # Access URL
HOMEPAGE_DESCRIPTION=Download Manager  # Description in Homepage

# MyJDownloader Credentials
MYJD_USERNAME=<myjd-username>  # Your MyJDownloader username
MYJD_PASSWORD=<myjd-password>  # Your MyJDownloader password
MYJD_CLIENT=<myjd-device-name>  # Unique device name set in JDownloader UI
```

* **Launch the Container:**  
    Start the JDownloader container: `docker-compose up -d`  
    If using Dockge, paste the docker-compose.yaml content into the Dockge UI and deploy.

## Integrating with Homepage

Integrating JDownloader with Homepage adds a widget for monitoring downloads.

1. **Confirm Labels:**  
    Ensure the `docker-compose.yaml` file includes Homepage labels for service discovery and widget setup.
2. **Configure MyJDownloader:**
    * Access the JDownloader web interface at `http://<your-nas-ip>:<port-number>`.
    * Go to Settings > MyJDownloader, enter the MyJDownloader credentials, and set a device name matching `MYJD_CLIENT` in the `.env` file.
3. **Validate Widget Functionality:**
    * Check the Homepage dashboard to confirm the widget displays correctly.
    * If an "*API Error: Unknown proxy service type*" occurs, ensure Homepage does not route external requests through the Docker proxy (e.g., adjust Homepage’s `settings.yaml` to bypass `127.0.0.1:2375` for external APIs).

## Enabling Dark Mode

To enable dark mode in the JDownloader interface:

1. **Set the Environment Variable:**  
    Confirm `- DARK_MODE=1` is in the environment section of `docker-compose.yaml`.
2. **Restart the Container:**  
    Apply the change: `docker-compose down && docker-compose up -d`

## Ad Removal within JDownloader

To remove ads in JDownloader:

1. Open the web interface and navigate to `Settings > Advanced Settings`.
2. Search for these keywords and disable (uncheck) the options:
    ```text
    premium alert
    oboom
    Special Deals
    Donate
    Banner
    ```
3. Restart JDownloader if needed. The main page banner should disappear immediately.

## Security Considerations

Enhance the security of the JDownloader setup:

* **Docker Socket Proxy:** Bind the proxy to `127.0.0.1:2375` to restrict external access (e.g., in the proxy’s `docker-compose.yaml`).
* **HTTPS:** Enable HTTPS by adding `ENABLE_SSL=yes` to the `environment` section and mounting SSL certificates (e.g., `/path/to/certs:/config/certs:ro`).
* **Credential Management:** Store sensitive data in the `.env` file to avoid hardcoding in `docker-compose.yaml`.

## Additional Improvements

Consider these enhancements:

* **Memory Optimization:** Monitor memory usage with `docker stats` and adjust `mem_limit` (e.g., to `2g`) if underutilized.
* **Configuration Backup:** Back up the `/config` directory periodically to preserve settings and history.
* **Reverse Proxy:** Use a reverse proxy (e.g., Nginx) for simplified access and HTTPS termination.

## Further documentation

* **GitHub:** [jlesage/docker-jdownloader-2](https://github.com/jlesage/docker-jdownloader-2)
* [JDownloader Homepage Widget](https://gethomepage.dev/widgets/services/jdownloader/)
