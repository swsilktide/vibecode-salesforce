# Section 3 — Deliverables Back to RevOps

**Evidence:** Screenshots in `assets/` (Lead field list, validation rules, First Touch write-once error, Contact conversion). Bulk results: `docs/TC-08_bulk_test_results.csv`.

## 1. Fields created/confirmed (API names and object placement)

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

All are custom fields. Source/Channel/Detail are Picklist; Date fields are DateTime.

---

## 2. Validation rules (names and intent)

| Object | Rule Name | Intent |
|--------|-----------|--------|
| Lead | Inbound_No_Outbound_Ch_First | Inbound First Touch cannot use ZoomInfo/Apollo/Outreach |
| Lead | Inbound_No_Outbound_Ch_Last | Inbound Last Touch cannot use ZoomInfo/Apollo/Outreach |
| Lead | Inbound_Google_PPC_SEO_First | Inbound + Google First Touch → Detail must be PPC or SEO |
| Lead | Inbound_Google_PPC_SEO_Last | Inbound + Google Last Touch → Detail must be PPC or SEO |
| Lead | First_Touch_Write_Once | First Touch fields are write-once once set |
| Contact | Inbound_No_Outbound_Ch_First | Same as Lead |
| Contact | Inbound_No_Outbound_Ch_Last | Same as Lead |
| Contact | Inbound_Google_PPC_SEO_First | Same as Lead |
| Contact | Inbound_Google_PPC_SEO_Last | Same as Lead |
| Contact | First_Touch_Write_Once | Same as Lead |

---

## 3. Permission sets and FLS (read/edit for all users)

**Baseline permission set:** `Attribution_Lead_Contact_All_Users`  
**Grants:** Read + Write on the 8 attribution fields on **Lead** and **Contact** only (16 field permissions).  
**Assignment:** Assign to **all users** so effective FLS is read and edit for Lead and Contact attribution. Correctness is enforced by validation rules; FLS is intentionally permissive.

**Optional:** `RevOps_Attribution_Admin_Sandbox` — Read + Write on Lead, Contact, and **Opportunity** (24 field permissions). Assign to RevOps/Admin if they need to edit Opportunity attribution.

**Confirmation:** _[ ] Attribution_Lead_Contact_All_Users assigned to all users; validation rules enforce correctness._

---

## 4. Lead conversion mapping

**Configured in Setup UI (Map Lead Fields):** Lead → Contact mappings for all 8 attribution fields.

| Lead Field | Mapped To |
|------------|-----------|
| First_Touch_Lead_Source__c | Contact.First_Touch_Lead_Source__c |
| First_Touch_Lead_Channel__c | Contact.First_Touch_Lead_Channel__c |
| First_Touch_Lead_Channel_Detail__c | Contact.First_Touch_Lead_Channel_Detail__c |
| First_Touch_Date__c | Contact.First_Touch_Date__c |
| Last_Touch_Lead_Source__c | Contact.Last_Touch_Lead_Source__c |
| Last_Touch_Lead_Channel__c | Contact.Last_Touch_Lead_Channel__c |
| Last_Touch_Lead_Channel_Detail__c | Contact.Last_Touch_Lead_Channel_Detail__c |
| Last_Touch_Date__c | Contact.Last_Touch_Date__c |

**Confirmation:** [x] Completed in sandbox (field mapping configured in Setup).

---

## 5. Test results summary (TC-01 through TC-08)

| ID | Description | Result | Notes / evidence |
|----|-------------|--------|-------------------|
| TC-01 | Field existence on Lead, Contact, Opportunity | **PASS** | Lead Fields & Relationships screenshot: 8 attribution fields (First/Last Touch Source, Channel, Channel Detail, Date). Contact/Opportunity same fields deployed. |
| TC-02 | Attribution edit access (FLS permissive) | **PASS** | FLS allows read/edit for all users via baseline permission set + System Administrator profile; validation rules enforce correctness. |
| TC-03 | First Touch write-once blocks edit | **PASS** | Lead "Valid1" edit: attempt to change First Touch triggered error "First Touch attribution is write-once and cannot be modified after it is set." (screenshot) |
| TC-04 | Inbound cannot use ZoomInfo/Apollo/Outreach | **PASS** | Bulk CSV: InvalidCh1–InvalidCh5 failed with "Inbound attribution cannot use outbound channels (ZoomInfo/Apollo/Outreach)." |
| TC-05 | Inbound Google must be PPC or SEO | **PASS** | Bulk CSV: InvalidGoogle1–InvalidGoogle5 failed with "When channel is Google (Inbound), detail must be PPC or SEO." |
| TC-06 | Lead conversion preserves attribution on Contact | **PASS** | Contact Valid1 (003Pt00000WI5BaIAJ): First Touch Inbound/Google/PPC, Last Touch Inbound/Website/Demo Request — matches Lead. (screenshot) |
| TC-07 | (Optional) Opportunity inherits from Primary Contact | **N/A** | Flow not implemented. |
| TC-08 | Bulk test via Salesforce Inspector (20 Leads) | **PASS** | 10 valid rows Succeeded/Inserted; 10 invalid rows Failed with expected validation messages. CSV export attached. |

**Bulk test CSV:** `docs/TC-08_bulk_leads.csv`. Export evidence: 10 insert success (Valid1–Valid10), 10 validation failures (InvalidCh1–5, InvalidGoogle1–5) with correct error text.

---

## Evidence to attach (from instruction sheet)

- [x] Screenshot of field list on Lead object  
- [x] Screenshot of validation rules list (names + active)  
- [x] FLS / edit access confirmed (permissive; validation rules enforce logic)  
- [x] Conversion proof: Lead and resulting Contact with same attribution  
- [x] CSV + Inspector success/failure log for TC-08  

---

**Sandbox:** vibecode (scottweilert@silktide.com.vibecode)  
**Completed by:** scott weilert  
**Date:** 02-17-2026
