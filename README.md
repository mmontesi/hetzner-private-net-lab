# Hetzner private network lab

This repository contains Ansible playbooks and roles to create a small private-network lab on Hetzner. It's intended as an educational example and automation reference — **not for production use!**

**Highlights**
- Create private networks on Hetzner (EU region).
- Provision a gateway (Ubuntu) with a public IP that provides:
  - WireGuard access (via `wg-easy` + `caddy` for HTTPS/DuckDNS).
  - HTTP/HTTPS proxy and DNS for private hosts (`squid`, `dnsmasq`).
- Deploy cloud servers (Ubuntu) into the private networks with automated provisioning.

**TO DO**
- [ ] Add a firewall rule to restrict SSH access to the gateway to a configurable IP (default: detected devbox IP).
- [ ] Provide a teardown playbook to destroy created networks, VMs and related resources cleanly.
- [x] ~~Add dynamic region and server-type selection to reduce failures from capacity limits (e.g. `server-type cpx42 not available in fsn1`)~~.
- [ ] Support routing between up to 3 private networks to work around per-network 100 servers limit.
- [ ] Support Debian and RH-like images (Rocky/Alma).

## Repository layout

- `inventory-example.yml` — example Ansible inventory
- `provision-networks.yml` — playbook to create private networks and gateway
- `provision-private-vms.yml` — playbook to create VMs inside the private networks
- `roles/` — Ansible roles for `gateway`, `network`, and `private-vm`

## Prerequisites

- A Hetzner Cloud project and an API token (`HCLOUD_TOKEN`).
- (Optional) A DuckDNS account if you want automatic HTTPS for `wg-easy`.
- Ansible and required Python packages are provided by the repository's devcontainer; if you don't use the devcontainer, install them locally.

## Quick start

1. Copy the example inventory:

```bash
cp inventory-example.yml inventory.yml
```

2. Create a local environment file `.local/testing.env` with the secrets referenced by your inventory, for example:

```bash
mkdir -p .local
cat > .local/testing.env <<EOF
HCLOUD_TOKEN=CTgMW5...    # your Hetzner API token
NEWUSER_PASSWORD=ChangeMe  # password for new user created on VMs
EOF
```

3. Load environment variables and run the playbooks:

```bash
set -a; source .local/testing.env; set +a

# create private network(s) and gateway
ansible-playbook -i inventory.yml provision-networks.yml -e new_user_password="$NEWUSER_PASSWORD" -e hcloud_token="$HCLOUD_TOKEN"

# create private VMs (limit example)
ansible-playbook -i inventory.yml provision-private-vms.yml --limit private-vm01 -e vm_new_user_password="$NEWUSER_PASSWORD" -e hcloud_token="$HCLOUD_TOKEN"
```

4. Inspect the generated resources in the Hetzner Cloud console and verify connectivity.

## Configuration notes

- Templates are in `roles/*/templates/*.j2` and are applied by the corresponding roles. Adjust variables in your `inventory.yml` or group/host vars as needed.
- The gateway role includes configuration for WireGuard, proxy and DNS; review `roles/gateway/templates` before running in any environment.

## Dynamic provisioning

This repository includes a `dynamic-private-vm` role that tries to create VMs across multiple locations and compatible server types until one succeeds. The role builds an ordered list of (location, server_type) attempts using the variables `vm_location`, `vm_fallback_locations`, `vm_server_type` and `vm_server_types_compatibility`.

Locations are deduplicated and the requested `vm_server_type` is prioritized; compatible alternatives are tried in order. The role iterates the combinations and calls the `hetzner.hcloud.server` module for each attempt, ignoring individual failures and stopping as soon as a server is created. On success it sets facts such as `vm_created`, `vm_server`, `vm_final_location`, `vm_final_server_type` and `vm_server_ip` for later tasks.

__NB__ Most of the code is duplicated, and the 2 roles `dynamic-private-vm` and `private-vm` should be merged, but this project is meant only to show possibilities ;)

## Security & warnings

- This is example code for learning and experimentation. Do not use it as-is for production environments.
- Keep your API tokens and passwords out of version control. Use `.local/` or a secret manager.

## Contributing

Contributions and suggestions are welcome. Open an issue or submit a PR with improvements or fixes.

## Files of interest

- See `inventory-example.yml` for how to structure hosts and variables.
- Playbooks: `provision-networks.yml`, `provision-private-vms.yml`.

## License

This repository is licensed under the MIT License — see the `LICENSE` file.

---
