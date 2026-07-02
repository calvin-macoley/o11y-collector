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

## Note: separate apt upgrade and install tasks

In `roles/alloy_commons/tasks/utilities.yml`, upgrading the system and
installing the utility packages are two distinct tasks. The `apt` module
treats `name`, `deb` and `upgrade` as mutually exclusive parameters, so a
single task combining `name: [...]` with `upgrade: yes` fails with
`parameters are mutually exclusive: deb|package|upgrade`. The first task runs
`update_cache` + `upgrade`, the second installs the packages.

