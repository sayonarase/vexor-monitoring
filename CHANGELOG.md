# Vexor — What's new

Short, public release notes for Vexor. Builds are rolling (early access), so the
dates below mark when each change reached the public RPM repo and Docker image.

## 2026-06-20

### Security & maintenance
- Refreshed bundled dependencies across the platform to clear known security
  advisories — backend (Python) and web UI (JavaScript) packages are now on
  patched versions. No action needed; updates ship with the rolling build.
- Updated the bundled log database (VictoriaLogs) to 1.51.0.

### Logs & alerting
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
