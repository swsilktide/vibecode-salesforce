# Attribution Implementation Summary - Sandbox Only

## SECTION 1 - FIELDS CREATED

### Lead Object (8 fields)
- `First_Touch_Lead_Source__c` (Picklist)
- `First_Touch_Lead_Channel__c` (Picklist)
- `First_Touch_Lead_Channel_Detail__c` (Picklist)
- `First_Touch_Date__c` (DateTime)
- `Last_Touch_Lead_Source__c` (Picklist)
- `Last_Touch_Lead_Channel__c` (Picklist)
- `Last_Touch_Lead_Channel_Detail__c` (Picklist)
- `Last_Touch_Date__c` (DateTime)

### Contact Object (8 fields)
- Same field names as Lead (API names match for mapping)

### Opportunity Object (8 fields)
- Same field names as Lead/Contact

**Total: 24 custom fields created**

## SECTION 2 - VALIDATION RULES CREATED

### Lead Object (5 validation rules)
1. **Inbound_No_Outbound_Ch_First**
   - Prevents Inbound Source from using ZoomInfo/Apollo/Outreach channels for First Touch
   
2. **Inbound_No_Outbound_Ch_Last**
   - Prevents Inbound Source from using ZoomInfo/Apollo/Outreach channels for Last Touch
   
3. **Inbound_Google_PPC_SEO_First**
   - Requires PPC or SEO detail when Inbound Source + Google Channel for First Touch
   
4. **Inbound_Google_PPC_SEO_Last**
   - Requires PPC or SEO detail when Inbound Source + Google Channel for Last Touch
   
5. **First_Touch_Write_Once**
   - Blocks modification of First Touch fields once they have values

### Contact Object (5 validation rules)
- Same validation rules as Lead (same names and logic)

**Total: 10 validation rules created**

## SECTION 3 - PERMISSION SETS (FLS = read/edit for all users)

**Attribution_Lead_Contact_All_Users** (baseline — assign to all users)
- Grants Read/Write to the 8 attribution fields on **Lead** and **Contact** only (16 field permissions)
- Intended to be assigned to all users so FLS allows read and edit for everyone; validation rules enforce correctness

**RevOps_Attribution_Admin_Sandbox** (optional)
- Grants Read/Write to all 24 attribution fields (Lead, Contact, Opportunity)
- Assign to RevOps/Admin/integration users if they need to edit Opportunity attribution

**Note:** The repo does not store profile metadata. FLS for Lead and Contact attribution is implemented via the baseline permission set above. Validation rules enforce correctness; FLS is intentionally permissive.

## SECTION 4 - LEAD CONVERSION FIELD MAPPING

**Note:** Lead field mappings must be configured in Setup UI:
1. Setup → Object Manager → Lead → Fields & Relationships
2. Click "Map Lead Fields" button
3. Map all 8 attribution fields from Lead → Contact:
   - First_Touch_Lead_Source__c → Contact.First_Touch_Lead_Source__c
   - First_Touch_Lead_Channel__c → Contact.First_Touch_Lead_Channel__c
   - First_Touch_Lead_Channel_Detail__c → Contact.First_Touch_Lead_Channel_Detail__c
   - First_Touch_Date__c → Contact.First_Touch_Date__c
   - Last_Touch_Lead_Source__c → Contact.Last_Touch_Lead_Source__c
   - Last_Touch_Lead_Channel__c → Contact.Last_Touch_Lead_Channel__c
   - Last_Touch_Lead_Channel_Detail__c → Contact.Last_Touch_Lead_Channel_Detail__c
   - Last_Touch_Date__c → Contact.Last_Touch_Date__c

**Reason:** Field mappings are not deployable via metadata API; must be configured in Setup UI.

## SECTION 5 - OPTIONAL: OPPORTUNITY INHERITANCE FLOW

**Status:** Skipped. Build spec and UI guide are in **docs/OPPORTUNITY_ATTRIBUTION_FLOW_SPEC.md** and **docs/OPPORTUNITY_ATTRIBUTION_FLOW_BUILD_GUIDE.md** if you add it later.

## SECTION 6 - TEST PLAN STATUS

**Ready for execution:** TC-01 through TC-08 (see instruction sheet).  
**Bulk test CSV:** **docs/TC-08_bulk_leads.csv** (20 leads: 10 valid, 10 invalid).  
**Test evidence:** Fill **docs/REVOPS_DELIVERABLES.md** with results and screenshots.

---

## DEPLOYMENT STATUS

✅ **Validation Passed:** All fields and validation rules validated successfully
⏳ **Ready for Deployment:** Run `sf project deploy start --target-org scottweilert@silktide.com.vibecode`

## NEXT STEPS (post-deploy)

1. **Run post-deploy runbook:** **docs/POST_DEPLOY_RUNBOOK.md** (Lead mapping, FLS, layouts, permission set assignment).
2. **(Optional) Build Opportunity attribution flow** from **docs/OPPORTUNITY_ATTRIBUTION_FLOW_SPEC.md**.
3. **Execute test plan** TC-01–TC-08; use **docs/TC-08_bulk_leads.csv** for bulk test.
4. **Complete deliverables:** Fill **docs/REVOPS_DELIVERABLES.md** with test results and evidence.
5. **Production:** See **docs/PRODUCTION_DEPLOYMENT_PLAN.md** for what to deploy, order, and validation in prod (no HubSpot/flow changes).
