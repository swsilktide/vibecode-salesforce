# PR Summary: FLS — Read/Edit for All Users (Lead + Contact + Opportunity Attribution)

## Summary

- **FLS for attribution fields is read + edit for all users** via (1) baseline permission set **Attribution_Lead_Contact_All_Users** and (2) **System Administrator profile** (stored as `Admin.profile-meta.xml`). The System Administrator profile must have explicit FLS so metadata configuration (e.g. Lead → Contact field mapping) works; it does not automatically inherit from permission sets.
- **Validation rules enforce correctness** (inbound/outbound channels, Google PPC/SEO, First Touch write-once). FLS is intentionally permissive.

## Mechanism

- **Profile:** System Administrator — retrieved as **Admin** (API name). File: `force-app/main/default/profiles/Admin.profile-meta.xml`. Added 24 `fieldPermissions` (Lead 8 + Contact 8 + Opportunity 8), all `<readable>true</readable>` and `<editable>true</editable>`.
- **Permission sets:** **Attribution_Lead_Contact_All_Users** (Lead + Contact only); **RevOps_Attribution_Admin_Sandbox** (Lead + Contact + Opportunity).

## Metadata files changed

| File | Change |
|------|--------|
| `force-app/main/default/profiles/Admin.profile-meta.xml` | **Retrieved** then **Updated** — added 24 attribution `fieldPermissions` (Lead, Contact, Opportunity) so System Administrator has explicit Read + Edit |
| `force-app/main/default/permissionSets/Attribution_Lead_Contact_All_Users/...` | Created earlier — 16 fieldPermissions (Lead + Contact) |
| `docs/POST_DEPLOY_RUNBOOK.md` | Updated (FLS, assignments) |
| `ATTRIBUTION_IMPLEMENTATION_SUMMARY.md` | Updated Section 3 |
| `docs/REVOPS_DELIVERABLES.md` | Updated section 3 |
| `docs/ATTRIBUTION_FIELD_CREATION_CHECKLIST.md` | **Created** — permanent checklist for new attribution fields |

**Profile:** 1 (Admin = System Administrator). **Permission sets:** 1 baseline + 1 RevOps.

## Post-deploy

- Assign **Attribution_Lead_Contact_All_Users** to all users for Lead + Contact read/edit.
- **System Administrator** now has explicit profile FLS for all 24 attribution fields; Lead → Contact mapping and other metadata configuration will work for admins.

## Validation

- Deploy to vibecode: **succeeded**. Validate in org: Setup → Profiles → System Administrator → Object Settings → Lead/Contact/Opportunity → Field Permissions; confirm attribution fields Read + Edit. Test: Lead → Map Lead Fields can be modified.
