# ansible-role-tailscale

Installs Tailscale on Debian/Ubuntu from `pkgs.tailscale.com`, joins a tailnet, and reconciles node settings on every run.

- Join when `BackendState != Running` or advertised tags differ (`tailscale up`, `--force-reauth` if already running).
- Every run: `tailscale set` for hostname, ssh, accept-dns, advertise-routes, advertise-exit-node, auto-update.
- Validate: `Running`, has IPs, tags ⊇ `tailscale_tags`.

## Credential

First non-empty wins:

1. `tailscale_auth_key`: auth key or OAuth client secret.
2. `tailscale_aws_secret_id`: Secrets Manager secret whose JSON has `tailscale_aws_secret_json_key` (default `authKey`). Host needs the AWS CLI, `secretsmanager:GetSecretValue`, `kms:Decrypt`.

`tskey-client-*` secrets get `?ephemeral=false&preauthorized=true` appended.

## Use

`requirements.yml`:

```yaml
roles:
  - name: tailscale
    src: https://github.com/sdnit-se/ansible-role-tailscale.git
    scm: git
    version: v0.1.0
```

```yaml
- hosts: gateway
  become: true
  roles:
    - role: tailscale
      vars:
        tailscale_hostname: gateway
        tailscale_tags: ["tag:server"]
        tailscale_aws_secret_id: tailscale/gateway
        tailscale_aws_region: eu-north-1
```

Variables: [defaults/main.yml](defaults/main.yml).

## Develop

```bash
mise install
mise run check   # yamllint, ansible-lint (production profile), syntax check
```
