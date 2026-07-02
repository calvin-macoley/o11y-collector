# o11y-collector

Ansible deployment of the Grafana Alloy collector (see `o11y-collector-deployment/`).

## Note: explicit PATH in the playbooks

The plays in `alloy-commons.yml` and `alloy-install.yml` set:

```yaml
environment:
  PATH: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

On minimal Debian/Ubuntu targets (e.g. LXC containers) sudoers may not define
`secure_path`, so the become-root session inherits the connecting user's PATH,
which lacks `/usr/sbin` and `/sbin`. `dpkg` then fails to install the Alloy
package because it cannot find `ldconfig` and `start-stop-daemon`. Setting the
full PATH at play level makes every task — including those from the
`grafana.grafana.alloy` collection role — run with the sbin directories
available. Alternatively, fix the host itself by adding
`Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"`
to its sudoers.

## Note: accessing the Alloy UI temporarily

Alloy serves its web UI on `127.0.0.1:12345` on the target VM, so it is not
reachable from outside. For a quick look, do not change the deployment —
open an SSH tunnel instead:

```bash
ssh -N -L 12345:localhost:12345 \
    -i o11y-collector-deployment/secrets/alloy \
    ansible@192.168.1.5
```

Then browse to `http://localhost:12345`. `-N` only forwards the port (no
shell); `Ctrl+C` closes the tunnel and access is gone. Nothing changes on the
VM: no config edit, no restart, no firewall opening, and the UI — which has no
authentication — is never exposed on the network. If 12345 is already in use
locally, change the left-hand port: `-L 8080:localhost:12345` →
`http://localhost:8080`.

For *permanent* exposure, pass the listen address to the
`grafana.grafana.alloy` role instead (requires a deploy to enable and another
to revert):

```yaml
alloy_env_file_vars:
  CUSTOM_ARGS: "--server.http.listen-addr=0.0.0.0:12345"
alloy_expose_port: true
```

## Note: separate apt upgrade and install tasks

In `roles/alloy_commons/tasks/utilities.yml`, upgrading the system and
installing the utility packages are two distinct tasks. The `apt` module
treats `name`, `deb` and `upgrade` as mutually exclusive parameters, so a
single task combining `name: [...]` with `upgrade: yes` fails with
`parameters are mutually exclusive: deb|package|upgrade`. The first task runs
`update_cache` + `upgrade`, the second installs the packages.

