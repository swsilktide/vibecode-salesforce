# Attribution Field Creation Checklist (RevOps)

Use this checklist whenever you **add a new attribution field** to Lead, Contact, and/or Opportunity so that FLS and metadata configuration (e.g. Lead → Contact mapping) work for all users and for System Administrator.

---

## Permanent architecture rule

- **Never rely on implicit System Administrator access** for attribution fields. Some metadata (e.g. Lead field mapping during conversion) uses profile FLS, not permission sets.
- **Always:** (1) Baseline permission set (all users), and (2) Explicit System Administrator profile FLS in the repo.

---

## For each new attribution field

### 1. Create the field metadata

- Add `CustomField` (.field-meta.xml) to the correct object(s): Lead, Contact, Opportunity.
- Use consistent API names across objects if they map (e.g. `First_Touch_Lead_Source__c`).

### 2. Baseline permission set (all users)

- **File:** `force-app/main/default/permissionSets/Attribution_Lead_Contact_All_Users/Attribution_Lead_Contact_All_Users.permissionset-meta.xml`
- Add one `<fieldPermissions>` block per object/field you added:
  ```xml
  <fieldPermissions>
      <editable>true</editable>
      <field>Lead.YourNewField__c</field>
      <readable>true</readable>
  </fieldPermissions>
  ```
- Include **Lead** and **Contact** (and **Opportunity** if the field exists on Opportunity and should be editable by all users; otherwise only RevOps_Attribution_Admin_Sandbox).

### 3. System Administrator profile FLS

- **File:** `force-app/main/default/profiles/Admin.profile-meta.xml`  
  (This is the **System Administrator** profile; API name in the org is **Admin**.)
- Add one `<fieldPermissions>` block per object/field:
  ```xml
  <fieldPermissions>
      <editable>true</editable>
      <field>Lead.YourNewField__c</field>
      <readable>true</readable>
  </fieldPermissions>
  ```
- Insert before the closing `</Profile>` tag. Keep alphabetical order by `<field>` if you prefer.
- Include **Lead**, **Contact**, and **Opportunity** for every attribution field that exists on that object.

### 4. RevOps permission set (optional)

- If the new field is on **Opportunity** and should be editable only by RevOps/Admin, add the field only to **RevOps_Attribution_Admin_Sandbox** (and to Admin profile). If it should be editable by all users, add it to **Attribution_Lead_Contact_All_Users** as well.

### 5. Lead conversion mapping (if field is on Lead/Contact)

- **Manual in Setup:** Setup → Object Manager → Lead → Map Lead Fields → add mapping from Lead field → Contact field. (Not deployable via metadata.)

### 6. Validation rules (if applicable)

- Add or update validation rules (e.g. inbound/outbound, Google PPC/SEO, write-once) for the new field on Lead and Contact as required.

### 7. Retrieve profile if not in repo

- If `Admin.profile-meta.xml` is missing or stale:
  ```bash
  sf project retrieve start --target-org vibecode --metadata "Profile:Admin"
  ```
- Then add the new field’s `fieldPermissions` to the profile and deploy.

---

## Quick reference: existing attribution fields

| API Name | Lead | Contact | Opportunity |
|----------|------|---------|-------------|
| First_Touch_Lead_Source__c | ✓ | ✓ | ✓ |
| First_Touch_Lead_Channel__c | ✓ | ✓ | ✓ |
| First_Touch_Lead_Channel_Detail__c | ✓ | ✓ | ✓ |
| First_Touch_Date__c | ✓ | ✓ | ✓ |
| Last_Touch_Lead_Source__c | ✓ | ✓ | ✓ |
| Last_Touch_Lead_Channel__c | ✓ | ✓ | ✓ |
| Last_Touch_Lead_Channel_Detail__c | ✓ | ✓ | ✓ |
| Last_Touch_Date__c | ✓ | ✓ | ✓ |

All of the above have: CustomField metadata, Attribution_Lead_Contact_All_Users (Lead + Contact), RevOps_Attribution_Admin_Sandbox (Lead + Contact + Opportunity), and Admin profile FLS (Lead + Contact + Opportunity).
