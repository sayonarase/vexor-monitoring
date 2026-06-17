# Vexor Monitoring – Installation (Rocky Linux 10 / RHEL 10-compatible)

> **Supported platform:** Rocky Linux 10, or any RHEL 10-compatible distribution
> (AlmaLinux 10, RHEL 10, Oracle Linux 10), **x86_64**.
> The public packages are built and tested **only** for EL10. EL9 and other
> distributions are not supported by the public release.

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

## Fallback: install from GitHub (if the Vexor repo is down)

The Vexor RPM server (`sayonara.dyndns.org`) is the primary source, but every
release also ships the **latest RPMs as assets on the GitHub release**, so you
can install even if that server is unreachable.

> The release assets are **EL10 (Rocky/RHEL 10), x86_64** only.

```bash
# 1. Download every RPM (+ checksums) from the latest release.
TAG=v0.1.0-preview
mkdir -p vexor-rpms && cd vexor-rpms
curl -s "https://api.github.com/repos/sayonarase/vexor-monitoring/releases/tags/$TAG" \
  | grep browser_download_url | cut -d'"' -f4 \
  | grep -E '\.(rpm|txt)$' | xargs -n1 curl -LO

# 2. Verify integrity.
sha256sum -c SHA256SUMS.txt

# 3. vexor-release still configures EPEL + InfluxData + CRB + the GPG keys,
#    which provide the third-party deps (nagios-plugins, influxdb2, ...).
dnf install -y ./vexor-release-*.rpm

# 4. Install everything from the local files. We disable the (unreachable)
#    'vexor' repo so dnf uses the downloaded RPMs, while EPEL / InfluxData /
#    AppStream stay enabled to resolve third-party dependencies.
dnf install -y --disablerepo=vexor ./vexor-*.rpm

# 5. First-run setup.
vexor-setup
```

> Note: this offloads the **Vexor** packages from the primary server, but a few
> third-party dependencies (e.g. `java-21-openjdk-headless`, `postgresql-server`,
> `nagios-plugins-*`, `influxdb2`) are still pulled from the standard EPEL /
> InfluxData / distro repositories — those are not redistributed here.

## Alternative: Docker (any Linux host)

Rocky / RHEL 10 is the supported production platform, but for evaluation — or
to run on a non-RHEL host (Ubuntu, Debian, …) — Vexor is also published as an
all-in-one Docker image. It installs and runs the **same RPMs** under systemd
inside one container, so it does not touch or depend on the host OS.

```bash
git clone https://github.com/sayonarase/vexor-docker
cd vexor-docker
cp .env.example .env
# Set VEXOR_PUBLIC_URL to EXACTLY how testers reach the container in the
# browser, including the port if you publish on anything other than 443
# (e.g. https://monitor.example.com or https://192.168.1.50:8453). It is
# registered with Keycloak on first boot so single sign-on works.
docker compose up -d

# Initial admin credentials (written on first boot):
docker compose exec vexor cat /etc/vexor/.initial-admin
```

Image: `ghcr.io/sayonarase/vexor:latest`. Full notes, caveats (systemd-in-Docker,
ICMP checks, data volumes) and the compose file live in the
**[sayonarase/vexor-docker](https://github.com/sayonarase/vexor-docker)** repo.

## Upgrades

```bash
dnf upgrade -y vexor-server
systemctl restart vexor-api naemon nginx
```

vexor-release is `%config(noreplace)` so any local edits to the repo
files survive upgrades.
