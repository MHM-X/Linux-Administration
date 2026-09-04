# Lab 44: "Cordoba": df is lying (or is it du?)

## Description
 Monitoring reports that the root filesystem is under pressure, but a quick du of /var/log shows almost nothing in the logs of the running application at /var/log/cordoba-app.

Find what is holding the space and reclaim it so df and du agree again for practical purposes; currently there's a ~300 MB discrepancy on the root partition /

The service unit is cordoba-hoarder.service.

Test: df -h / and sudo du -sh / report the same used space after reclaiming the ~300 MB discrepancy.

🔗 **Lab Link:** [SadServers - "Cordoba": df is lying (or is it du?)](https://sadservers.com/scenario/cordoba)

<br>

## 🪜 Steps

### Step 1: 

```bash
```
