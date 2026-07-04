# Vexor — What's new

## 2026-07-04.3
- New **Agent deployment** page (under Monitoring) gathers everything about
  installing the agent in one place: the one-click Windows installer and Linux
  one-liner, ready-to-copy manual and mass-deployment commands (PowerShell,
  cmd and bash for PDQ/GPO/Ansible), direct downloads of every agent file
  (including Deploy-VexorAgent.ps1), add-on packages, cloud instance import and
  the firewall port reference. Cloud agents now live under this menu.
  (vexor-ui 0.1.0-132)

## 2026-07-04.2
- Microsoft SQL Server monitoring works out of the box again: Vexor now ships
  its own SQL Server health check, so the full set of MSSQL checks (connection
  time, active users, transactions, cache hit ratio, backup age, database free
  space, failed Agent jobs, long-running queries and more) run without any
  extra plugins or drivers to install. (vexor-api 0.1.0-236)
- Dashboards: you can now pick your own default dashboard. Click the star on a
  dashboard you created to make it the one Vexor opens on — the shared Vexor
  Overview is used only until you set your own. (vexor-ui 0.1.0-131)

## 2026-07-04.1
- More reliable Windows service discovery: the monitoring agent's response
  size was raised so Vexor now reads the host's full service list when you
  scan it, instead of stopping after the first ~30 services. Roles that used
  to be missed (databases, web servers and other services further down the
  alphabet) are now detected. Re-deploy the agent on existing hosts to get the
  larger response; detection already works either way.
  (vexor-api 0.1.0-235)

## 2026-07-03.22
- SQL Server discovery: when you scan or add a Windows host that runs
  Microsoft SQL Server, Vexor now reliably detects it - even on busy servers
  where the agent's service list was previously cut off - and offers SQL
  Server as a monitorable role. Tick the ready-made database checks (server
  online, backup age, blocked processes, connections, deadlocks, free space,
  transaction-log size) and enter a SQL login to enable them.
  (vexor-api 0.1.0-234, vexor-ui 0.1.0-130)

## 2026-07-03.21
- MSSQL checks fixed: SQL Server checks imported from op5 Monitor (backup job
  status, blocked processes, query counts, query response time and query
  regex checks) now work again. They previously failed with "plugin binary
  not found" because they relied on an old Perl SQL plugin that was never
  installed. Vexor now ships a native, dependency-free SQL Server check.
  (vexor-api 0.1.0-233)

## 2026-07-03.20
- New shared "Vexor Overview" dashboard: a built-in dashboard that always
  exists and that every user can switch to from the dashboard picker (under
  "Central"). It won't replace your own default dashboard - it's simply always
  available as a ready-made overview.
- Fixed "Problems" counting: the Services "Problems" filter and the top-bar
  problem badge now count only unhandled problems. Once you acknowledge an
  alert it no longer inflates the problem count, so the number always matches
  what the problems list actually shows.
  (vexor-ui 0.1.0-129, vexor-api 0.1.0-232)


## 2026-07-03.19
- Clearer navigation icons: every sidebar item now has its own distinct,
  meaningful icon, so you can recognise entries at a glance instead of seeing
  the same generic symbol repeated across many unrelated items.
- The Logs "Overview" entry is now called "Log overview" to avoid confusion
  with the main Overview section, and the "Problems" shortcut now highlights
  correctly in the menu.
  (vexor-ui 0.1.0-128)


## 2026-07-03.18
- Small navigation polish: menu items that open a sub-menu are now shown in
  bolder text, so they stand out from ordinary links and the grouped
  Configuration/Administration menus are easier to scan at a glance.
  (vexor-ui 0.1.0-127)


## 2026-07-03.17
- Navigation redesign: the left sidebar is reorganised into seven clear,
  role-based sections - Overview, Monitoring, Incidents & Response, Logs,
  Reports & SLA, Configuration and Administration. Related tools are grouped
  (e.g. Configuration and Administration now use tidy sub-menus), and you only
  see the sections your role can act on, so the menu is far less crowded.
- New global "+ Add" button in the top bar (add a host, run discovery, network
  discovery or import an op5 backup from anywhere) plus a one-click AI Assistant
  shortcut. Your personal settings - account, two-factor auth and API tokens -
  along with the product tour and Help & docs now live in the avatar menu.
  (vexor-ui 0.1.0-126)


## 2026-07-03.16
- Help -> "Install the agent": the Manual / PDQ section now has direct download
  links for Deploy-VexorAgent.ps1 and all support files (NSClient++ MSI,
  nsclient.ini, DH cert, helper plugins). Previously the page told you to use
  the script for mass/offline deploys but didn't say where to get it.
  (vexor-api 0.1.0-231)


## 2026-07-03.15
- Fix: the "Vexor Agent Self-Update" Scheduled Task failed to register on install
  because the NSClient++ scripts path contains a space ("Program Files"). The
  installer now uses a small wrapper .cmd so the task registers reliably. Re-run
  the installer once on already-installed agents. (vexor-api 0.1.0-230)


## 2026-07-03.14
- Fix: the one-file Windows installer didn't fetch the self-update helper, so the
  hourly "Vexor Agent Self-Update" Scheduled Task was never created. New installs
  now get it. If you already installed an agent, re-run the installer once to
  activate self-update. (vexor-api 0.1.0-229)


## 2026-07-03.13
- Agents now self-update their Vexor plugins. Once installed, each monitored host
  keeps its Vexor plugins in sync with the master automatically, so plugin fixes
  reach every agent without re-running the installer. On Windows an hourly Scheduled
  Task (LocalSystem) applies updates; on Linux a root systemd timer does. Two new
  optional checks report the state: "Agent plugin self-update (Windows)" and
  "(Linux)" — e.g. "up-to-date" or "updated N plugin(s)". Existing hosts pick it up
  the next time the installer/bootstrap is run.
  (vexor-api 0.1.0-228, vexor-naemon 0.1.0-65)


## 2026-07-03.12
- Uptime check is now useful: it shows how long the host has been up and its boot
  time (e.g. "OK: up 1w 2d 16:10, booted 2026-06-23 23:33"), instead of just "OK".
  It also accepts optional Warning/Critical filters (units m/h/d) so you can alert
  when a host rebooted recently, e.g. warning=uptime<2h, critical=uptime<1h.
  (vexor-naemon 0.1.0-64, vexor-api 0.1.0-227)


## 2026-07-03.11
- Windows Update check: fixed a false CRITICAL that appeared when no updates were
  pending. The combined status hid its real reason because a '|' separator was
  parsed as Nagios perfdata; it now uses ' / '. PendingFileRenameOperations is no
  longer treated as a servicing reboot by default (noisy/not update-related) —
  shown as a note, opt in with -IncludeFileRename 1. (vexor-api 0.1.0-226)


Short, public release notes for Vexor. Builds are rolling (early access), so the
dates below mark when each change reached the public RPM repo and Docker image.

## 2026-07-03.10
- **Windows disk check no longer lists nameless volumes.** The "all drives"
  disk check enumerated `drive=*`, which included EFI/recovery/system
  partitions without a drive letter (odd entries like `: Total: 96MB`). It now
  uses `all-drives`, so only lettered fixed drives (C:, D:, ...) are reported.
- **Edit service: Save now enables when you change a threshold.** Editing a
  check's argument (e.g. a critical value) no longer gets reset on the fly, so
  the Save button activates and your change can be saved.

## 2026-07-03.9
- **Windows agent checks fixed: "Could not complete SSL handshake".** The
  agent config (`nsclient.ini`) shipped a hard-coded `allowed hosts` list, so a
  freshly installed NSClient++ silently rejected NRPE queries from your Vexor
  server and every check failed with `CHECK_NRPE: (ssl_err != 5) Could not
  complete SSL handshake`. The installer now locks `allowed hosts` to the exact
  Vexor server the agent was installed from, so checks work immediately. Re-run
  the one-file installer on affected hosts to pick up the corrected config.

## 2026-07-03.8
- **Windows installer now works with self-signed certificates out of the box.**
  Most Vexor servers use a self-signed TLS certificate, so the one-file installer
  now trusts it by default. Downloads use curl (or a PowerShell fallback) that
  handle self-signed certs cleanly and without tripping antivirus heuristics, so
  you no longer have to obtain a public certificate just to roll out agents.

## 2026-07-03.7
- **Get started: clearer enrollment step.** Removed the confusing "Auto-enroll
  on/off" switch. You now simply create a token and the token you pick decides
  approval: **auto-approve** (hosts appear immediately) or **manual approval**
  (you approve them under Settings -> Agent Recipes). The Windows installer
  download unlocks as soon as a token exists.

## 2026-07-03.6
- **Fully English UI and installer.** Translated the last remaining Swedish
  text to English across the product: the Get started page, the sidebar entry,
  the Windows agent deployment scripts and the NSClient++ config comments.
  Vexor is now consistently English throughout.

## 2026-07-03.5
- **Windows installer now warns about antivirus up front.** The Get started
  page and install docs tell you before download that some antivirus / EDR /
  SmartScreen products may flag the downloaded PowerShell installer, so you can
  allow it across your environment before rolling it out.

## 2026-07-03.4
- **Windows one-file installer hardened.** The downloadable installer no
  longer trips Windows Defender (it was being flagged as a *ClickFix*
  downloader). It now uses standard, inspectable PowerShell and enforces a
  valid TLS certificate on your Vexor server instead of bypassing the check.

## 2026-07-03.3
- New **Get started** page (top of the menu) — go from zero to monitoring in
  minutes. Generate an enrollment token, then:
  - **Windows:** download a single, ready-made installer file. Right-click →
    *Run as administrator* and it fetches NSClient++, installs, opens the
    firewall and enrolls the machine automatically — nothing else to copy.
  - **Linux:** one copy-paste command with the token already baked in.
  - A clear **firewall port table (both directions)** so you know exactly what
    to open, plus a shortcut to **scan a host** and pick services to monitor.

## 2026-07-03.2
- New **ZeroSSL certificate monitoring**. Vexor can now watch *every*
  certificate in your ZeroSSL account through the ZeroSSL API and warn you
  before any of them expire — or if one already lapsed without being renewed.
  It looks at the newest certificate per domain, so a renewal clears the alert
  automatically. Add it from **Add service -> Certificates -> ZeroSSL account
  cert expiry** and set your warning/critical day thresholds. A step-by-step
  **help page** (get your API key, store it safely, test it, add the service)
  is included in the in-app Help section. Read-only — Vexor never changes your
  ZeroSSL account.

## 2026-07-03.1
- New **patch & update monitoring**, read-only, for both Windows and Linux:
  - **Windows Updates** — pending updates (with a security-only view), how many
    days since the machine last patched, whether it's **waiting for a reboot**
    after updates, failed update installs, the Windows Update service state, and
    an **end-of-support** warning that knows the exact Windows 10/11 and Server
    edition/build (including 24H2/25H2).
  - **Linux patch status** — pending (and security) updates, days since the last
    upgrade, reboot-required detection, failed package transactions, distro
    end-of-support, and whether automatic updates are configured. Works out of
    the box on RHEL-family (dnf/yum) and Debian/Ubuntu (apt); openSUSE, Arch and
    Alpine are best-effort.
- New **application dependency monitoring**. Point Vexor at a project (or scan a
  whole directory tree) and it reports **outdated dependencies** and known
  **security vulnerabilities** for **npm, pip and composer** projects — either
  per project or aggregated across the host. All of the above show up in the
  Add-service catalog under **Windows Updates**, **Linux Updates** and
  **App Dependencies**.

## 2026-06-28.2
- More **built-in setup help** in the Add-service dialog: the rest of the
  PLANit checks (status, running, running with staleness, BusySonic, REQMIN,
  versions, path size, path size with filter) and the generic mountpoint
  check now show short setup notes and pre-filled argument hints, matching
  the AppServer agents check.

## 2026-06-28.1
- The **PLANit AppServer agents** check (check_nrpe_pladm_agents) now shows
  built-in setup help when you add it to a host: which files to deploy, the
  one permission tweak the NRPE user needs on the check script, and how to
  pick the right system name (production vs. training). The System, Warning
  and Critical fields also get inline hints.

## 2026-06-26.1
- Fix the recurring **blank/white page after an update**. The web server kept
  letting browsers cache the app shell (index.html), so after a new version was
  deployed the cached page still pointed at script files that had been replaced
  — they failed to load and the page stayed blank. index.html is now always
  revalidated, while the versioned asset files are cached for a year. If you
  still see a blank page once, do a single hard refresh (Ctrl/Cmd+Shift+R);
  after that it self-corrects on every future update.

## 2026-06-25.25
- Distributed pollers now run **service checks**, not just host checks.
  Previously only host (ping) checks were expanded for the poller, so
  pinned hosts showed no service results. The poller now receives fully
  expanded, ready-to-run service commands.
- Credentialed checks (agentless WMI/SSH and anything using master-local
  key/payload files) are detected and **skipped on pollers** — they can't
  run remotely. The host detail page warns you which checks are skipped on
  a poller-pinned host, so you can run them from the master or use an
  on-host agent instead.
- Fix: poller service check intervals were interpreted in the wrong unit
  (treated minutes as seconds). Each check now runs on its correct schedule.
- Reliability: result push to Naemon is now non-blocking (a stalled Naemon
  can no longer wedge the API), and a poller's status now reflects how many
  results were actually injected.
- Fix: nightly orphan-poller cleanup now returns affected hosts to active
  checking instead of leaving them stuck as stale/passive.
- Stale-detection windows for poller-pinned checks now scale with the check
  interval instead of a flat 5 minutes.

## 2026-06-25.24
- Fix: the Settings -> System page froze the entire UI (infinite render
  loop, React #185). After opening it, navigating to other menus changed
  the URL but the page never loaded. Root cause: Mantine useForm objects
  in useEffect dependency arrays. Removed them from the deps.

## 2026-06-25.23
- Fix: certificate/service notes now actually appear on the host detail
  page. The host list endpoint that feeds that page was not returning
  per-service notes (only the single-host endpoint was), so the note
  stayed hidden on the host even though it showed on the Certificates page.

## 2026-06-25.22
- Agent install / downloads help pages now pre-fill the URLs with this
  Vexor server's own address instead of the YOUR-VEXOR-SERVER placeholder.
- The Linux agent one-liner now uses curl -k, so a self-signed Vexor
  certificate (the common case) works without extra flags.

## 2026-06-25.21
- Service/cert notes are now visible on the host detail page (shown under
  each service) and editable from the Edit service modal. Previously a note
  added to a certificate check only appeared on the Certificates page.
- API: get_host and service detail now return per-service notes; PATCH
  /services/{id} accepts notes.

## 2026-06-25.20

**Certificates: add a note to each cert check**

When you add a certificate check on the **Certificates** page you can now fill in an optional **Note** - for example the customer environment name or any other context. Notes show in a new column in the cert list and can be edited inline (click the note to change it). The note is also written to the underlying monitoring service, so it is available wherever service notes are shown.

## 2026-06-25.19

**One-liner to install + auto-enroll the Linux agent**

You can now install the Vexor agent on a Linux host and have it register itself with a single command, just like the Windows deploy script. Generate an enrollment token under **Settings -> Agents**, then run:

```
curl -fsSL https://YOUR-VEXOR-SERVER/api/v1/nrpe/bootstrap.sh | sudo VEXOR_TOKEN=YOUR-TOKEN bash
```

It installs nrpe + the standard Nagios plugins + Vexor's helper plugins, whitelists your Vexor server, starts nrpe, and enrolls the host (auto-approve or pending, depending on the token). The installer is idempotent and non-destructive: re-running it on a host that already has an agent just refreshes Vexor's command set and whitelist - it never uninstalls your nrpe.

A new **Help -> "Install the agent (Linux & Windows)"** page documents the one-liners and options.

## 2026-06-25.18

**Fixed: adding a check returned a server error**

Adding a check to a host could fail with a 500 error (a runbook-URL field wasn't mapped on the host/service model). Fixed, and the optional runbook URL now saves correctly.

**Fixed: Re-scan host (NRPE introspection) found nothing**

Re-scanning a Windows host could finish instantly with "nothing new" when the agent's `nsclient.ini` didn't expose the version query. Detection now also uses the standard `check_cpu` / `check_drivesize` commands, so drives and checks are discovered reliably. Suggested per-drive checks now use the detailed **Disk C:** / **Disk D:** format (Total / Used / Free), and the redundant "Agent version" suggestion was removed.

## 2026-06-25.17

**Auto-enrolled Windows hosts get one service per drive (op5-style)**

Instead of a single combined disk service, auto-enrollment now **discovers the host's fixed drives** and creates one clean **"Disk C:"**, **"Disk D:"**, ... service per drive, each reporting Total / Used (%) / Free (%) (warn at free < 20%, critical at free < 10%). Tiny letterless system/EFI/recovery partitions are skipped. If the agent isn't reachable during enrollment, a single combined all-drives check is used as a fallback. Already-enrolled hosts keep their current checks (delete + re-add to pick up per-drive services).

**Fixed: adding a multi-argument check could crash the UI**

Adding a check such as `check_nrpe_disk` from a host page could throw a blank-screen error (React #185, infinite render loop). Fixed.

## 2026-06-25.16

**Auto-enrolled Windows hosts now monitor all fixed drives**

The default disk check for auto-enrolled Windows agents now covers **every fixed drive** in the machine (C:, D:, ...), not just C:. It uses a new `check_nrpe_disk_all` command (NSClient++ `check_drivesize` filtered to `type = 'fixed'`, so optical/removable drives are skipped) and reports per-drive Total / Used (%) / Free (%) with warning at free < 20% and critical at free < 10%. Already-enrolled hosts keep their current checks.

## 2026-06-25.15

**Better default checks for auto-enrolled Windows hosts**

The default check set applied to auto-enrolled Windows agents has been tweaked: the **Agent version** check is no longer added by default, and the disk check now uses a detailed drive check that reports **Total / Used (%) / Free (%)** per drive (thresholds: warn when free < 20%, critical when free < 10%) instead of the terse "all drives are ok" line. Already-enrolled hosts keep their existing checks; this affects newly enrolled machines.

## 2026-06-25.14

**Audit log: filter on agent enrollment**

The Audit log (Reports -> Audit) action filter now lists the agent self-registration actions (`agent.enroll.auto`, `.pending`, `.exists`, `.denied`, `.approved`, `.rejected`) with colour-coded badges, so you can filter straight to enrollment events while a deploy runs.

## 2026-06-25.13

**Follow agent auto-enrollment in the Audit log**

Every time an agent self-registers, Vexor now records an entry in the **Audit log** (Settings/Audit, or `GET /api/v1/audit`) with the client IP and outcome: `agent.enroll.auto` (host created), `agent.enroll.pending` (queued for approval), `agent.enroll.exists` (already monitored, no-op) and `agent.enroll.denied` (wrong token). Approving or rejecting a queued host is logged too (`agent.enroll.approved` / `agent.enroll.rejected`). Filter the Audit log by action `agent.enroll` to watch new machines roll in as your deploy script runs.

## 2026-06-25.12

**Auto-enrollment: choose auto-approve vs. pending per machine**

Zero-touch enrollment now supports **two tokens**. In **Settings -> Agent Recipes** an admin can generate an **auto-approve token** (hosts are created and monitored immediately) and a separate **pending token** (agents wait in an approval queue). Each token has its own badge and can be rotated or cleared independently. Agents enrolled with the pending token appear in a **Pending approval** table on the same page where an admin can **Approve** (create the host with default checks) or **Reject** them. Put whichever token you want into a given machine group's deploy script — the auto-approve token remains the simple default.

## 2026-06-25.11

**Zero-touch agent auto-enrollment**

Freshly deployed agents can now register themselves automatically. In **Settings -> Agent Recipes** an admin enables **Auto-enrollment**, picks the default OS (Windows/NSClient++ or Linux/NRPE) and generates a single **shared enrollment token** (shown once; rotate or clear it any time). Drop that token into your agent deploy script and each machine calls `POST /api/v1/agents/enroll` on install. Vexor creates the host automatically with a sensible set of default agent checks (CPU, memory, disk, uptime, agent version) and reloads monitoring — no manual host registration, auto-approve. Re-running the deploy script on an already-enrolled host is a safe no-op.

## 2026-06-25.10

**Fix: self-monitoring checks failed with 'plugin not found' on fresh installs / the demo**

The 10 self-monitoring check plugins (systemd units, MariaDB/Postgres, NTP, memory, Naemon livestatus, API health, license, log ingest) are now shipped inside the vexor-api package under `/opt/vexor/plugins/self-monitor/`. Previously they were only written at runtime by the self-monitor installer, so any host where that step was skipped (for example the all-in-one demo container, which has no bootstrap token at build time) showed every self-check as `execvp(... ) failed errno 2: No such file or directory`. The installer still re-writes them idempotently for self-heal and upgrades.

## 2026-06-25.9

**AI settings get their own page; AI Assistant is now just a consumer**

All AI provider configuration now lives on a dedicated admin page at **Settings -> System -> AI** (internal Ollama and external cloud providers, model management, automation rules and audit log) instead of being embedded in the System settings page. Settings are system-wide and shared by every user.

The **AI Assistant** page is now a pure consumer: it shows the active provider/model, lets you pick from the models that are already configured, and runs the chat — with no setup controls. This also removes the earlier issue where opening the System settings page could leave the menu unresponsive.

## 2026-06-25.8

**External AI moved into System settings + AI log analysis**

The external-AI / LLM provider configuration now lives under **Settings -> System -> External AI**. Whatever an admin saves there (provider, API key, base URL, model) is **system-wide** and shared by every user; the API key is stored encrypted and never shown again. Local Ollama model management stays on the AI Assistant page.

New on the **Logs Dashboard**: an **Analyze with AI** button runs an SRE-style triage of your current log query (summary, notable events, likely root cause, recommended next steps) using the configured AI provider. Available to operators and admins.

## 2026-06-25.7

**Backups now include a fresh, consistent Keycloak dump**

When you create a backup, Vexor now triggers an on-demand PostgreSQL dump of the
Keycloak (auth) database right before archiving, instead of bundling whatever
nightly dump happened to exist (which could be up to 24h old). The backup manifest
marks the Keycloak dump as fresh, stale, or missing so you always know what you
restored. The dump runs through the privileged vexor-jobd helper, so it works even
though the API itself runs unprivileged.

## 2026-06-25.6

**Log alert notifications fixed**
- Log alert rules that are **not** bound to a host now correctly deliver their notifications. These direct notifications were previously sent to an outdated internal address and failed silently; they now flow through the normal notification pipeline (email, SMS, ntfy, webhooks, …) like every other alert.
- Host-bound log rules were unaffected — they notify through the monitoring engine — and are now also protected from duplicate alerts.

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
