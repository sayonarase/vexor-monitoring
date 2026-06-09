# Vexor Monitoring – Installation (Rocky Linux 10 / 9)

## Quick install on a fresh server

Run as `root` (or prefix every command with `sudo`):

```bash
# 1. One-shot bootstrap — installs the Vexor + InfluxData repo definitions,
#    pulls in EPEL, enables CRB and imports both GPG keys automatically.
dnf install -y https://sayonara.dyndns.org:8443/vexor-release-latest-el$(rpm -E %rhel).rpm

# 2. Install the whole server (api, ui, naemon, keycloak, mariadb-server,
#    nginx, java, influxdb2, wmi-plugin, log-aggregation, ...). All
#    third-party deps now resolve through repos configured in step 1.
dnf install -y vexor-server

# 3. Finish first-run setup (creates DBs, generates passwords, configures
#    keycloak, enables systemd units, opens firewall).
vexor-setup
```

That's it — three commands, no manual repo or dependency wrangling.

## What `vexor-release` does for you

Installing `vexor-release` configures the following on the host:

| Item                                  | Purpose                                                       |
|---------------------------------------|---------------------------------------------------------------|
| `/etc/yum.repos.d/vexor.repo`         | Vexor RPM repository                                          |
| `/etc/yum.repos.d/influxdata.repo`    | InfluxData repository (for `influxdb2` + `influxdb2-cli`)      |
| `/etc/pki/rpm-gpg/RPM-GPG-KEY-vexor`  | Vexor signing key (imported into the rpm keyring)             |
| `epel-release` (hard dep)             | EPEL — required for `nagios-plugins-*`, `sshpass`, `lftp`, etc.|
| `dnf-plugins-core` (hard dep)         | Provides `dnf config-manager` so we can enable CRB             |
| `crb` / `powertools` repo             | Enabled in `%post` — needed by several EPEL perl modules       |
| InfluxData GPG key                    | Imported in `%post`, so installing influxdb2 doesn't prompt    |

## Minimal install (no log aggregation / no WMI plugin)

`vexor-server` declares the optional pieces as `Recommends`, not `Requires`,
so a slimmer install is one command away:

```bash
dnf install -y --setopt=install_weak_deps=False vexor-server
```

This skips `vexor-plugin-wmiplus`, `vexor-logs`, `vexor-vector` and
`vexor-victorialogs`. The core monitoring + alerting + UI still install.

## Air-gapped install

If your server can't reach the internet, mirror the three .repo sources
locally and point `vexor.repo` / `influxdata.repo` / `epel.repo` at your
mirror. The dependency graph itself is unchanged.

## Upgrades

```bash
dnf upgrade -y vexor-server
systemctl restart vexor-api naemon nginx
```

vexor-release is `%config(noreplace)` so any local edits to the repo
files survive upgrades.
