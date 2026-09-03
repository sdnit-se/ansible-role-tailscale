# ansible-role-tailscale

Installs Tailscale on Debian/Ubuntu from `pkgs.tailscale.com`, joins a tailnet, and reconciles node settings on every run.

- Join when `BackendState != Running` or advertised tags differ (`tailscale up`, `--force-reauth` if already running).
- Every run: `tailscale set` for hostname, ssh, accept-dns, advertise-routes, advertise-exit-node, auto-update.
- Validate: `Running`, has IPs, tags ⊇ `tailscale_tags`.

## Credential

First non-empty wins:

1. `tailscale_auth_key`: auth key or OAuth client secret.
2. `tailscale_federated_client_id` + `tailscale_federated_audience`: workload identity federation. `tailscale up --client-id --audience` exchanges the host's cloud identity (AWS instance role via `sts:GetWebIdentityToken`, GCP, GitHub) for an auth key. Needs a Tailscale federated identity with scope `auth_keys` and the host's tags; nothing stored on the host. Tailscale 1.90+. On AWS set `tailscale_aws_region`.

`tailscale_force_reauth: true` re-authenticates once even if running (cutovers).
`tailscale_dry_run: true` changes nothing and still validates the node.

`tskey-client-*` secrets get `?ephemeral=false&preauthorized=true` appended.

## Use

`requirements.yml`:

```yaml
roles:
  - name: tailscale
    src: https://github.com/sdnit-se/ansible-role-tailscale.git
    scm: git
    version: v0.3.0
```

```yaml
- hosts: gateway
  become: true
  roles:
    - role: tailscale
      vars:
        tailscale_hostname: gateway
        tailscale_tags: ["tag:server"]
        tailscale_federated_client_id: k1234511CNTRL-kABCDE11CNTRL
        tailscale_federated_audience: api.tailscale.com/k1234511CNTRL-kABCDE11CNTRL
        tailscale_aws_region: eu-north-1
```

Variables: [defaults/main.yml](defaults/main.yml).

## Develop

```bash
mise install
mise run check   # yamllint, ansible-lint (production profile), syntax check
```
