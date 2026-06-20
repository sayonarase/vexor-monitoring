# Vexor — What's new

Short, public release notes for Vexor. Builds are rolling (early access), so the
dates below mark when each change reached the public RPM repo and Docker image.

## 2026-06-20

### Security & maintenance
- Refreshed bundled dependencies across the platform to clear known security
  advisories — backend (Python) and web UI (JavaScript) packages are now on
  patched versions. No action needed; updates ship with the rolling build.
- Updated the bundled log database (VictoriaLogs) to 1.51.0.

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
