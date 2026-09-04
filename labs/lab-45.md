# Lab 45: "Montevideo": restore test snapshot would clobber production

## Description
This host runs a small app whose live data lives under /production/. Nightly snapshot backups land under /snapshots/ in an rsnapshot-style layout.

A recent bug in the snapshot job may have pointed some files in the latest rotation (/snapshots/daily-2026-06-01/) at the live tree instead of making independent copies.

Ops scheduled a dry-run restore from that snapshot into /production/. If any snapshot path still aliases live data, the restore would overwrite production in place.

Read /home/admin/backup-notes.txt. Find every incorrectly shared file between /production/ and /snapshots/daily-2026-06-01/, then repair the snapshot so it is safe to restore from. Do not damage live production data, and do not break intentional space-saving deduplication inside older snapshot rotations.

Note: the tools rsync and dd are available in this server.

Test: Every file in /snapshots/daily-2026-06-01/ mirroring a name in /production/ must be an independent copy: restoring the snapshot must not alias or overwrite live files.
Older rotations (snapshot-1–snapshot-3) must keep shared config.ini hard links.

🔗 **Lab Link:** [SadServers - "Montevideo": restore test snapshot would clobber production](https://sadservers.com/scenario/montevideo)

<br>

## 🪜 Steps

