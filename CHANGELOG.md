# Vexor — What's new

Short, public release notes for Vexor. Builds are rolling (early access), so the
dates below mark when each change reached the public RPM repo and Docker image.

## 2026-06-25.5

**Use your own cloud AI provider**
- AI features (incident root-cause, summaries, suggestions) can now run on a **cloud LLM of your choice** instead of — or alongside — local Ollama. Pick a provider in **AI settings** and paste your own API key:
  - OpenAI (ChatGPT), Anthropic (Claude), Google Gemini, Azure OpenAI, any OpenAI-compatible endpoint (OpenRouter, Groq, Mistral, vLLM…), and GitHub Models.
- Your API keys are **encrypted at rest** and never shown back in the UI. Each provider keeps its own key, base URL and model, so you can switch between them freely.
- A **Test connection** button validates your key and model in one click. Local Ollama remains the default and works exactly as before.

## 2026-06-25.4

**Pull external metrics into Vexor**
- New **check_prometheus** check (category *Metrics*): scrape any external Prometheus / OpenMetrics endpoint — or a JSON API — and turn a chosen metric into a normal Vexor check with warning/critical thresholds, perfdata graphs, SLA and alerting. Supports label filters, aggregation (sum/max/min/avg), bearer-token headers, self-signed TLS, and a JSON value path.

**AI on incidents**
- The **Incident timeline** now has an **AI root-cause analysis** button that summarizes the merged timeline (state changes, notifications, acknowledgements, comments) and suggests likely causes and next actions.

## 2026-06-25.3

**Snooze & mute alerts**
- New **Mute Rules** (Notifications settings): silence notifications for matching hosts/services using glob patterns, with an optional severity ceiling, a time window or indefinite, and a reason. Monitoring keeps running and problems still show in the UI — only the notifications are muted.
- New **Snooze** quick action on a problem: mute its notifications for 1h, 4h or 24h in one click.
- Mute is applied before all other suppression rules (quiet hours, rate limits, storm control).

## 2026-06-25.2

**Virtualization monitoring — easier setup**
- Virtualization checks are now clearly labelled by layer: **[Hypervisor]**, **[Virtual machines]** and **[Storage]**, so it's obvious what each check covers.
- The Add-check dialog now shows a step-by-step **"How to create the read-only user"** guide for Proxmox VE (PVEAuditor token), VMware vCenter (Read-only role) and XCP-ng (read-only RBAC). Vexor only ever reads — it never changes your virtualization platform.
- New one-click **bundle templates** for Proxmox VE, VMware vCenter and XCP-ng pools that add ping + cluster/host health + VM states + storage checks in one go.

## 2026-06-25 (later)

### New features
- **Auto-remediation with event handlers:** define handler commands (for example
  a service-restart script) and attach them to any service. When that service
  changes state, Naemon now runs the handler automatically. Previously handlers
  could be configured but were never applied to the monitoring core - they are
  now written and reloaded, with the reload result surfaced in the UI. Manage
  them under Settings -> Event Handlers.

## 2026-06-25

### New features
- **Better certificate monitoring:** the Certificates page now reliably lists
  every cert check you have (HTTPS, check_ssl_cert, raw TLS and on-host cert
  files), enriched with the issuer, number of SANs and the exact expiry date,
  sorted by what expires soonest.
- **Certificate expiry notifications:** opt in to a daily *expiry digest* that
  alerts you through your notification channels when any monitored certificate
  is within a configurable number of days of expiring. A *Send test digest*
  button lets you confirm delivery right away.

## 2026-06-24 (later)

### New features
- **TrueNAS / FreeNAS storage monitoring (agentless, read-only):** keep an eye
  on your TrueNAS CORE/SCALE and FreeNAS storage via the native REST API using a
  read-only API key. Checks cover ZFS pool health, capacity, disks and SMART,
  vdev/RAID redundancy, scrub/resilver status and system alerts - Vexor only ever
  reads, never writes.

### Fixes
- **Problem counter now matches the list:** the problem count in the top bar (and
  the notification bell) no longer includes problems you have already acknowledged,
  so it stays in sync with the Problems list.

## 2026-06-24

### New features
- **Virtualization monitoring (agentless, read-only):** monitor **Proxmox VE**,
  **VMware vCenter/ESXi** and **XCP-ng/XenServer** through their native APIs
  using a dedicated read-only account (Proxmox `PVEAuditor`, vSphere *Read-only*
  role, XCP-ng read-only RBAC). Track cluster/host health, VM power state,
  datastore/storage-repository usage and capacity - Vexor never modifies anything
  in your hypervisors.
- **Prometheus metrics endpoint:** host and service state plus performance data
  are now exposed in Prometheus exposition format at `/api/v1/metrics/monitoring`,
  ready to scrape into Prometheus/Grafana or any compatible TSDB.
- **Monitoring coverage score:** a new per-host *Monitoring Score* (graded A-F)
  highlights gaps in your coverage - missing standard checks, no notifications,
  no thresholds - so you know where to harden your monitoring.

## 2026-06-23 (later)

### New features
- **Chat & on-call notifications:** Slack, Microsoft Teams, Discord and
  Telegram are now first-class notification channels with rich, native
  formatting (colour-coded by severity), alongside **PagerDuty** (Events
  API v2) and **Opsgenie** — these open an alert on a problem and resolve
  it automatically on recovery.
- **Acknowledge from the message:** problem notifications now include a
  one-click *Acknowledge* link, so you can ack an alert straight from a
  Slack/Teams/Discord/Telegram message or e-mail without logging in.

## 2026-06-23

### New features
- **NOC wallboard:** a new fullscreen, distraction-free status board
  (open *NOC Wallboard* in the sidebar, or visit `/wallboard`) that
  auto-refreshes — ideal for a wall-mounted screen in an operations room.
- **Notification center:** a bell in the top bar shows a live count of
  unhandled problems and lists them with one-click links to the affected
  host or service.
- **Saved views:** save your favourite filters on the Hosts, Services and
  Events pages and switch between them in a click — each user keeps their own.
- **Runbook links:** hosts and services can now carry a *Runbook / docs URL*.
  It appears as a one-click link on the detail pages and is passed through to
  the monitoring engine, so the runbook is always one hop away from an alert.
- **On-call schedules:** define daily or weekly rotations of contacts. A
  notification policy can opt in to a schedule so the person currently on call
  is automatically added to the recipients.
- **Status-page subscriptions:** visitors to your public status page can
  subscribe (with email confirmation) to be notified about incidents.
  Sending is off by default and enabled from *Notification settings*.
- **Alert storm control:** an optional safeguard that groups and suppresses
  repeated alerts from the same host during a storm, so a single flapping
  host can't flood your inbox. Off by default.
- **Guided product tour:** a quick in-app walkthrough of the main areas,
  available any time from *Product Tour* in the sidebar.


### Easier to use
- **System Health page:** a new *Settings → System Health* page shows live,
  at-a-glance status of the core building blocks — database, monitoring
  engine, command pipe, live status feed, single sign-on and disk space — so
  you can confirm everything is healthy without digging through logs.
- **Better on mobile:** the navigation menu now closes automatically after you
  pick a page, so it no longer covers the content on phones and tablets.
- **In-app help:** contextual help icons next to page titles explain key
  concepts (for example what SLA availability and services mean).
- **Test your notifications with a custom message:** the per-channel test
  dialog now lets you type your own message before sending.
- **Friendlier empty pages:** the Acknowledgements and Downtimes pages now show
  a clear explanation when there is nothing to display yet.
- **Clearer public status page:** the public status page now shows an
  operational / degraded / outage summary at the top.

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
