# Secure private network lab on Hetzner

## Disclaimer
This is a WIP project to collect examples and it's meant for educational purposes. 

**IT'S NOT PRODUCTION READY!!!**

## Overview

A collection of Ansible playbooks and roles to automate the creation of:

- A set of (up to 3) private networks on Hetzner (EU network).
- A gateway server with a public IP able to:
  - Manage a Wireguard VPN to access the private network with `wg-easy` (Let's Encrypt certificate provided by `caddy` for a DuckDNS subdomain).
  - Provide HTTP/HTTPS proxy (`squid`) and DNS (`dnsmasq`) functionalities for the private servers.
- A firewall to allow SSH conncetions to the gateway server only from your public IP.
- Cloud servers in the private network, with automated configuration for HTTP proxy and DNS.

## Prerequisites

- A valid [DuckDNS](https://www.duckdns.org/) account (free up to 5 subdomains).
- A valid API token for a project in Hetzner.

## How to run it

Open the devcontainer in VS Code (Ctrl+Shift+P -> Dev containers: Reopen in Container)

Create a file called `.local/testing.env` containing:
```
HCLOUD_TOKEN=CTgMW5******
NEWUSER_PASSWORD=<your password>
```
create your own `inventory.yml` file (see `inventory-example.yml`), and then
```bash
set -a; source .local/testing.env; set +a
# create private network(s) with gateway
ansible-playbook -i inventory.yml provision-networks.yml -e new_user_password="$NEWUSER_PASSWORD" -e hcloud_token=$HCLOUD_TOKEN

# create private VMs
ansible-playbook -i inventory.yml  provision-private-vms.yml --limit private-vm01  -e vm_new_user_password="$NEWUSER_PASSWORD" -e hcloud_token=$HCLOUD_TOKEN
```