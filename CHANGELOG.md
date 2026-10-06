# Vexor — What's new

## 2026.10.06.2

### Security

**Security updates of bundled libraries.**

Several libraries bundled with the Vexor server and web interface have been
updated to close published security advisories:

- PyJWT 2.15.1, which checks sign-in tokens. This closes one critical and
  several high-severity advisories about how tokens are validated.
- urllib3 2.8.0, Mako 1.4.3, multidict 6.9.1 and Werkzeug 3.1.9.
- DOMPurify 3.4.16 in the web interface, which cleans formatted text before it
  is shown.

No settings change. Install the update as usual with `dnf update`.

## 2026.10.06.1

### Security

**Sign-in (Keycloak) updated to 26.8.0.**

The sign-in service bundled with Vexor has been updated from Keycloak 26.7.3 to
26.8.0. This closes the security issues fixed in Keycloak 26.7.4, 26.7.5 and
26.8.0. Among other things, a disabled client application can no longer appear
in the tokens Keycloak issues.

Existing users, groups, roles and LDAP settings are carried over automatically
the first time the new version starts. You do not need to do anything beyond
the normal update. If brute-force protection is turned on, failed sign-in
attempts are now kept in the database, so an account locked after repeated wrong
passwords stays locked when the server is restarted.

**Log storage (VictoriaLogs) updated to 1.53.0.**

The log database has been updated to VictoriaLogs 1.53.0. That version only
accepts deletions sent as a POST request. Vexor's per-host log retention used
an older form of the request and would have stopped removing old log entries
without any warning. It has been changed to work with the new version, so
retention keeps working after the update.

## 2026.09.04.3

### Fixed

**Log shipping did not work on a new installation.**

The address that log shippers send to, `/api/v1/logs/push`, was missing from the
web server configuration Vexor installs. Anything sent to it was rejected as an
unknown address, so a newly installed server collected no logs from the machines
it monitors. The address is now part of the installation, and log shipping works
as soon as Vexor is set up.

### Improved

**The log ingest token can now be managed from Vexor itself.**

Log shippers authenticate with a token. Until now that token could only be
created or replaced by editing two files on the Vexor server over SSH, and
getting either of them wrong left log collection broken.

Under Logs, Vexor now shows whether a token is configured, lets an administrator
reveal and copy it, and can generate or replace it directly. Before replacing
one, Vexor lists the machines that are currently sending logs, since each of
them stops being accepted until it is given the new token; you are asked to
confirm in writing before it goes ahead. The change is verified before it is
applied, and reverted automatically if anything is wrong with it, so a mistake
here cannot take the web interface down.

**Background work is picked up straight after an update.**

Certain tasks started from the web interface are carried out by a separate
component that was not restarted when Vexor was updated. It therefore kept
running the previous version and refused work that the newly installed version
knew how to do, until the server was restarted. It is now restarted as part of
the update.


## 2026.09.04.2

### Fixed

**Setup reported the wrong number of self-monitoring checks.**

After installing, setup said it had created 29 checks on the `vexor-self` host
when it had actually created 30. The number was written into the message by
hand and was never updated when a new check was added. The installer now
reports the number it actually created, so the two cannot drift apart again.

### Improved

**Setup now tells you up front if the server is too small.**

A machine with too little memory or disk space used to fail part-way through
the installation with nothing to explain why. A Vexor server runs its database,
metrics store, login service, monitoring core and web API alongside each other,
so it needs roughly 4 GB of memory and 20 GB of free disk space; 8 GB of memory
is comfortable.

Setup now points this out before it starts working. It is only a warning and
never stops the installation, because virtual machines that size their memory
on demand report a small figure while idle and grow once the work begins. The
requirements are now written down in the installation guide as well.


## 2026.09.04.1

### Fixed

**A new installation came up with monitoring switched off.**

Installing Vexor on a clean machine left its monitoring core unable to start. The
command definitions Vexor ships were written into your own definitions file
without checking whether the core already provided them, and six of them were
duplicated as a result. The monitoring core refuses to start while a command is
defined twice, so a brand new installation had no scheduler, no checks running,
and no self-monitoring — while the installer otherwise reported success.

Installing now writes only the definitions that are genuinely missing. If you
installed Vexor recently and it has never run any checks, upgrading removes the
duplicates and the core starts. Your own definitions are untouched: the ones
removed have never been able to load.

**Setup stopped without saying anything when run from a script.**

Running setup anywhere without a terminal attached — over ssh without a
terminal, from an automation tool, or from a first-boot script — stopped at the
first question and exited with no output at all. Setup now recognises this and
continues with generated passwords, saving them where it always does.

**Pressing Enter at a password question set an empty password.**

Setup offers to generate each password for you, but accepting that offer with
Enter cleared it instead. Depending on the question this either stopped the
install partway through or left the administrator accounts with no password.
Enter now accepts the generated password, as it always said it would.

**Vexor was not watching whether its own alerts could still be delivered.**

Vexor includes a check that sends a probe through the whole alerting path, so a
system that has quietly stopped alerting is not mistaken for one with nothing to
report. It was included with Vexor but never actually scheduled, so it was not
running anywhere. It is now part of self-monitoring on new installations, and
re-running setup adds it to an existing one.

## 2026.09.03.13

### Fixed

**The ownership repair announced in the previous release did not actually reach
your installation.**

The previous release said that upgrading would repair the alert log file's
ownership. The repair was written, tested, and correct - but it was placed in a
packaging file that no build reads, so it was never included in the package.
Nothing about it was visible: the release completed normally and the change looked
shipped. It was working on our own server only because we had also applied it
there by hand.

The repair now lives in the file the build actually uses, and we verified it is
present in the package itself rather than only in the source. If you upgraded for
the previous release and alerting is still not delivering, this upgrade is the one
that fixes it.

**New checks were never added to installations that already existed.**

Checks we add to Vexor arrive as definitions the monitoring core has to know about.
Those definitions were only written when Vexor was installed for the first time, so
an existing installation would upgrade, receive the new check's program, and still
have no way to run it. Upgrading now adds any definitions your installation is
missing. Definitions you have edited yourself are left exactly as they are, and if
adding one would stop the monitoring core from starting, the change is undone
before that can happen.

### Added

**Vexor now proves its own alerting works, rather than assuming it does.**

Vexor already tested its notification channels, but that test started inside the
part of Vexor that sends notifications - so it could not see a fault in the stretch
between the monitoring core and that point. That stretch is precisely where
alerting had been broken for three months without anyone noticing.

A new self-check called "Notification path" now travels the whole route: it runs
the same script the monitoring core runs, under the same account, and then asks
Vexor whether the alert arrived. It cannot be fooled by a quiet-hours schedule or a
rate limit, and it never contacts anyone - it is not a test alert, so nobody's
phone rings. If it goes quiet, Vexor's external dead-man switch reports the system
as unhealthy, which is the one signal that still works when alerting itself is what
has failed.

The check appears automatically on the self-monitoring host. If it is not there,
add "Notification path" to that host and activate the change.

## 2026.09.03.12

### Fixed

**Alerts raised by the monitoring core were being discarded before they were sent.**

This is the most serious fault we have found in Vexor, and it deserves a plain
explanation.

When a check goes critical, the monitoring core hands the alert to a small script
that forwards it to Vexor's notification engine, which then decides who to tell and
how. That script wrote a line to its own log file and sent the alert in what was,
by a mistake in how it was written, a single indivisible step. If the log file
could not be written, the send did not merely go unlogged - it never happened at
all. The script then reported success, so nothing further up ever saw a problem.

Whether the log could be written came down to which account happened to create the
file first. If anything ever ran the script as an administrator - during setup, or
while testing an alert - the file ended up owned by that account, and the service
account that does the real work could no longer append to it. From that moment on,
every alert the monitoring core raised was silently thrown away.

There was no visible symptom. Checks still turned red in the interface, dashboards
were correct, SLA figures were correct, and the notification settings all looked
right. Only the delivery was missing. On our own server this had been the case
since June.

Sending and logging are now separate steps: the alert goes out first, and the log
entry is written afterwards on a best-effort basis. A delivery that fails is
recorded in the system log even when the file cannot be written, so this can never
again be invisible. Upgrading also repairs the file ownership.

**Please check that alerting works on your own installation after upgrading.** In
the interface, open Notifications, use the test-send function, and confirm the
delivery appears in the notification log. If you have been wondering why an alert
never arrived, this is very likely why.

### Added

**A check that tells you when a build pipeline has stopped guarding you.**

Aimed at teams whose release process depends on automated checks. A build pipeline
fails in two ways that are easy to miss: it starts failing and stays that way until
people stop reading it, or it quietly stops running and everything looks fine
because nothing is red.

The new check watches both. A fresh failure is treated as normal - someone pushed
something that did not work - and it only raises an alert once a pipeline has
stayed broken longer than you would expect a fix to take, with the elapsed time
measured from when it first broke rather than when it last ran. It also reports a
pipeline that has not run at all for a long time.

Add it like any other check and point it at your repositories; it needs a
read-only access token and only ever reads. We wrote it after finding one of our
own blocking checks had been failing for eighteen days without anyone noticing.

## 2026.09.03.11

### Fixed

**Deploy configuration was being left behind on the Vexor server.**

When you push the monitoring agent to a Windows machine from the Vexor interface, Vexor
writes two short-lived files to its own temporary directory: the deploy configuration and
the agent's ini file. They are meant to be deleted the moment the push finishes.

They usually were. But the deletion was the last thing the push did rather than something
that always happened, so any push that did not reach the end left its files behind
permanently. The most common cause was ordinary maintenance: upgrading Vexor restarts the
service, and every push still running at that moment was interrupted. Nothing ever came
back to clean up.

This matters because of what those files contain. Between them they name the machines you
were deploying to, the administrative account used to reach them, and - if automatic
registration is enabled - the enrollment token. The files were only ever readable by the
Vexor service account, so this was not exposed to other users of the server, but material
of that kind should not sit on disk indefinitely, and there was no upper bound on how long
it did.

Two changes: cleanup now runs whether the push succeeds, fails, or is interrupted, and each
push first clears any leftovers from previous ones. The second part matters because it also
clears whatever has already accumulated on your server - you do not need to go looking. The
sweep only touches Vexor's own push files, only ones older than a day, and leaves anything
that could still belong to a running deployment alone.

If you would like to confirm your own server is clear, the files are named
`vexor-winpush-*.json` and `vexor-ini-*.ini` in `/tmp`. After upgrading, the next agent push
removes any that remain.

## 2026.09.03.10 — Everything Vexor is built from, brought up to date

Vexor is assembled from a large number of open-source components. Sixty-five of
them had drifted behind their current releases, some by a long way. All of them
are now current, and a security scan of the result reports nothing outstanding.

None of this changes how Vexor looks or behaves. It matters because a component
that is left behind eventually becomes one nobody can update safely, and because
fixes published upstream — including security fixes — only reach you once we take
them.

Three components moved across a major version boundary, which is where behaviour
is allowed to change. Rather than assume the test suite would notice, we
exercised each one against the part of Vexor that depends on it: PDF reports were
generated and opened, including one with an embedded chart, and the graph images
those reports contain were rendered and checked.

Along the way we found a genuine fault in one of our own safety checks, and it
was the quiet kind. Vexor has a test whose job is to walk every part of its web
interface that can change something and confirm none of it can be reached without
logging in. The newer version of one component changed how that list of endpoints
is presented internally, and our test could no longer read it. It did not report
an error. It found nothing to check, checked nothing, and passed — a security
guard reporting success while looking at an empty room.

The check now reads the same list from a stable, published description of the
interface, and it refuses to pass if that list ever comes back suspiciously
short. We confirmed no endpoint escapes it. To be clear about scope: the login
requirement itself was never affected and no endpoint was ever left unprotected.
What had stopped working was the thing that proves it, which is precisely what
you want to hear about rather than discover later.

We also removed a leftover file from our build system. It was an old list of
component versions that nothing used any more, but it looked current enough that
automated security scanning treated it as real and raised thirty-seven warnings
against it, one of them critical. Every one was false — the versions that matter
are recorded elsewhere and were already up to date. Thirty-seven false warnings
is worse than none, because the one that is real stops standing out.

## 2026.09.03.9 — Anomaly baseline checks now return a result

A baseline check compares a metric against its own recent history and alerts when
it drifts outside the normal range. Every one of these checks has been failing
since the feature was introduced. Instead of a result you got a red alert
containing a shell error message, on every host that used one.

The cause was a single character. Our monitoring engine treats a semicolon in its
configuration as the start of a comment, and this was the only check whose
command contained one. The engine kept only the text before it, cutting the
command in half, and what was left could not run.

Two more faults were hidden behind that one and would have appeared as soon as it
was fixed, so all three are corrected together. The checks now return a proper
result with a value, a baseline and performance data for graphing.

Nothing needs to be reconfigured. Existing baseline checks start working after
the update, and the history they compare against was being collected all along.

## 2026.09.03.8 — The alternative log collector now works

Vexor can collect logs with either of two programs. Vector is the default and is
what you get unless you choose otherwise. The alternative is Fluent Bit, which
you install by hand if you prefer it.

Fluent Bit has never worked. Following our own installation guide — install the
package, start the service — failed, and it failed in four separate places, each
one hidden behind the one before it. The service pointed at the wrong location
for the program itself, so it could not start. Once that was corrected it sent
logs to an address our log store does not answer on, so everything it sent was
refused. Once that was corrected the log text arrived in a field the log store
does not read, so every line would have appeared in the log view as a placeholder
instead of the message. And the host name attached to each line was blank.

All of it is fixed and the whole path has been tested from end to end: the
service starts, both plain log files and the system journal are collected, and
lines arrive with the right host name and their actual text.

If you use Vector, which is almost certainly the case, nothing changes for you.

The package's version number also changes from 5.0.5 to 1.0.0. It looked like it
was tracking Fluent Bit's own version, and it was not: this package contains only
configuration, never the Fluent Bit program, which comes from your Linux
distribution and is updated along with the rest of it. The number now describes
what the package actually is. Existing installations still update normally.

## 2026.09.03.7 — Log collector updated, and we now watch our own ingredients

Vector, the component that collects log files from this server and forwards them
to the log store, has been updated from 0.57.0 to 0.58.0.

The version we shipped needed a setting switched on that Vector itself calls
dangerous. It was not dangerous in our case: the setting relaxes a check on
where values in the configuration are allowed to come from, and the only values
we use it for are the host name and the service name that label your log lines.
Vector's check was flagging a case it should not have. Upstream has now fixed
the check, so the setting is gone. If you had edited your log collector
configuration yourself, the update removes the setting from your copy too, but
only after confirming that your configuration still works without it — if yours
genuinely needs it, it is left alone and you are told so.

Alongside that: Vexor bundles a handful of pieces of software that we did not
write — the component that handles signing in, the monitoring engine, the log
collectors and the log store — and we choose which version of each one ships.
Until now nothing watched those projects for new releases, and we found that out
the hard way with the security update in the previous entry. From now on we
check every one of them against its upstream project every morning, so a
security fix cannot sit unnoticed again.

## 2026.09.03.6 — Security update for the sign-in component

Keycloak, the component that handles signing in, has been updated from 26.7.1 to
26.7.3. The two releases in between fix 27 security issues. Three of them matter
directly to a monitoring platform that sits behind a login page: one let an
attacker take over an account through the "forgot my password" flow without ever
signing in, one let an attacker take over an account by guessing a predictable
value used when linking accounts, and one meant that when Vexor authenticated
users against your LDAP or Active Directory server, it did not check that the
certificate it was offered actually belonged to that server.

None of these required any change on your side, and we have no indication that
any of them were used against a Vexor installation. Updating is still the right
thing to do, and the update applies as soon as you install it.

We also fixed a mistake of our own. When we build the Keycloak package, we copy
the component's directory from our build server. That directory contains a
configuration file, and the copy included our own build server's database
password — which meant it was sitting in a file inside a package anyone could
download. Your installation was never at risk: Vexor generates a database
password unique to your server during installation and writes it over that file
every time the package is installed or updated, so no installation has ever used
the password that was in the package. The packaged file no longer contains a
password at all, and we have changed the one on our build server.

## 2026.09.03.5 — Signing in still works after a restart

Keycloak, the component that handles signing in, stores its accounts in a
PostgreSQL database. Its startup instructions told the system only to wait for
the network, not for that database. On a restart the two were therefore free to
start in either order, and on a freshly installed server we measured Keycloak
starting a full second ahead of the database it depends on.

It got away with it there, but only just. On a slower machine, or after an
unclean shutdown where PostgreSQL has to repair itself before accepting
connections, Keycloak would fail to connect. It would then retry a handful of
times, give up, and stay stopped — and because it is what serves the login page,
the whole platform would be left with no way to sign in until someone logged in
over SSH and started it by hand.

Keycloak now waits for the database before starting, and is given a far more
generous window to keep retrying if the database is slow to come up. Restarting
your server is safe again; the fix applies as soon as you update.

## 2026.09.03.4 — No more false "no logs" warning after installing

A newly installed server reported a warning on its own log collection —
"VictoriaLogs reports 0 bytes ingested (no logs?)" — even though logs were
arriving normally and were perfectly searchable. It was the check that was
wrong, not your log server, but it left every new installation showing a
warning on day one and taught people to ignore a check that is supposed to tell
them when log collection has genuinely stopped.

Vexor ships the checks it runs against itself as part of the package, but the
first-run setup step used to overwrite them with its own older copies. Those
copies had fallen behind: one of them still looked for a storage measurement
that VictoriaLogs renamed several versions ago, and finding nothing, concluded
that nothing had been stored. A second check lost the ability to recognise that
some background tasks are started by a timer rather than running continuously,
and could report them as failed while they were working correctly.

The packaged checks are now used as-is and setup only fills in a check if one
is genuinely missing, so the two cannot drift apart again. Upgrading repairs a
server that already shows the warning — no action needed beyond the update. The
log-collection check now also raises a critical alert if VictoriaLogs stops
accepting new logs because it has run out of disk space, which is the failure
that actually deserves your attention.

## 2026.09.03.3 — The admin account works right after setup

Setup finishes by printing an administrator username and password for your new
server. Signing in with them did not work: the browser sent you to an "Update
Account Information" form before letting you in, and anything connecting
through the API — scripts, integrations, our own tooling — was refused
outright with "Account is not fully set up".

Vexor's sign-in service requires every account to have a first and last name,
and setup created the administrator with only a username and an email address.
The account was therefore incomplete from the moment it was created. Nothing
was wrong with the password, which made the failure particularly confusing.

Setup now creates the administrator as a complete account, so the printed
credentials work immediately, in the browser and over the API alike. Upgrading
also repairs an administrator account that is missing those details, so
existing servers are fixed without anyone editing anything by hand. You can set
the name yourself with the VEXOR_INITIAL_ADMIN_FIRSTNAME and
VEXOR_INITIAL_ADMIN_LASTNAME environment variables.

## 2026.09.03.2 — Local accounts can log in on a fresh install

If you signed in with a local Vexor account rather than single sign-on, the
login could fail with a server error on a newly installed server. Vexor signs
local-account sessions with a secret it kept in `/etc/vexor/local-jwt.secret`,
and it tried to create that file the first time someone logged in. The service
deliberately runs with write access to very little of the system, so on a fresh
install that attempt was refused and the login failed — with the correct
password. A wrong password still returned a normal "invalid credentials", which
made the fault look like a password problem rather than a server one.

The secret is now created when the package is installed, so local login works
from the start. Upgrading keeps any existing secret, so nobody is signed out.
If the file is ever missing and cannot be created, Vexor now says exactly that
instead of returning a generic server error.

Single sign-on (Keycloak, LDAP via Keycloak) was never affected.

## 2026.09.03.1 — A deleted host stays deleted

Deleting a host removed it from monitoring straight away, but it could come back.

Vexor keeps a last-known-good copy of your monitoring configuration so a bad
change can be rolled back automatically. That copy was only refreshed after a
successful Activate, so a host you deleted still existed in it until then. If the
next Activate failed its config check — or the server was restarted with a broken
configuration — the rollback restored the deleted host along with everything else.
It came back as a ghost: no longer in Vexor, but still scheduled by the monitoring
engine, with its checks reporting "Host not found", and still counted in views and
reports. It disappeared again only after the next Activate that succeeded.

Deleting a host now removes it from the last-known-good copy at the same time, so
a rollback restores everything else exactly as before but cannot bring back a host
you deleted. The same applies to the cleanup that runs on every Activate.

## 2026.09.02.1 — Status badges no longer truncate

A service showing `OK` could render as `O..` in the services table on a host
page. The status badge let itself be squeezed by a wide neighbouring column
until only the icon and an ellipsis were left. A shortened status is worse than
ugly — `UNKNOWN` and `UNREACHABLE` shortened to the same thing — so badges now
always show their full label and the column is sized to fit.

## 2026.08.31.1 — Monitor Kubernetes and OpenShift without installing anything

Vexor can now monitor a Kubernetes or OpenShift cluster the same way it monitors
everything else — with alerts, graphs, SLA reports and notification policies — and
without putting an agent on a single node.

You give Vexor a read-only ServiceAccount token. Vexor reads the cluster's own API
and turns it into ordinary Vexor services. Nothing is installed in the cluster,
nothing is changed in it, and no workload runs there on Vexor's behalf.

- **Add a cluster the way you add a host.** Choose Kubernetes or OpenShift in the
  Add host wizard, give Vexor the API server address and a ServiceAccount token,
  and Vexor probes the cluster and tells you what it can see before you commit to
  anything. The new **Kubernetes / OpenShift** page lists your clusters, their
  nodes and their checks.
- **Fifteen checks, and Vexor works out which ones apply.** Ten apply to any
  cluster: workloads running below their intended replica count; pods
  crashlooping, stuck pulling an image, killed for memory or stuck Pending; node
  Ready state, cordon and drain status and resource pressure; PersistentVolumeClaims
  stuck Pending or Lost; services whose endpoints have no ready backend, which is
  the usual shape of "the site is down but every pod looks fine"; failed and
  overrunning batch jobs; warning events grouped by cause; API server liveness and
  readiness; pending certificate signing requests, a quiet cause of nodes dropping
  out of a cluster months later; and ServiceAccount token expiry.
- **Plus what is specific to your distribution.** On OpenShift: cluster operators,
  the cluster version and update status, and MachineConfigPool rollouts. On vanilla
  Kubernetes: the self-hosted control-plane pods, and kubelet version skew between
  nodes after a partial upgrade.
- **Restart rate, not restart count.** A pod that misbehaved last month does not
  alarm forever.
- **Checks that do not apply say so.** Point an OpenShift check at a plain
  Kubernetes cluster and it reports "not applicable", not a failure.
- **Whole cluster, or one namespace at a time.** One team's failing workloads need
  not raise alerts against another team's service. Cluster-wide facts such as node
  health and the control plane are always judged cluster-wide.
- **A cluster counts as one host.** A monitored cluster consumes one licence, no
  matter how many nodes it has, and the nodes Vexor discovers for context cost
  nothing. If you additionally monitor a node the classic way — with an agent on it
  — that node counts as a normal host, as it always has.

### Built so it stays trustworthy

- **The cluster credential stays on the Vexor server.** Vexor reads the cluster on
  a schedule and caches what it found; the individual checks read that cache. The
  token is never handed to the check runner, and the checks make no network calls
  of their own.
- **One read per cluster, not one per check.** A large cluster can return tens of
  megabytes per listing. Vexor fetches it once per interval, and only the resources
  your enabled checks actually need.
- **A monitoring outage is not reported as a service outage.** If Vexor cannot
  reach the cluster, or its cached data has gone stale, the affected checks report
  UNKNOWN rather than CRITICAL — which would otherwise wrongly destroy the measured
  availability of every service in the cluster at once.
- **Read-only, always.** Vexor only ever reads from the cluster, in keeping with how
  it treats every other monitored system.

Upgrade with `dnf upgrade 'vexor-*'`, then restart the services. The database
migration runs automatically. Existing hosts and checks are untouched.

## 2026.08.18.1 — Agent version management, plus a security and platform refresh

You can now manage which NSClient++ build Vexor deploys to Windows hosts, instead
of being tied to whatever version happened to ship with the product. This release
also refreshes the whole underlying platform and fixes several security issues.

- **Choose the agent version you deploy.** A new **Agent versions** page (under
  Onboarding) lists every NSClient++ package Vexor knows about and lets you pick
  the one used for new deployments. You can fetch a stable release straight from
  the official upstream project, upload your own MSI, or keep using the bundled
  build — whichever you set as primary is what gets deployed.
- **Roll back safely.** Previous versions stay in the registry, so if a new agent
  build misbehaves you can switch back with one click rather than hunting down
  the old installer.
- **Know when a new agent is out.** Vexor can check upstream for new stable
  releases on a schedule and tell you when one is available, so staying current
  is a decision rather than a chore. Nothing is downloaded or promoted without
  you asking for it.
- **Help where you'd look for it.** The Platform menu now links straight to the
  in-app documentation.
- **Fixes.** The Logs page no longer comes up blank in a browser tab that was
  left open across an upgrade; large tables (including log views) render their
  rows reliably again; and an expired session now recovers cleanly instead of
  leaving live views silently stalled.

### Security

- Updated bundled Python dependencies to pick up fixes for several published
  vulnerabilities, including a critical SQL-injection issue in the MySQL driver
  (CVE-2025-65896). **Upgrading is recommended for all installations.**
- Refreshed the bundled platform components — log storage, log shipper, identity
  server — to current upstream releases. This includes a log-query fix that could
  cause certain valid searches to be rejected.

Upgrade with `dnf upgrade 'vexor-*'`, then restart the services (or reboot). No
manual configuration changes or database migrations are required: the log shipper
brings a new upstream version with stricter configuration rules, and its own
default configuration is migrated for you during the upgrade. If you hand-edited
your shipper configuration, check the service after upgrading — the upgrade will
tell you if it needs your attention.

## 2026.08.13.1 — Cleaner actions, clearer data and safer bulk operations

The third wave of our product-experience work closes the medium-priority
findings. No new features beyond bulk host operations — the focus is
consistency, clarity and control.

- **Consistent, uncluttered row actions.** Per-row action buttons now follow one
  pattern across the app: the primary action inline and the rest in a tidy
  overflow menu, using a single set of semantic colours (danger is always red,
  success always green) instead of ad-hoc styling.
- **More accessible controls.** Icon-only buttons now carry proper accessible
  names, so screen readers and keyboard users can tell them apart.
- **A welcome that doesn't get in the way.** The product tour no longer pops a
  modal over the page on first login — you get a dismissible welcome prompt and
  can start the tour whenever you like.
- **Cleaner, more trustworthy data.** SLA reports no longer leak raw `uname`
  output into the alias column, MTTR explains what "—" means, and empty
  discovery screens now guide you on what to do next.
- **Resilient job console.** Deployment and log-shipper jobs can be retried,
  re-attached after a dropped connection, and their full logs downloaded.
- **Bulk host operations.** Select multiple hosts to add them to a host group,
  or delete them in one guarded, type-to-confirm action — reusing the same safe
  purge path as single-host deletion (pending until you Activate).

Standard upgrade: `dnf upgrade vexor-api vexor-ui && systemctl restart vexor-api`.
No configuration changes or migrations are required.

## 2026.08.12.2 — A more consistent, responsive and accessible interface

The second wave of our product-experience work. No new features — consistency,
readability and accessibility across the whole app.

- **Your branding shows up everywhere.** White-label logo, product name and
  accent colour now propagate across the header, login, public status page and
  browser tab — not just a few screens.
- **Works on tablets and phones.** The top bar no longer collides with itself on
  narrow screens, headers stack cleanly, and wide tables scroll horizontally
  instead of overflowing the page.
- **No more blank screens on failure.** Lists and panels now show clear loading
  skeletons, empty-state messages and actionable error states instead of going
  blank when a request fails.
- **Actions tell you what happened.** Create, update and delete actions now
  surface success and failure as toast notifications, and destructive actions
  ask for confirmation first — no more silently swallowed errors.
- **Readable, consistent timestamps.** Times are shown as friendly relative
  values (“3 min ago”) with the exact local time and timezone on hover; raw
  ISO strings no longer leak into the UI.
- **Fast, paged history.** Events and the audit logs now page through results on
  the server instead of capping at 500 rows, keeping large histories responsive.
- **Accessibility in CI.** Automated accessibility (axe) scanning now runs in our
  pipeline — the public status page on every push, and the main authenticated
  views on the full end-to-end run — so regressions are caught early.

Standard upgrade: `dnf upgrade vexor-api vexor-ui && systemctl restart vexor-api`.
No configuration changes or migrations are required.

## 2026.08.12.1 — First-impression polish, safer host deletion and a cleaner audit trail

This release closes the six highest-priority findings from our product
experience review — the small things an evaluator notices on the very first
screens. No new features; correctness, safety and polish only.

- **No more silent errors on every page.** Vexor checked for optional modules
  (like Logs) using a route that did not exist, so every page quietly fired a
  failed request and the Logs navigation could disappear. That endpoint now
  exists and always returns a stable answer.
- **Safer host deletion.** Deleting a host now shows how many dependent services
  will be removed and asks you to type the host name to confirm — no more
  one-click accidents. Viewers no longer see Delete, row selection or the bulk
  action bar (buttons that previously appeared and then failed with a
  permission error).
- **An audit trail that reads like one.** Action authors are shown as-is, and
  opaque internal identifiers are now clearly labelled `unknown (…)` with a
  tooltip instead of dumping raw UUIDs.
- **Fonts that actually load.** Vexor declared its Inter and JetBrains Mono
  typefaces but never shipped them, so everyone silently fell back to a system
  font. They are now bundled and self-hosted for a consistent look everywhere.
- **Help pages render properly.** Bold text, links and formatting inside Help
  documentation tables now display correctly instead of showing raw markup.
- **Removed a stray internal note** that had leaked into System Settings.

Standard upgrade: `dnf upgrade vexor-api vexor-ui && systemctl restart vexor-api`.
No configuration changes or migrations are required.

## 2026.08.08.1 — Deploy the Windows agent remotely, straight from the GUI

You can now roll out the Vexor Windows agent to one or many machines without
touching them — no RDP, no USB stick, nothing typed on the target.

- **Remote push (SMB)** — a new tab under Agent deployment. Enter admin
  credentials and a list of Windows hosts, and Vexor installs NSClient++ over
  SMB (PsExec-style) as SYSTEM, then cleans up after itself.
- **Auto-register** — optionally have each freshly installed agent enroll itself
  against the master automatically, so new hosts start reporting with no extra
  steps.
- **Safe overwrite** — if an agent is already present it is stopped and replaced,
  the same way the local installer behaves.
- **Credentials stay secret** — the admin password is never written to logs,
  command lines or config files; it is passed only to the child install process.

Prerequisites per target: TCP 445 + the ADMIN$ share reachable, a local-admin
account, and (on workgroup hosts) `LocalAccountTokenFilterPolicy=1`.

## 2026-08-07.1 — Redesigned navigation and a unified Home

We reorganized Vexor's left-hand navigation around what you are trying to do, so
the right page is easier to find:

- **Purpose-driven sections** — the sidebar is now grouped into clear areas:
  Home, Monitoring, Response, Reports, Configure and Administer (plus Logs when
  the logs module is enabled). Related tools now live together — for example all
  credentials (SNMP included) live under Administer, and agent deployment moved
  to Configure -> Onboarding.
- **A single Home** — the landing page is now a tabbed hub: your Overview
  dashboard, the Monitoring score and Top talkers are tabs on one page instead of
  separate menu entries. Your own dashboards and the full-screen NOC wallboard
  stay their own destinations.
- **Nothing moved out of reach** — existing bookmarks keep working; old links
  such as the Monitoring score and Top talkers pages redirect into the new Home
  tabs automatically.
- SNMP-credential management is now admin-only, so credentials have a single,
  properly protected home.

This is a navigation and findability improvement — your hosts, checks, data and
workflows are unchanged.

## 2026-08-03.1 — Faster, more consistent tables across the whole app

The final phase of a large front-end modernization is now live. Every list and
table view in Vexor has been moved onto one unified data grid, so the whole app
looks and behaves consistently:

- **Sorting, pagination and a column picker** on list/table views, plus a
  density toggle — show only the columns you care about.
- **Virtualized rendering for large lists** (hosts, services, logs) so big tables
  stay smooth and responsive instead of bogging the browser down.
- **Consistent loading, empty and error states** everywhere, replacing a patchwork
  of one-off spinners and tables.
- Behavior is otherwise unchanged — this is a look-and-feel/consistency and
  performance improvement, not a workflow change.

Under the hood this completes the front-end design-system rollout and turns on
strict code-quality guardrails, so future UI regressions are caught before release.


## 2026-08-02.2 — Backup & host-config bug fixes

Fixes shipped alongside a new enforced code-quality gate (lint + type checks now
block CI, so regressions are caught before release):

- **Config backups now include the database again** — one backup code path
  referenced a helper that was never defined, so the database dump was silently
  skipped in that path. The dump is now built correctly (credentials passed
  securely via the environment, never on the command line).
- **Poller-pinned hosts no longer break config generation** — generating the
  monitoring config for a host assigned to a specific poller hit an undefined
  reference and could fail; fixed.
- Corrected a couple of smaller internal issues (two-factor secret handling and
  plugin-install logging) uncovered by the same pass.
- **Dependency security fix** — bumped `pyasn1` to 0.6.4 to clear three known
  advisories in the previous version.


## 2026-08-02.1 — Host incident timeline fix

- Fixed the host **incident timeline** coming back empty. Naemon state changes,
  notifications, acknowledgements and downtimes now load correctly on the host
  timeline view again (an internal monitoring-socket path had drifted).

## 2026-08-01.1 — Security & reliability hardening

A broad hardening pass across Vexor's backend and OpenVMS bridge, focused on
things that protect your data and keep monitoring dependable under load:

- **SSRF protection** — synthetic/HTTP checks and the AI (`ollama_url`) integration
  now validate outbound targets through a single guard that blocks requests to
  loopback, private, link-local and cloud-metadata addresses.
- **Secrets encrypted at rest** — the OpenVMS monitoring bridge now encrypts its
  stored credentials (Fernet) instead of keeping them in plaintext, with a
  per-install key provisioned automatically on upgrade.
- **SSH host-key verification** — OpenVMS and log collectors now pin and verify
  each host's SSH key (trust-on-first-use), so a poll can't be silently
  redirected to an impostor host.
- **Locked-down metrics endpoint** — the Prometheus metrics export is no longer
  world-readable; it requires a scrape token or a local/private client.
- **Dependable config reloads** — monitoring-config writes are now atomic, guarded
  by a cross-process lock, and leader election fails closed, so a crash or a
  second node can no longer corrupt the running configuration.
- **Snappier UI under load** — blocking system commands in the API are now offloaded
  to worker threads and the auth secret is cached, keeping the interface
  responsive while background work runs.

No action is required — these ship automatically. If you scrape Vexor's metrics
endpoint from a public address, set a scrape token (see the Operations guide).

## 2026-07-25.9 — Public demo refreshed with a full MSSQL showcase

The public demo at **demo.vexormon.com** now runs the latest Vexor build and includes a synthetic MSSQL Server host (`mssql-demo01`) with 24 hours of realistic point-in-time data — trends, top queries, wait stats, IO latency and a simulated blocking incident — so you can explore the new MSSQL dashboard without needing a real SQL Server. Also fixed: VictoriaLogs (which backs MSSQL snapshots and the log explorer) wasn't starting in the all-in-one container image.

## 2026-07-25.8 — MSSQL dashboard: professional redesign

Fixed a sticky-navigation overlap bug where the time-travel bar and the tab strip could hide each other while scrolling. Added colour-blind-friendly status icons alongside colour on every KPI/metric card, a collapsible time-travel scrubber (compact by default, expand for the chart), and a one-click **Export** button that produces a clean printable report for management.

## 2026-07-25.7 — MSSQL dashboard: clearer navigation

Reworked the MSSQL dashboard's menu structure so it's easier to find your way around after a few clicks — clearer section labels and breadcrumbs so you don't lose track of where you are.

## 2026-07-25.6 — MSSQL: trend graphs and snapshot scheduling

Added per-database trend graphs (size, sessions, IO latency growth over time) to the MSSQL dashboard, documented how to schedule the **SQL Snapshot (point-in-time)** service so a database ships a full snapshot every minute, and improved the message shown when no snapshots exist yet in the selected time window.

## 2026-07-25.5 — MSSQL: per-database A/B comparison

You can now compare two points in time for a single database — pick a baseline and an incident moment and see exactly what changed (size, sessions, blocking, hot queries) for that specific database, not just the whole server.

## 2026-07-25.4 — MSSQL dashboard: point-in-time drill-down and A/B compare

New MSSQL point-in-time dashboard: reconstruct exactly how a SQL Server looked at any past moment — active sessions, blocking chains, wait stats, per-database IO latency and top queries with their SQL text — and compare any two moments side by side to see what changed during an incident.

## 2026-07-25.3 — MSSQL: deep performance & diagnostics collector

Brand-new, additive MSSQL collector (it doesn't touch any existing MSSQL checks) that matches and extends what Telegraf/Grafana show: buffer cache hit ratio, page life expectancy, batch requests/sec, wait stats, blocking chains, top queries by CPU, per-database IO latency, tempdb usage and more — all viewable in the new MSSQL dashboard.

## 2026-07-25.1 — OpenVMS: dedicated least-privilege monitoring account + hardened SSH

Vexor's OpenVMS monitoring now runs under its own dedicated, least-privilege account instead of a shared/system account, with setup steps documented in the OpenVMS help. Also fixed SSH algorithm negotiation for modern OpenSSH-for-OpenVMS servers, scoped the orphan-session cleanup to only Vexor's own sessions (so it can no longer kill unrelated long-lived SSH sessions such as an sshfs mount), added a longer connection timeout to ride out brief CPU spikes, and made the monitoring bridge start up and shut down cleanly and quickly.

## 2026-07-19.3 — OpenVMS crash-dump check no longer false-alarms on a fresh system

The **Crash dumps** check now ignores the reserved `SYSDUMP.DMP;1` file, which OpenVMS pre-allocates at install to hold a full memory dump and is therefore always present (and large) even when nothing has crashed. The check now goes CRITICAL only on a **preserved** dump - a copy in `SYS$ERRORLOG:*.DMP` or a higher-version `SYSDUMP.DMP;n` left behind by a real crash.

## 2026-07-19.2 — OpenVMS: crash-dump, disk-integrity, RAID & performance checks

Four new OpenVMS checks surface data the bridge already collected but never alerted on: **Crash dumps** (an actionable, non-empty `SYSDUMP.DMP`/`*.DMP` goes CRITICAL), **Disk integrity (DFU)** (INDEXF.SYS free headers, “INDEXF cannot extend”, low/fragmented free space, volume bitmap drift and per-volume file-header usage), **RAID / physical disks** (Smart Array / MSA member FAILED/REBUILDING) and **Performance** (MONITOR page-fault rate, deadlocks and disk I/O queue length). Add them from the OpenVMS check catalog; each reports a benign OK when its data source isn't present.

You can now set a **Backup scan** interval (Settings → OpenVMS → Collection schedules) to throttle how often the backup file list is scanned, and the VSI update check remains throttled by its interval with a package list shared across all hosts of the same architecture (so many OpenVMS hosts make only one VSI request per interval). The OpenVMS help page now documents the exact SSH/DCL commands each check runs, for full transparency.

## 2026-07-19.1 — OpenVMS backup freshness check

New OpenVMS **Backup freshness** check: warns/criticals when the newest backup file is
older than configurable thresholds (default WARNING ≥24h, CRITICAL ≥48h). Point out
where your backups live in the host's **Config** tab (Monitor backups toggle + Backup file
path), or add the *Backup freshness* service from the OpenVMS check catalog. Reports newest
backup age and file count as perfdata; CRITICAL with a clear hint when no files are found.

## 2026-07-18.5 — Clearer OpenVMS "Overview" check

- **Improved:** the OpenVMS **Overview** check now explains itself. Previously it
  read like `OK - openvms1 all systems operational | ... uptime=127202s`, which
  didn't say what it covered and showed uptime in raw seconds. It now returns a
  labelled summary (e.g. `OK - openvms1: all collectors reachable (CPU 0%, mem
  8%, up 1d 11h 20m)`) plus detail lines: collector reachability (SSH / SNMP /
  iLO) and a health snapshot with **human-readable uptime**. It also states that
  it is an at-a-glance roll-up and does not replace the per-subsystem checks.

## 2026-07-18.4 — Jump from a log check straight to its logs

- **New:** log-alert and log-freshness checks now have a **"View logs"** button on
  the service detail page. Clicking a `vexor_logs_*` check (e.g. an OpenVMS log
  freshness dead-man) and pressing **View logs** opens the Logs explorer already
  filtered to that host, so you go from "this host's logs look stale/alerting"
  to the actual log stream in one click — no manual query needed.

## 2026-07-18.3 — OpenVMS end-to-end audit fixes

- **Fixed (important):** a deleted host could linger as a ghost in Naemon. If an
  activation failed right after a delete, the last-known-good rollback restored
  the host's config file even though its database row was already gone, so the
  deleted host kept being scheduled with "Host not found" checks. Every
  **Activate** now reconciles the on-disk host configs against the database and
  removes any orphans (and cleaned up an existing `openvms-01` ghost).
- **OpenVMS Hardware check:** now reports a calm **OK "iLO not configured"** on
  hosts without an HPE iLO (VMs, most Alpha) instead of a misleading UNKNOWN.
  UNKNOWN is kept only when iLO *is* configured but unreachable.
- **OpenVMS help & in-product text:** rewritten to be GUI-first — add hosts with
  the OpenVMS template, manage everything from the host's **Config tab**, set the
  VSI login under **Settings → OpenVMS**. Editing `bridge.yaml`/`bridge.env` by
  hand is now clearly optional. The Processes/Queues "nothing configured" hints
  point at the Config tab instead of a YAML file.

## 2026-07-18.2 — OpenVMS watched-process/queue tag hygiene

- **Fixed:** a stray comma or blank entry in the OpenVMS *Watched processes* / *Watched
  queues* fields (Host → Config tab) no longer becomes a bogus "NOT RUNNING" process
  that falsely turns the **Processes** check CRITICAL. The GUI now splits pasted
  comma/space/semicolon lists into clean tags, trims and upper-cases names, and drops
  punctuation-only junk; the bridge applies the same filter defensively.
- Note: after editing watched processes it can take up to ~1–2 minutes for the change
  to reflect (the bridge re-polls OpenVMS over SSH, then the Naemon check re-runs).

## 2026-07-18.1

**More OpenVMS metrics over SNMP — no extra setup.** Using the SNMP agent OpenVMS
already runs, Vexor now offers three additional checks when you add or edit an
OpenVMS host:

- **Process count** — running processes vs the system's `MAXPROCESSCNT`, so you get
  warned before the process table fills up.
- **Logged-in users** — number of interactive sessions, handy for spotting runaway
  logins.
- **Network** now also tracks per-interface input/output **errors and discards** and
  reports link **utilization %**; you can optionally alert when utilization crosses a
  threshold.

All of this is collected agentlessly — nothing new needs to be installed on the
OpenVMS side.


## 2026-07-17.5

**OpenVMS logs now flow into the log server.** When you add an OpenVMS host with log
collection enabled (or flip the new **Ship logs to Vexor log server** switch on the host's
Config tab), Vexor forwards that system's OPCOM, security-audit, error and accounting log
entries into Logs. They show up in the host's **Logs** tab and can be searched and alerted
on exactly like agent-shipped logs — no OpenVMS-side agent required. We also removed the last
place that told you to hand-edit `bridge.yaml`: the Processes and Queues checks now point you
at the host's Config tab to choose what to watch.


## 2026-07-17.4

**Fixed**

- Deleting a host now cleans up *everything* tied to it. Previously, if a host had log-based alerting enabled, removing it could leave a stray log-alert check behind that made the next *Activate* fail and roll back with a "Could not find any host matching …" error. Host deletion now also removes those log-alert rules and their monitoring entries automatically. OpenVMS hosts are additionally de-registered from the monitoring bridge (including their stored credentials) on delete.

## 2026-07-17.3

**Adding an OpenVMS system is now point-and-click.** OpenVMS is a first-class choice in the Add Host wizard — pick it like any other template and Vexor creates the full set of OpenVMS checks for you. The wizard now collects the SSH login and CPU architecture inline and registers the host with the monitoring bridge on submit, so there are no config files to edit afterwards. We also made the whole wizard friendlier with clearer field labels and template descriptions, and fixed a crash that could occur when un-ticking a check during setup.

## 2026-07-17.2

**Manage OpenVMS entirely from the GUI — no file editing.** Every OpenVMS setting is now configurable from Vexor. Each OpenVMS host has a Config tab where you set the architecture, SSH credentials, and the processes and queues to watch; passwords are write-only and never shown back. A new **Settings → Monitoring definitions → OpenVMS bridge** page covers the global defaults (SNMP, VSI update-check login, and all collection schedules). Changes are applied live — you no longer need to touch `bridge.yaml` on disk.

## 2026-07-17.1

**Built-in OpenVMS help.** A dedicated *OpenVMS monitoring* page now appears under Help & Documentation, covering how to prepare the OpenVMS side (SSH log housekeeping, a least-privilege monitoring account), configure the bridge, add the host in Vexor, and read each check — plus troubleshooting for tokens, updates and iLO.

## 2026-07-13.1

**Fixed**

- The Linux **Uptime** check could fail with a "CHECK_NRPE: Receive header underflow" / "illegal metacharacter" error. Linux uptime now uses plain warning/critical thresholds in hours, so the check runs cleanly. Update your Vexor packages on the server to get the fix.

## 2026-07-08.3

**New: monitor your OpenVMS systems.** Vexor can now monitor OpenVMS (x86, Itanium and Alpha) with no agent installed on the OpenVMS side. An optional, self-contained bridge collects everything over SSH/SNMP and presents it to Vexor as a normal host with a full set of checks — CPU, memory, swap, disks, network, processes, batch queues, licenses, available VSI updates, hardware/iLO, log version-file growth and an overview — so OpenVMS gets the same alerting, SLA reports, graphs and dashboards as the rest of your estate.

## 2026-07-08.2

**Reports now default to English.** The insights and SLA report language selector defaults to English out of the box, with Swedish still available from the dropdown. We also finished translating the last few internal developer notes to English so the whole product speaks one language.


## 2026-07-08.1

**Fixed**

- Uploading a custom NRPE plugin script could fail with a server error (HTTP 500) on some installs where the plugin storage folder ended up owned by the wrong user. The folder is now always provisioned correctly, and the fix automatically repairs affected installs on upgrade.

## 2026-07-07.7

**Jump straight to Keycloak to manage your SSO users.** The Users page now has a clear *Manage SSO users in Keycloak* card with one-click links to the Keycloak admin console, the user list and group role-mappings for your realm. It also explains the difference between single sign-on users (managed in Keycloak) and local users (managed in Vexor), and reminds you that the Keycloak console uses its own administrator login.

## 2026-07-07.6

**Get set up faster, and see your whole platform's health at a glance.** A new **guided setup** walks you through the first things that matter — adding your first host (Vexor scans it for services and spots any existing agent automatically), turning on alerting, and protecting your configuration with a backup — all from one screen with live progress. And under System Health, a new **Platform components** view shows every part of Vexor with its installed and latest available version, an update badge when something newer is out, and whether each service is running — so you always know your monitoring platform itself is healthy and up to date.

## 2026-07-07.5

**Get told when Vexor itself goes down — the one alert it could never send.** Every monitoring system has a blind spot: if it crashes, loses its database, or its host dies, it simply stops alerting and no one finds out. The new **notification watchdog** closes that gap. Vexor continuously proves its alerting pipeline works and sends a heartbeat to an external dead-man's-switch service you control (such as Healthchecks.io). If the heartbeats stop, that independent service raises the alarm. It can also run a periodic end-to-end self-test through your real channels, catching a broken SMTP password or a revoked Slack webhook before a real incident does. Enable it under Alerting setup → Watchdog.

## 2026-07-07.4

**See which network each log IP belongs to.** Log results now display the network operator behind every public IP address — for example `Google LLC` or `Cloudflare, Inc.` — right next to the country flag, with the AS number on hover. This makes it far quicker to tell whether traffic is coming from a cloud provider, an ISP, or somewhere unexpected, without leaving the log view.

## 2026-07-07.3

**See the real IP of network devices sending syslog.** Switches, firewalls and routers that stream syslog to Vexor now show up on the Log shippers page with their actual source IP address. Vexor gained a built-in relay that captures the sender's real IP (something the raw syslog receiver couldn't do) and forwards it to the log store, so diskless network gear is no longer listed without an address. Existing syslog senders are picked up automatically the next time they send a line.

## 2026-07-07.2

**See the IP of every log sender.** The Log shippers page (Logs → Shippers) now has an **IP** column so you can tell at a glance which address each host is shipping from. It's the address directly for hosts registered by IP, and resolved via DNS for named hosts; devices without a DNS record show “—”.

## 2026-07-07.1

**Network devices show up on the Log shippers page.** If you point a switch, firewall or router at Vexor's built-in syslog receiver (port 514), it now appears in **Logs → Shippers** alongside your agent hosts — tagged as a **syslog** source (agents are tagged **agent**). Each row shows last-seen status and a **View logs** link that jumps straight to that device's logs. Handy for diskless gear that can't store its own logs.

## 2026-07-04.21

**Snappier log search.** Typing in the Logs search box is smooth again — the results table, histogram and inactive tabs no longer re-render on every keystroke, so large result sets don't cause input lag.

## 2026-07-04.20

**Log subsystem: field extraction, metrics, reports, context & GeoIP.** The Logs area gains a batch of new capabilities:

- **Field extraction** (Logs → Field extraction): define rules that pull named fields out of raw log lines (e.g. `ip`, `status`) so you can filter, group and chart on them. In the search view, flip on **Apply field extraction** to run your query with the rules applied. Preview a rule against recent logs before saving.
- **Log metrics** (Logs → Log metrics): turn any log query into a graphable metric (count or events-per-second), optionally broken down per source. Set Warning/Critical thresholds to be notified when the value crosses a limit, and view a 24h series.
- **Reports** (Logs → Reports): schedule digests (daily, weekly or every N hours) that summarise a query — total volume, top sources and an optional error count — delivered through your notification channels. Preview or send on demand.
- **Show context**: from any search result, open the surrounding log lines (before/after) for the same host to see what happened around an event.
- **GeoIP**: IP addresses in search results are annotated with a country flag, so you can spot where traffic is coming from at a glance.

Also fixed a small rendering glitch in the Logs dashboard title.

## 2026-07-04.19

**New RDP sessions check.** You can now monitor how many users are logged in over Remote Desktop on a Windows host (via the agent). Set Warning and Critical to the maximum number of RDP users you want to allow - Vexor alerts when more than that are connected. Choose to count all sessions, only active ones, or only disconnected ones. Read-only, no changes to the monitored host.

## 2026-07-04.18

**Fixed empty agent check output.** Windows/Linux agent checks (Uptime, CPU, Memory, Disks, Services) could show blank values like `:` or `/ (%)` in their status. They now display the full details again - e.g. `OK: up 1w 3d, booted ...` and `committed 45.6GB/65.5GB (69%)`.

## 2026-07-04.17

**Simpler uptime alerts.** The Uptime check now just asks for a number of hours in the Warning and Critical fields. Want to be warned if a host rebooted within the last 2 hours and alerted critically within 1 hour? Just type 2 and 1 - no more filter syntax to remember.

## 2026-07-04.16

- Fixed Windows NSClient++ checks (uptime, CPU, memory, service, disk) that were showing a stray "$" in front of the status text; output is now clean both in the live check and the Run-check-now preview.

## 2026-07-04.15

- The interactive Windows log-shipper installer now guides you through log selection: confirm the standard Windows event logs, then add as many extra sources as you like - a file, a folder (all files under it), a wildcard, or another event channel.

## 2026-07-04.14

- Fixed the interactive Windows log-shipper installer so log paths that contain spaces (such as the SQL Server log folder under Program Files) work correctly.

## 2026-07-04.13

- Windows log shipping can now handle non-UTF-8 log files (e.g. SQL Server ERRORLOG in UTF-16, or legacy ANSI logs) via a character-set option, so the text shows up correctly instead of as garbled characters.

## 2026-07-04.12

- Windows log shipping can now collect log files and folder globs (e.g. IIS or app logs), not only Windows event channels. Pass file paths/globs alongside channel names when installing the agent.

## 2026-07-04.11

- Windows log shipping now starts correctly (fixed a Vector configuration field that prevented the agent from loading its config).

## 2026-07-04.10

- Fixed the Windows log-shipper installer so it completes cleanly on a first install (the NSSM service registration no longer fails with a cannot-open-service error).

## 2026-07-04.9
- **Windows log-shipper install fix.** Installing the Vector log shipper on Windows could stop right after the download with a "Cannot bind parameter 'RemainingScripts'" error; the installer now completes cleanly.

## 2026-07-04.8
- **Spot trouble in your logs automatically.** The new **Logs -> Anomaly detection** page finds unusual log activity for you — without writing a single rule. Turn on ready-made detections (SSH brute force, service crashes, out-of-memory, disk errors, privilege escalation, error-rate surges, log sources going silent, or brand-new never-seen messages) with one click, or build your own. Each detection can post to the monitoring console (so it counts toward SLA and business services), and you can ask the built-in AI to explain any detection in plain language.
- **The log-agent installer just works now.** Downloading the Windows log-shipper install script no longer fails with a "Missing bearer token" error — fresh machines can pull it straight away.

## 2026-07-04.7
- The **Agent deployment** page now has a **Log shipper** tab, so setting up log
  streaming lives right next to installing the monitoring agent instead of being
  tucked away in a separate section. (vexor-ui 0.1.0-136)

## 2026-07-04.6
- **Log shippers are easier to install.** The Log shippers page now shows the
  **log ingest token** (with reveal + copy), and every install command already
  has the token filled in — no more hunting for where to set it or replacing
  `<TOKEN>` by hand.
- Fixed the Windows log-shipper install command failing with *“Could not
  establish trust relationship for the SSL/TLS secure channel”* against a
  self-signed Vexor certificate: it now uses `curl.exe -k`, and the broken
  backslash that split the pasted command has been removed.
  (vexor-logs 0.1.0-21, vexor-ui 0.1.0-135)

## 2026-07-04.5
- **Upload your own agent checks.** Agent deployment now has a **Custom
  scripts** tab where you can upload your own NRPE plugins to be bundled with
  the standard ones pushed out to hosts. Every upload is malware-scanned and
  held in a review queue — you see the verdict and any findings, then approve
  or reject before it ships.
- Scanning uses a built-in heuristic scanner (reverse shells, `curl | sh`,
  embedded secrets, TLS-verification bypass, destructive commands). You can
  additionally install **ClamAV** with one click, and enable an optional
  **VirusTotal** hash-only lookup — your script contents never leave the
  server, only a SHA-256 hash is sent. (vexor-api 0.1.0-237, vexor-ui 0.1.0-134)

## 2026-07-04.4
- Fixed the Save button not activating when you change a check's arguments
  (for example a warning or critical threshold) while editing a service.
- Fixed unreadable code snippets in light theme — commands on the Agent
  deployment, Get started and Log shippers pages showed as dark text on a
  dark background. (vexor-ui 0.1.0-133)

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
