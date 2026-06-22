# Vexor — What's new

Short, public release notes for Vexor. Builds are rolling (early access), so the
dates below mark when each change reached the public RPM repo and Docker image.

## 2026-06-22

### Security & reliability
- **Tighter API access control:** administrators can now restrict which web
  origins are allowed to talk to the Vexor API (CORS), instead of accepting any
  origin. Sensible defaults ship out of the box.
- **Stronger data integrity:** the database now enforces relationships between
  hosts, services, checks, contacts and related records, so deleting or editing
  one no longer leaves orphaned or inconsistent data behind. Existing
  installations are upgraded automatically.
- **Login audit trail:** successful and failed sign-ins are now recorded in the
  audit log, making it easier to spot unauthorised access attempts.
- **Stability fixes across the web UI:** the host/service wizards, service
  discovery, job console, log dashboard and reports got internal robustness
  fixes that prevent stale data and unnecessary background refreshes.

## 2026-06-21

### Logs & alerting
- **More ready-made log filters for Microsoft SQL Server (2014 and later):** 9
  new one-click filters - high-severity engine errors, login failures,
  deadlocks, I/O/corruption (823/824/825), backup/restore failures and memory
  pressure, plus Always On Availability Groups failover/role change,
  not-synchronizing replicas and connectivity/lease loss.
- **More ready-made log filters for app servers:** 10 new one-click filters for
  Progress OpenEdge 11/12 and Apache Tomcat - PASOE agent/server errors, classic
  AppServer/WebSpeed broker failures, "agent/server died", AdminServer,
  NameServer and database `.lg` connection errors; plus Tomcat engine errors,
  startup/out-of-memory failures and access-log 5xx spikes.

### Agents & deployment
- **NRPE agent autostart fixed:** installing the NRPE agent (from the GUI or the
  one-liner) now reliably enables the service to start on boot, even if the very
  first start hiccups, and the installer prints a clear "enabled (autostart on
  boot)" / "running" status so you can see it worked.

### Access & roles
- **Read-only users can open the whole Logs section again:** viewer-role users
  (and the public demo account) can now reach every Logs page - dashboard,
  search/live-tail, filter library, alerts, shippers and log settings - so the
  full menu is navigable. Making changes (creating alerts, deploying shippers,
  editing settings) still requires operator/admin.

## 2026-06-20

### Security & maintenance
- Refreshed bundled dependencies across the platform to clear known security
  advisories — backend (Python) and web UI (JavaScript) packages are now on
  patched versions. No action needed; updates ship with the rolling build.
- Updated the bundled log database (VictoriaLogs) to 1.51.0.

### Logs & alerting
- **More ready-made log filters:** 20 new one-click filters for everyday Linux
  and Windows servers - disk full, filesystem/disk errors, kernel panic,
  hardware/MCE, failed services, MariaDB/MySQL errors, SELinux denials, fail2ban
  bans, time-sync loss, NFS stalls and root SSH logins on Linux; service/app
  crashes, unexpected shutdowns, disk/NTFS errors, account lockouts, Defender
  detections, failed logons and bugchecks on Windows.
- **Live-tail fixed:** the Logs search page can stream logs live again (the
  start button previously failed silently).
- **Delete saved searches** straight from the Logs page.
- **Windows log agent** now labels logs with the host name as Vexor knows it and
  always includes the message text, so Windows logs show under the right host
  and the Windows filters match. Re-run the installer to pick up the fix.
- **Deploy a log agent from the GUI:** push the Vector log shipper to a Linux
  host over SSH and watch the install stream live. Logs are now correctly
  labelled with the Vexor host name (so they show under the right host and feed
  per-host log checks), carry their real message text, and work against
  self-signed Vexor servers out of the box.
- **Log-based checks:** turn log data into monitoring. Get alerted when certain
  messages appear (errors, failed logins, …) or when a host stops sending logs
  entirely (dead-man switch). Each check is a full monitoring service, so it
  feeds straight into SLA reports, Business Service Monitoring and notifications.
- **Host “Logs” tab:** a host's recent logs, its log checks and a history of when
  each check moved between OK / WARNING / CRITICAL, all in one place. Add a
  "logs stopped" check straight from the Add Host wizard.
- **Syslog receiver:** point firewalls, switches and appliances at Vexor over
  syslog (UDP/TCP 514). Messages are parsed automatically and become searchable
  and alertable like any other logs. Off by default; enable it under Logs →
  Settings.
- **Configurable log retention:** set a global retention window with an optional
  disk-usage cap, and keep individual hosts for a shorter time using per-host
  overrides.
- **No-LogsQL log alerts:** a new “Simple” mode lets you build a log alert from
  plain fields (message contains / minimum level / which host, file or service)
  with a live preview — no query language required. “Advanced (LogsQL)” is still
  there for power users.
- **Ready-made log filters:** one-click starter filters for nginx, Apache/httpd,
  Caddy and Progress OpenEdge, on top of the existing system filters.

### Usability & fixes
- **Logs now show under the right host:** fixed two issues that made log
  views appear empty - the built-in shipper tagged logs as "unknown", and
  agents deployed to a host labelled logs with the box's own hostname instead
  of the host name as Vexor knows it. Re-deploy a log agent to pick up the fix.
- **Live install output:** deploying a log shipper or installing plugin
  dependencies now streams its progress in a console window (with a clear
  OK/FAIL result) instead of spinning silently.
- Fixed plugin dependency installs that could fail with a “sudo: no new
  privileges” error (e.g. when installing the Perl NaServer module).

## 2026-06-19

### Added
- **Add Host → Deep scan (nmap):** optional checkbox that scans the top 1000
  ports and lists any extra open ports as opt-in checks to monitor.
- **SNMP detection in Add Host:** hosts that answer SNMP are now flagged, and
  their SNMP checks (sysUptime, sysDescr, interfaces, UPS/printer status…) are
  offered as optional, un-ticked checks — never forced.
- **Business Service Monitoring (BSM):** roll several hosts/services up into a
  single business service with quorum logic and an SLA view.

### Fixed
- **Add Host:** pre-ticked default checks and discovery selections are now saved
  automatically — they could previously be lost before the final Save.
- **Add Host:** the Linux service scan now lists per-mount disks and running
  services, matching the Windows experience.

---
Earlier changes were part of the early-access preview (`v0.1.0-preview`).
