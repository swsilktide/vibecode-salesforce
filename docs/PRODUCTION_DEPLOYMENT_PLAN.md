# Attribution — Production Deployment Plan

Deploy attribution metadata (fields, validation rules, permission sets, profile FLS) to **Production** only. No HubSpot or Flow changes. This plan covers what to deploy, order of deployment, and how to validate in production.

---

## 1. What to Deploy

### 1.1 Custom fields (24 total)

| Object      | Count | API names (8 per object) |
|------------|-------|---------------------------|
| Lead       | 8     | First_Touch_Lead_Source__c, First_Touch_Lead_Channel__c, First_Touch_Lead_Channel_Detail__c, First_Touch_Date__c, Last_Touch_Lead_Source__c, Last_Touch_Lead_Channel__c, Last_Touch_Lead_Channel_Detail__c, Last_Touch_Date__c |
| Contact    | 8     | Same as Lead (same API names for mapping) |
| Opportunity| 8     | Same as Lead/Contact |

**Paths:** `force-app/main/default/objects/{Lead|Contact|Opportunity}/fields/*.field-meta.xml` (attribution fields only; other object fields in the repo are excluded if you use a manifest, or deploy full project).

### 1.2 Validation rules (10 total)

| Object   | Count | Rule names |
|----------|-------|------------|
| Lead     | 5     | Inbound_No_Outbound_Ch_First, Inbound_No_Outbound_Ch_Last, Inbound_Google_PPC_SEO_First, Inbound_Google_PPC_SEO_Last, First_Touch_Write_Once |
| Contact  | 5     | Same as Lead |

**Paths:** `force-app/main/default/objects/{Lead|Contact}/validationRules/*.validationRule-meta.xml` (attribution rules only; Account validation rules are separate).

### 1.3 Permission sets (2)

| Permission set | Purpose |
|----------------|--------|
| **Attribution_Lead_Contact_All_Users** | Read + Edit on 8 Lead + 8 Contact attribution fields. Assign to **all users**. |
| **RevOps_Attribution_Admin_Sandbox**    | Read + Edit on Lead + Contact + Opportunity (24 fields). Assign to RevOps/Admin only if they need to edit Opportunity attribution. |

**Paths:** `force-app/main/default/permissionSets/Attribution_Lead_Contact_All_Users/`, `force-app/main/default/permissionSets/RevOps_Attribution_Admin_Sandbox/`.

### 1.4 Profile (FLS)

| Profile | Change |
|---------|--------|
| **Admin** (System Administrator) | 24 field permissions added (Read + Edit for Lead, Contact, Opportunity attribution fields). Required so admins can configure Lead → Contact mapping and other Setup. |

**Path:** `force-app/main/default/profiles/Admin.profile-meta.xml`.

**Note:** If production uses a different profile name (e.g. “System Administrator” with different API name), retrieve that profile from production and add the same 24 `fieldPermissions` before deploying, or deploy Admin and align manually.

---

## 2. Deployment Order

Deploy in this order to avoid dependency and FLS issues:

| Step | What | Why |
|------|------|-----|
| **1** | **Custom fields** (Lead, Contact, Opportunity) | Fields must exist before validation rules, permission sets, or profile FLS reference them. |
| **2** | **Permission sets** (Attribution_Lead_Contact_All_Users, RevOps_Attribution_Admin_Sandbox) | Grant FLS on the new fields so users can see and edit once rules are on. |
| **3** | **Profile** (Admin) | System Administrator needs explicit FLS for mapping and Setup; deploy after fields exist. |
| **4** | **Validation rules** (Lead, Contact) | Deploy last so existing data and integrations are not blocked before FLS is in place. |

**Single-command option:** If your production org has no conflicting changes, you can deploy the full project in one go; Salesforce will resolve dependencies. The order above is the **recommended sequence** if you deploy in stages (e.g. fields first, then validate, then rest).

---

## 3. How to Deploy

### 3.1 Prerequisites

- Production org alias or username (e.g. `Production` or `org@company.com.prod`).
- CLI: `sf` (Salesforce CLI) installed and authenticated to production.
- **Validate against production first** (dry run): `sf project deploy validate --target-org <prod-alias>`.

### 3.2 Validate (dry run) — required before deploy

```bash
# From repo root
sf project deploy validate --target-org <your-production-alias>
```

Fix any errors (e.g. missing dependencies, profile conflicts) before proceeding.

### 3.3 Deploy (full project)

```bash
sf project deploy start --target-org <your-production-alias>
```

This deploys all source in `force-app/main/default`, including the attribution components above. If you need to exclude non-attribution metadata, use a manifest (see 3.4).

### 3.4 Optional: Deploy only attribution metadata (manifest)

To deploy **only** attribution-related metadata and avoid touching other objects/profiles:

1. Create a `manifest/package-attribution.xml` that lists:
   - Custom fields on Lead, Contact, Opportunity (attribution 8 per object).
   - Validation rules on Lead and Contact (5 per object).
   - Permission sets: Attribution_Lead_Contact_All_Users, RevOps_Attribution_Admin_Sandbox.
   - Profile: Admin (if you are comfortable deploying only the Admin profile; otherwise add FLS manually in production).

2. Deploy with:

   ```bash
   sf project deploy start --manifest manifest/package-attribution.xml --target-org <your-production-alias>
   ```

If you use a manifest, deploy in the order of **Section 2** (e.g. two deploys: (1) fields, (2) permission sets + profile + validation rules), or one deploy if the manifest includes all and the org has no conflicts.

---

## 4. Post-Deploy Configuration (Production, no HubSpot/Flow)

These steps are **manual** in production (same as sandbox).

### 4.1 Lead → Contact field mapping

1. Setup → Object Manager → Lead → Fields & Relationships.
2. **Map Lead Fields** → map all 8 attribution fields from Lead to Contact (same names).
3. Save.

### 4.2 Assign permission set to all users

- Assign **Attribution_Lead_Contact_All_Users** to all users (e.g. via Permission Set Group or by profile/role).
- Optionally assign **RevOps_Attribution_Admin_Sandbox** to RevOps/Admin if they need to edit Opportunity attribution.

### 4.3 Page layouts (optional but recommended)

- Add an **“Attribution (System Managed)”** section to Lead, Contact, and Opportunity layouts with the 8 attribution fields (as in sandbox).

---

## 5. Validation in Production (no HubSpot, no Flows)

Confirm the following in production **after** deploy and post-deploy steps. No HubSpot or Flow changes are required.

### 5.1 Deployment and Setup checks

| Check | How |
|-------|-----|
| Deploy succeeded | CLI output or Deployment Status in Setup. |
| Fields exist | Setup → Object Manager → Lead/Contact/Opportunity → Fields & Relationships; confirm all 8 attribution fields per object. |
| Validation rules active | Setup → Object Manager → Lead/Contact → Validation Rules; confirm 5 rules each. |
| Permission sets | Setup → Permission Sets; confirm both sets exist and have correct field permissions. |
| Admin profile FLS | Setup → Profiles → System Administrator → Object Settings → Lead/Contact/Opportunity → Field Permissions; confirm attribution fields Read + Edit. |
| Lead mapping | Setup → Lead → Map Lead Fields; confirm all 8 attribution fields mapped to Contact. |

### 5.2 Functional tests (manual, in production)

Run these **after** assigning **Attribution_Lead_Contact_All_Users** to test users.

| Test | Steps | Expected |
|------|--------|----------|
| **Valid Lead** | Create Lead with Lead Source = Inbound, Channel = Google, Channel Detail = PPC (or SEO); set First/Last Touch as needed. | Save succeeds. |
| **Invalid: Inbound + outbound channel** | Create or edit Lead: Source = Inbound, Channel = ZoomInfo (or Apollo/Outreach). | Validation error: Inbound cannot use outbound channels. |
| **Invalid: Google without PPC/SEO** | Lead: Source = Inbound, Channel = Google, Detail = something other than PPC or SEO. | Validation error: When channel is Google (Inbound), detail must be PPC or SEO. |
| **First Touch write-once** | Create Lead with First Touch fields populated; save; edit and try to change a First Touch field. | Validation error: First Touch is write-once. |
| **Lead convert** | Convert a Lead with attribution populated to Contact. | Contact has same 8 attribution values (via Map Lead Fields). |

### 5.3 Optional: Bulk insert test (e.g. Salesforce Inspector)

- Use **docs/TC-08_bulk_leads.csv** (10 valid, 10 invalid rows).
- Bulk insert to Lead in production (or a sandbox copy of prod).
- Expect: 10 succeed, 10 fail with the validation errors above.

---

## 6. Rollback

- **Validation rules:** Deactivate or delete the 10 rules in Setup if you need to stop enforcement temporarily; no code deploy required.
- **Permission sets:** Unassign from users; optionally delete the permission sets (after unassign).
- **Profile:** Revert Admin profile field permissions for the 24 fields (manual or retrieve/edit/redeploy).
- **Fields:** Custom fields cannot be deleted if they contain data; you can remove from layouts and hide via FLS. Deleting fields requires data migration and is a separate change.

---

## 7. Summary

| Item | Deploy via | Post-deploy (manual) |
|------|------------|------------------------|
| 24 attribution fields | Metadata (CLI) | — |
| 10 validation rules | Metadata (CLI) | — |
| 2 permission sets | Metadata (CLI) | Assign Attribution_Lead_Contact_All_Users to all users |
| Admin profile FLS | Metadata (CLI) | — |
| Lead → Contact mapping | — | Setup → Map Lead Fields (all 8) |
| Layouts | — | Optional: add “Attribution (System Managed)” section |

**Order:** Fields → Permission sets → Profile → Validation rules (or one full deploy after `sf project deploy validate`).

**Production validation:** Deployment + Setup checks (5.1), then manual functional tests (5.2); no HubSpot or Flow changes.
