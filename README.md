<!-- @format -->

# I have updated the charts. I haven't updated the below guide yet. Basic changes are:

1. Now using the Dockge default network, not the Traefik default network. I may change this to make an entirely separate network in the future.
2. Added middleware based on smarthomebeginner guides. Adjusted based on Dockge usage.
3. To support middleware, smarthomebeginner guides suggests setting up a rules directory. So I mounted a directory, created folders and files based on the guide here: https://www.smarthomebeginner.com/traefik-v3-docker-compose-guide-2024/ . Dont need to follow the whole guide. But you do need to follow the guide to setup acme.json, the permissions (chmod 600) and the rules folder.
4. I'm not including the rules i've setup. But they're the same as the guide from: https://www.smarthomebeginner.com/traefik-v3-docker-compose-guide-2024/. Look towards the end of the guide and follow which files to add to the rules folder, and what content to put in the files.
5. Added google oAuth. Follow this guide to setup google correctly: https://www.smarthomebeginner.com/google-oauth-traefik-forward-auth-2024/. This should get you your client ID and secret. Put the ids/secret/oauth secret in the .env section of Dockge.

# TrueNAS Scale Docker Compose Setup with Jailmaker and Cloudflare Tunnels

This guide will help you set up Docker containers on TrueNAS Scale using Jailmaker and Dockge, configure your domains via Cloudflare tunnels, and get Traefik and various home server containers running. Follow the steps below to get started.

## Prerequisites

- **TrueNAS Scale** on Dragonfish
- **Jailmaker** setup and **Dockge** running
- A domain managed by **Cloudflare**
- Follow these YouTube videos:
  - [Setup Cloudflare tunnels and configure domains/subdomains](https://www.youtube.com/watch?v=yMmxw-DZ5Ec)
  - [Setup TrueNAS Scale to run Jailmaker and Dockge](https://www.youtube.com/watch?v=yMmxw-DZ5Ec)

## Troubleshooting Tips

### Dockge Domain Setup

When setting up Dockge and Jailmaker, you might want to make Dockge available on a domain, e.g., `dockge.somedomain.com`. Follow these steps:

1. Follow the Cloudflare setup tutorial.
2. Create a domain name like `dockge.somedomain.com`.
3. Point the URL to the internal IP of your Docker instance with port `5001`.

> **Note:** Cloudflare tunnels allow external access by pointing the domain to the internal IP address and necessary port.

### Other Domains Setup

For other services like `plex.somedomain.com` or `sonarr.somedomain.com`:

1. Point them to the IP of your Docker instance with port `443`.
2. Traefik will forward requests to the appropriate services based on the domain.

> **Note:** Plex requires an additional label for HTTPS communication. This is included in the compose files to avoid Cloudflare errors.

## Setup Guide

### Docker Jail and Mount Points

1. Set up a Docker jail instance with Dockge.
2. Add mount points to the Docker jail as explained in the tutorial.
3. Mount your directories (e.g., Plex, torrent, config folders) to the Docker jail.

### User Permissions

- Docker Jail and Dockge run as root.
- Other apps should run as a specific user (e.g., user 3000) and group (e.g., group 568).
- Set the datasets and configs with appropriate permissions.

### Environment Variables

Various compose files require Cloudflare tokens, API keys, etc. Move these keys to the `.env` section of Dockge.

### Compose Files Order

1. **Traefik**: Sets up the default Traefik network. It won't work initially as Cloudflared isn't running yet.
2. **Cloudflared**: Sets up the tunnel to Cloudflare. Enables remote access via domain names.
3. **Plex**: Points to your existing library.
4. **Flaresolverr**: Used by Prowlarr.
5. **Prowlarr**: Sets up indexers. Connects to Sonarr and Radarr.
6. **Sonarr**: Connects to Prowlarr using an API key.
7. **Radarr**: Connects to Prowlarr using an API key.
8. **Qbittorrent**: Sets up download clients. Sync changes with Sonarr/Radarr.
9. **Overseer**: Connects to Plex, Radarr, and Sonarr.
10. **Unpackerr**: Extracts downloaded files. Needs API keys for Radarr and Sonarr.
11. **Romm** (Optional): Used for ROMs.

> **Note:** All containers should be on the same network (default_traefik) for communication.

## Security and Future Improvements

This is a basic setup for proof of concept. Enhance security by studying more about Traefik middleware and removing insecure APIs. Experiment and add security measures as needed.

## Conclusion

This guide provides a simple way to set up and access apps from the outside world using Cloudflare tunnels and Traefik. Follow the steps and videos carefully, and you'll have a functional and accessible HomeLab setup.
