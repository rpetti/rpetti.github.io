---
title: "JFrog Xray dbsync will not migrate to v3"
layout: "post"
---

Ran into this issue while trying to migrate JFrog Xray to v3: `dbsync_v1_to_v3 migration prerequisites are not fulfilled: Custom vulnerabilities or components exist`

> Note: I make no guarantees about your data if you follow these instructions. Proceed at your own risk, and contact JFrog support if you have doubts.

Apparently there's some way of creating custom vulnerabilities or components, despite no such feature appearing to exist anywhere in the UI, API, or official documentation. V3 migration bails out of there are more than 10 of them, and JFrog, for whatever reason, requires you to contact support for this issue.

If you don't want to contact support and don't care about your data, you can run this from the console on the xray database (assuming postgres) to see what the counts of these "custom" vulnerabilities and components are:

```sql
SELECT count(*) FROM public_vulnerabilities WHERE vuln_id NOT ILIKE 'xray-%' OR provider NOT ILIKE 'jfrog';
SELECT COUNT(*) FROM public_vulnerabilities WHERE vuln_id ILIKE 'xray-n%';
SELECT COUNT(*) FROM custom_vulnerabilities;
SELECT COUNT(*) FROM custom_components;
```

Then update your xray configuration in `/var/opt/jfrog/xray/etc/system.yaml` to increase the number allowed to be greater than the numbers reported above.

```yaml
server:
  dbSync:
    migration:
      maxCustomVulnerabilitiesAllowed: 200
      maxNpmAuditVulnerabilitiesAllowed: 200
```

Then restart xray, and try to trigger the db migration from the UI again. Note that it will destroy these "custom" vulnerabilities/components, though you likely have never created any such thing to begin.
