# Proxmox Backups to ZimaOS (Prod)

A short write-up of how I back up my Proxmox home lab to my ZimaOS production machine, and why I set it up this way.

## Overview

| Item | Detail |
|---|---|
| Source | Proxmox VE host |
| Destination | ZimaOS (Prod environment), RAID 1 storage |
| Transport | SMB network share |
| Schedule | Monday to Friday at 12pm |
| Retention | Last 3 backups |

---

## Step 1: Attaching the Network Drive from ZimaOS (Prod)

I attached an SMB network share from my ZimaOS (Prod) machine to Proxmox as a storage target.

### Why I did this

- **Backups live off the host.** If backups are stored on the same machine they protect, a disk failure or a bad update can take out the VMs and the backups together. Putting them on a separate machine avoids that single point of failure.
- **ZimaOS is already my production storage.** It has RAID 1, so the backups sit on storage that is already built to survive a single drive failure.
- **SMB is simple and well supported.** Proxmox supports SMB/CIFS shares natively, so there is no extra software to install or maintain.

---

## Step 2: Creating the Backup Job

I created a scheduled backup job that backs up all Proxmox files to the ZimaOS (Prod) RAID 1 storage. It runs Monday to Friday at 12pm.

### Why I did this

- **Automation over memory.** A backup I have to remember to run is a backup that eventually gets skipped. A scheduled job removes that risk.
- **Regular, predictable coverage.** Running every weekday means that at any point I have a recent copy, so the most I would lose is a day or so of changes.
- **Backing up everything.** Covering all Proxmox files means I can rebuild the lab from one place instead of piecing it together.
- **RAID 1 on the target.** The backups are mirrored across two drives, so a single disk failure on the ZimaOS side does not lose them.

---

## Step 3: Retention (Keep the Last 3 Backups)

The job is set to keep only the 3 most recent backups and automatically purge anything older.

### Why I did this

- **Storage stays under control.** Backups grow quickly. Without a limit they would slowly fill the RAID array.
- **Old backups rarely matter.** If something breaks, I almost always want a recent known-good state, not one from weeks ago.
- **Less to manage.** Pruning happens automatically, so there is no manual cleanup.
- **Still some history.** Three copies gives me a small safety margin if the most recent backup turns out to include a problem I did not notice straight away.

---

## Summary

This gives me an automated, hands-off backup flow:

1. Proxmox writes backups to a separate machine (ZimaOS Prod) over SMB.
2. The destination is RAID 1, so the backups are mirrored.
3. A weekday schedule keeps them fresh.
4. Retention keeps only the last 3, so storage never runs away.

## Possible Future Improvements

- Add an off-site or cloud copy to complete a 3-2-1 backup strategy.
- Run periodic test restores to confirm the backups actually work.
- Add alerting for failed backup jobs.
