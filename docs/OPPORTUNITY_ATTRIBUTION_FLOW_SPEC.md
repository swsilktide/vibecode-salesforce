# Opportunity Attribution from Primary Contact — Flow build spec

**Purpose:** On Opportunity create, copy First Touch and Last Touch attribution from the **Primary Contact** (Opportunity Contact Role) to the Opportunity when those fields are blank. First Touch on Opportunity is never overwritten once set.

**Trigger:** Record-Triggered Flow on **Opportunity**, **After Save**, when **Record is created**.

**Build in UI:** Use **docs/OPPORTUNITY_ATTRIBUTION_FLOW_BUILD_GUIDE.md** for step-by-step Flow Builder instructions and the retrieve command to add the flow to the repo.

---

## Assumption

- Primary Contact is represented by **OpportunityContactRole** where **IsPrimary** = true for the Opportunity.
- If your org uses a custom “Primary Contact” lookup on Opportunity instead, adjust the “Get Primary Contact Role” step to use that lookup.

---

## Flow outline

1. **Start**  
   - Object: Opportunity  
   - Trigger: After Save  
   - Entry conditions: **Formula** `ISNEW()` = true (or “When a record is created”).

2. **Get Primary Contact Role**  
   - Get Records: **OpportunityContactRole**  
   - Filter: `OpportunityId` = **{!$Record.Id}**, `IsPrimary` = true  
   - Sort: none  
   - How many: **1**  
   - Store in: e.g. `varPrimaryContactRole`

3. **Decision: Primary Contact Role found?**  
   - If `varPrimaryContactRole` is empty → **End** (do nothing).  
   - If not empty → continue.

4. **Get Contact**  
   - Get Records: **Contact**  
   - Filter: `Id` = **{!varPrimaryContactRole.ContactId}**  
   - How many: 1  
   - Store in: e.g. `varPrimaryContact`

5. **Decision: Contact has attribution to copy?**  
   - Check that at least one of the following is not blank:  
     `varPrimaryContact.First_Touch_Lead_Source__c`,  
     `varPrimaryContact.First_Touch_Lead_Channel__c`,  
     `varPrimaryContact.First_Touch_Lead_Channel_Detail__c`,  
     `varPrimaryContact.First_Touch_Date__c`,  
     `varPrimaryContact.Last_Touch_Lead_Source__c`,  
     `varPrimaryContact.Last_Touch_Lead_Channel__c`,  
     `varPrimaryContact.Last_Touch_Lead_Channel_Detail__c`,  
     `varPrimaryContact.Last_Touch_Date__c`.  
   - If all blank → **End**.  
   - If any not blank → continue.

6. **Build Update Record (Opportunity)**  
   - Create a single **Update Records** on **Opportunity** with Id = **{!$Record.Id}**.  
   - For each attribution field, set only when **Opportunity field is blank** and **Contact value is not blank**:

   **First Touch (only if Opportunity field is blank):**  
   - First_Touch_Lead_Source__c ← `varPrimaryContact.First_Touch_Lead_Source__c` (if Opp blank)  
   - First_Touch_Lead_Channel__c ← `varPrimaryContact.First_Touch_Lead_Channel__c` (if Opp blank)  
   - First_Touch_Lead_Channel_Detail__c ← `varPrimaryContact.First_Touch_Lead_Channel_Detail__c` (if Opp blank)  
   - First_Touch_Date__c ← `varPrimaryContact.First_Touch_Date__c` (if Opp blank)

   **Last Touch (only if Opportunity field is blank):**  
   - Last_Touch_Lead_Source__c ← `varPrimaryContact.Last_Touch_Lead_Source__c` (if Opp blank)  
   - Last_Touch_Lead_Channel__c ← `varPrimaryContact.Last_Touch_Lead_Channel__c` (if Opp blank)  
   - Last_Touch_Lead_Channel_Detail__c ← `varPrimaryContact.Last_Touch_Lead_Channel_Detail__c` (if Opp blank)  
   - Last_Touch_Date__c ← `varPrimaryContact.Last_Touch_Date__c` (if Opp blank)

   **Implementation note:** In Flow, use **Formula** or **Decision** nodes so that each assignment is only applied when `ISBLANK($Record.First_Touch_Lead_Source__c)` (and same for other fields) and the Contact value is not blank. Alternatively, use one **Assignment** set to build a collection or single record variable, then **Update Records** from that.

7. **Fault handling**  
   - Add a **Fault Connector** from the “Get Primary Contact Role” or “Get Contact” or “Update Records” elements.  
   - On fault: **Create a Debug Log** and/or set a custom **Error** field on the Opportunity (if you have one).  
   - Option: send an email to the admin or post to a Chatter group.

8. **Activate** the flow and test with an Opportunity that has a Primary Contact with attribution populated.

---

## Quick test

1. Create a Contact with First/Last Touch fields filled.  
2. Create an Opportunity and add that Contact as an **Opportunity Contact Role** with **Primary** checked.  
3. Save the Opportunity.  
4. Open the Opportunity and confirm First/Last Touch match the Contact.  
5. Edit the Opportunity and clear one Last Touch field; add the Contact again as Primary and save — flow should repopulate only blank fields.

---

## If you use a custom Primary Contact lookup

Replace step 2 with:  
- Get **Contact** where `Id` = **{!$Record.YourPrimaryContactLookup__c}**  
and use that Contact for the rest of the flow.
