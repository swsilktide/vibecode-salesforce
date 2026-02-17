# Attribution — Post-Deploy Runbook (Sandbox)

Complete these steps in the sandbox after deploying the attribution metadata.

---

## 1. Lead conversion field mapping

**Goal:** When a Lead is converted, all First/Last Touch fields copy to the new Contact.

1. In Setup, go to **Object Manager** → **Lead** → **Fields & Relationships**.
2. Click **Map Lead Fields** (or use Quick Find: "Map Lead Fields").
3. Under **Contact**, add or confirm these mappings (Lead field → Contact field):

   | Lead Field | Contact Field |
   |------------|---------------|
   | First_Touch_Lead_Source__c | First_Touch_Lead_Source__c |
   | First_Touch_Lead_Channel__c | First_Touch_Lead_Channel__c |
   | First_Touch_Lead_Channel_Detail__c | First_Touch_Lead_Channel_Detail__c |
   | First_Touch_Date__c | First_Touch_Date__c |
   | Last_Touch_Lead_Source__c | Last_Touch_Lead_Source__c |
   | Last_Touch_Lead_Channel__c | Last_Touch_Lead_Channel__c |
   | Last_Touch_Lead_Channel_Detail__c | Last_Touch_Lead_Channel_Detail__c |
   | Last_Touch_Date__c | Last_Touch_Date__c |

4. Save.

**Note:** Conversion only fills Contact (and optional Opportunity) from Lead. It does not overwrite existing Contact values.

---

## 2. Field-level security (read/edit for all users)

**Goal:** All users can read and edit Lead and Contact attribution fields. Correctness is enforced by validation rules (inbound/outbound channels, Google PPC/SEO, First Touch write-once), not by FLS.

**Permission set (repo has no profile metadata):**  
- **Attribution_Lead_Contact_All_Users** — Grants Read/Write to the 8 attribution fields on Lead and Contact only.  
- Assign this permission set to **all users** (e.g. via Permission Set Group, or assign to each profile/role so everyone has it).  
- After assignment, effective FLS for Lead and Contact attribution fields is read + edit for everyone.

**RevOps_Attribution_Admin_Sandbox** (optional):  
- Grants Read/Write to Lead, Contact, **and Opportunity** attribution fields.  
- Assign only to RevOps/Admin/integration users if you need them to edit Opportunity attribution; otherwise the baseline permset above is sufficient for Lead + Contact.

---

## 3. Page layouts — “Attribution (System Managed)” section

**Goal:** All 8 attribution fields appear in one section. All users can edit (FLS is read/edit; validation rules enforce correctness).

For **Lead**, **Contact**, and **Opportunity**:

1. Setup → **Object Manager** → **[Object]** → **Page Layouts**.  
2. Edit the layout(s) used by your app (e.g. Lead Layout, Contact Layout, Opportunity Layout).  
3. Add a new section:
   - **Section name:** `Attribution (System Managed)`
   - **Columns:** 2 (or 1 if you prefer).
   - **Layout:** drag these 8 fields into the section (order as below).

**Fields to include (same order on all three objects):**

- First Touch Lead Source  
- First Touch Lead Channel  
- First Touch Lead Channel Detail  
- First Touch Date  
- Last Touch Lead Source  
- Last Touch Lead Channel  
- Last Touch Lead Channel Detail  
- Last Touch Date  

4. Save.

Sales users will see the section but fields will be read-only if FLS is set per Option B above.

---

## 4. Assign permission sets

1. **Attribution_Lead_Contact_All_Users** (required for FLS):  
   Setup → **Permission Sets** → **Attribution Lead Contact All Users** → **Manage Assignments** → assign to **all users** (or to every profile/role so everyone has it).

2. **RevOps_Attribution_Admin_Sandbox** (optional):  
   If you need RevOps/Admin to edit **Opportunity** attribution fields, assign this permission set to those users.

---

## 5. (Optional) Opportunity attribution from Primary Contact

**Skipped for now.** To add later: build per **docs/OPPORTUNITY_ATTRIBUTION_FLOW_SPEC.md** and **docs/OPPORTUNITY_ATTRIBUTION_FLOW_BUILD_GUIDE.md**, then activate.

---

## Checklist

- [x] Lead → Contact field mapping (all 8 attribution fields)  
- [x] FLS: **Attribution_Lead_Contact_All_Users** assigned to all users (read/edit Lead + Contact)  
- [x] Lead layout: “Attribution (System Managed)” section with 8 fields  
- [x] Contact layout: “Attribution (System Managed)” section with 8 fields  
- [x] Opportunity layout: “Attribution (System Managed)” section with 8 fields  
- [ ] (Optional) RevOps_Attribution_Admin_Sandbox assigned to RevOps/Admin if editing Opportunity attribution  
- [ ] (Skipped) Opportunity attribution flow — not implemented  

Sandbox attribution setup is complete. Test plan TC-01–TC-08 and evidence are in **REVOPS_DELIVERABLES.md**.

---

## 6. TC-08 bulk test (Salesforce Inspector Reloaded)

1. Use the CSV **docs/TC-08_bulk_leads.csv** (20 rows: 10 valid, 5 invalid outbound channel, 5 invalid Google detail).
2. In Salesforce Inspector Reloaded: **Data** → **Insert** → choose **Lead**, upload the CSV.
3. Map columns to Lead fields; omit **Expected_Result** (reference only) or map to a custom text field if you have one.
4. Run insert. **Expected:** 10 records succeed; 10 fail with validation errors (Inbound cannot use outbound channels / When channel is Google (Inbound), detail must be PPC or SEO).
5. Export or screenshot the success/failure log and attach to **REVOPS_DELIVERABLES.md**.
