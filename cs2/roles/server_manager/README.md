# `lanets.cs2.server_manager`

Install Auto Tournament's CS2 Server Manager (`csm`) and prepare a Linux host
with a CS2 server running Ready Up.

The role creates or reuses the configured Linux account, installs the official
CSM release for the host architecture, runs `csm setup-host`, bootstraps one
CS2 server, and installs Ready Up from its beta channel by default. The
executable is installed system-wide, while CSM commands that manage servers
run as the configured regular user. See the
[official install guide](https://docs.autotournament.gg/cs2/server-manager/install)
and [host requirements](https://docs.autotournament.gg/modules/cs2/server-requirements).

## Example

```yaml
- hosts: cs2_machines
  become: true
  roles:
    - role: lanets.cs2.server_manager
      vars:
        cs2_server_manager_user: cs2servermanager
        cs2_server_manager_version: latest
        cs2_server_manager_rcon_password: "{{ vault_cs2_rcon_password }}"
```

The role runs `csm bootstrap` only when `~/server-1` is absent, so a routine
Ansible run will not redeploy an existing server. `cs2_server_manager_rcon_password`
must be supplied from Ansible Vault or another secret store; the
`build-placeholder` value from a container build should not be used on a live
server. Ready Up's channel and license choice are set to `beta` and
`noncommercial`, then `csm update-plugins` installs the plugin once for that
channel/license combination. Set `cs2_server_manager_force_readyup_update: true`
for a one-off plugin update; unset it afterward to avoid updating during a
match.

If you select `commercial`, supply `cs2_server_manager_license_key`; the role
stores it with `csm license set` without printing it in Ansible output.

`cs2_server_manager_num_servers` is used only on the initial bootstrap; use
CSM's server management commands to add or remove servers later.
If bootstrap fails, Ansible reports disk space and points to
`/tmp/csm-bootstrap.log` on the target. Bootstrap output is hidden because it
may contain the RCON password.

For a platform-managed instance host instead of a standalone bootstrapped
server, set `cs2_server_manager_bootstrap_servers: false` and
`cs2_server_manager_configure_instance_backend: true`. After the role runs, use
the one-time command from **Servers → Machines → Add machine** on the host as
the configured CSM user. It links the machine and starts its host agent; the
platform can then create and scale servers. The link code is deliberately not
stored in inventory.

Set `cs2_server_manager_preload_game: true` when bootstrapping is disabled to
download the shared CS2 installation ahead of time (about 70 GB). Plan for at
least 8 GB RAM plus 1–2 GB per running server, and open each game port (UDP and
TCP) and GOTV port (UDP) in the machine's firewall. The defaults are game port
27015 and GOTV port 27020, increasing by 10 per server. `csm update-game` can
stop and start servers it manages, so preload before hosting matches.

## Variables

| Variable | Default | Purpose |
| --- | --- | --- |
| `cs2_server_manager_user` | `cs2servermanager` | Linux account that owns CSM data and runs its commands. |
| `cs2_server_manager_group` | `{{ cs2_server_manager_user }}` | Primary group for that account. |
| `cs2_server_manager_home` | `/home/{{ cs2_server_manager_user }}` | Home used by CSM. |
| `cs2_server_manager_binary` | `/usr/local/bin/csm` | Installed executable path. |
| `cs2_server_manager_version` | `latest` | Latest release or a pinned tag such as `v1.27.4`. |
| `cs2_server_manager_bootstrap_servers` | `true` | Run `csm bootstrap` if `server-1` does not exist. |
| `cs2_server_manager_num_servers` | `1` | Number of servers for bootstrap. |
| `cs2_server_manager_rcon_password` | required for bootstrap | RCON password, preferably stored in Ansible Vault. |
| `cs2_server_manager_hostname_prefix` | `CS2 Server` | Hostname prefix passed to bootstrap. |
| `cs2_server_manager_base_game_port` | `27015` | First server's game port. |
| `cs2_server_manager_base_tv_port` | `27020` | First server's GOTV port. |
| `cs2_server_manager_readyup_channel` | `beta` | Ready Up release channel (`stable` or `beta`). |
| `cs2_server_manager_readyup_license` | `noncommercial` | Ready Up license choice (`noncommercial` or `commercial`). |
| `cs2_server_manager_license_key` | empty | Required for the commercial choice; store it in Ansible Vault. |
| `cs2_server_manager_force_readyup_update` | `false` | Run `csm update-plugins` on each play when true. |
| `cs2_server_manager_configure_instance_backend` | `false` | Configure instance mode for platform-managed hosts. |
| `cs2_server_manager_preload_game` | `false` | Download/update CS2 during the play when bootstrap is disabled. |

Supported hosts are Debian or Ubuntu on `x86_64` or `arm64` with systemd. CSM
installs its system dependencies, including SteamCMD, during `setup-host`.
