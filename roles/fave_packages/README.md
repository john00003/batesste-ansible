# fave_packages

Install a curated set of development and systems packages
on Ubuntu.

## Packages Installed

The role installs the following packages on all supported
platforms:

aspell, bmon, build-essential, cmake, driverctl,
emacs-nox, expect, fio, git, gnuplot, gpg-agent,
libaio-dev, liburing-dev, linux-generic, npm,
pipx, prometheus-node-exporter,
prometheus-node-exporter-collectors, python-is-python3,
python3, python3-pip, python3-venv, mutt, sysstat, tmux,
tree, xfsprogs.

The role also installs `uv` via pip so that `uv` and `uvx`
are available, and installs `ptop` from the latest GitHub
release for supported `amd64` and `arm64` Ubuntu systems.

`pipx` is included so that tools like `ansible-core` and
`ansible-lint` can be installed into isolated environments
without conflicting with system Python packages. On Ubuntu
24.04 this is the recommended way to get a recent
`ansible-core` (the apt package is outdated).

On Ubuntu noble and resolute the role also installs:
elpa-cmake-mode, elpa-rust-mode, virtiofsd, and
conditionally git-email (when git < 2.43).

## Node Exporter Configuration

After installing `prometheus-node-exporter` the role
deploys `/etc/default/prometheus-node-exporter` with
extra flags controlled by
`fave_packages_node_exporter_extra_args`. By default
this enables the WiFi collector (`--collector.wifi`)
which is disabled upstream.

### Role Variables

| Variable | Default | Description |
|---|---|---|
| `fave_packages_node_exporter_extra_args` | `--collector.wifi` | Extra CLI flags for `prometheus-node-exporter` |
| `fave_packages_install_packer` | `true` | Install `packer`, adding the HashiCorp apt source and signing key |

Override the variable to add more collectors or disable
the WiFi collector:

```yaml
# Enable WiFi and thermal zone collectors
fave_packages_node_exporter_extra_args: >-
  --collector.wifi --collector.thermal_zone

# Disable all extra collectors (use upstream defaults)
fave_packages_node_exporter_extra_args: ""
```

## HashiCorp packer

`packer` is the only package this role takes from
`apt.releases.hashicorp.com`. Installing it also leaves that
apt source and its signing key on the host, so set
`fave_packages_install_packer: false` to skip the repository
entirely.

The signing key is re-downloaded and re-dearmored on every
run rather than being written once. HashiCorp rotate the key
periodically (the current one, `FC9CA96ACA026560`, was issued
2026-09-09), and a host whose keyring predates a rotation
otherwise fails every later `apt update` with `NO_PUBKEY`.

## Example Playbook

```yaml
---
- name: Install favourite packages
  hosts: all
  roles:
    - sbates130272.batesste.fave_packages
```

Skip the HashiCorp apt source:

```yaml
---
- name: Install favourite packages without packer
  hosts: all
  roles:
    - role: sbates130272.batesste.fave_packages
      vars:
        fave_packages_install_packer: false
```

## Dependencies

Includes `check_platform` automatically.

## Supported Platforms

- Ubuntu noble (24.04)
- Ubuntu resolute (26.04)

## Note on prometheus-node-exporter

This role installs `prometheus-node-exporter` as part of its default package
list. The `grafana_setup` and `consul_setup` roles also install it independently
so each role remains usable standalone. Running more than one of these roles on
the same host is safe (idempotent), but if you want to manage node-exporter
exclusively through one role, set `grafana_setup_install_node_exporter: false`
and/or remove the package from `fave_packages_packages` as needed.

## License

Apache-2.0

## Author

Stephen Bates (sbates@raithlin.com)
