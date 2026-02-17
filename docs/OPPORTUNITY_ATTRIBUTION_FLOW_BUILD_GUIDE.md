# Build the Opportunity Attribution Flow in Flow Builder

Follow these steps in **Setup → Flows** to create the record-triggered flow. When finished, retrieve it into the repo (see bottom).

---

## 1. Create the flow

1. **Setup** → search **Flows** → **New Flow** → **Record-Triggered Flow**.
2. **Object:** Opportunity.
3. **Trigger the Flow When:** A record is **created**.
4. **Run the Flow:** After the record is saved ( **After Save** ).
5. **Condition Requirements:** None (run every time an Opportunity is created).
6. Click **Done**. Name the flow: **Opportunity Attribution From Primary Contact** (API name will be `Opportunity_Attribution_From_Primary_Contact`).

---

## 2. Get Primary Contact Role

1. From the **+** after the start node, add **Get Records**.
2. **Object:** OpportunityContactRole.
3. **Filter:**  
   - Opportunity Id | Equals | **{!$Record.Id}**  
   - Is Primary | Equals | **True**
4. **How Many Records to Store:** 1.
5. **How to Store Record Data:** Automatically store all fields.
6. **Output:** Create a new variable:  
   - **Variable API Name:** `varPrimaryContactRole`  
   - **Data Type:** Record (OpportunityContactRole)  
   - **Available for input:** unchecked  
   - **Available for output:** unchecked  
7. Label: **Get Primary Contact Role**.

---

## 3. Decision: Primary Contact Role found?

1. From **Get Primary Contact Role**, add **Decision**.
2. **Label:** Primary Contact Role found?
3. **Outcome 1:**  
   - **Label:** Has Primary Role  
   - **Condition:** Resource = `varPrimaryContactRole`, Field = Id, Operator = **Is Null**, Value = **False** (or “when resource has a value”).
4. **Default Outcome:**  
   - **Label:** No / End  
   - (No condition; this path goes to the end.)

---

## 4. Get Contact (only when Has Primary Role)

1. From the **Has Primary Role** outcome, add **Get Records**.
2. **Object:** Contact.
3. **Filter:** Id | Equals | **{!varPrimaryContactRole.ContactId}**.
4. **How Many Records to Store:** 1.
5. **How to Store Record Data:** Automatically store all fields (or only the 8 attribution fields).
6. **Output:** Create variable:  
   - **Variable API Name:** `varPrimaryContact`  
   - **Data Type:** Record (Contact).
7. Label: **Get Contact**.

---

## 5. Update Opportunity (only when Contact has attribution)

1. From **Get Contact**, add **Update Records**.
2. **How to Find Records:** Use the record that triggered the flow → **{!$Record.Id}**.
3. **Set Field Values for the Opportunity:**  
   For each attribution field, set the value with “only when blank” behavior. In Flow Builder you can use **Formula** for each field:

   - **First Touch Lead Source:**  
     `IF(ISBLANK({!$Record.First_Touch_Lead_Source__c}), {!varPrimaryContact.First_Touch_Lead_Source__c}, {!$Record.First_Touch_Lead_Source__c})`
   - **First Touch Lead Channel:**  
     `IF(ISBLANK({!$Record.First_Touch_Lead_Channel__c}), {!varPrimaryContact.First_Touch_Lead_Channel__c}, {!$Record.First_Touch_Lead_Channel__c})`
   - **First Touch Lead Channel Detail:**  
     `IF(ISBLANK({!$Record.First_Touch_Lead_Channel_Detail__c}), {!varPrimaryContact.First_Touch_Lead_Channel_Detail__c}, {!$Record.First_Touch_Lead_Channel_Detail__c})`
   - **First Touch Date:**  
     `IF(ISBLANK({!$Record.First_Touch_Date__c}), {!varPrimaryContact.First_Touch_Date__c}, {!$Record.First_Touch_Date__c})`
   - **Last Touch Lead Source:**  
     `IF(ISBLANK({!$Record.Last_Touch_Lead_Source__c}), {!varPrimaryContact.Last_Touch_Lead_Source__c}, {!$Record.Last_Touch_Lead_Source__c})`
   - **Last Touch Lead Channel:**  
     `IF(ISBLANK({!$Record.Last_Touch_Lead_Channel__c}), {!varPrimaryContact.Last_Touch_Lead_Channel__c}, {!$Record.Last_Touch_Lead_Channel__c})`
   - **Last Touch Lead Channel Detail:**  
     `IF(ISBLANK({!$Record.Last_Touch_Lead_Channel_Detail__c}), {!varPrimaryContact.Last_Touch_Lead_Channel_Detail__c}, {!$Record.Last_Touch_Lead_Channel_Detail__c})`
   - **Last Touch Date:**  
     `IF(ISBLANK({!$Record.Last_Touch_Date__c}), {!varPrimaryContact.Last_Touch_Date__c}, {!$Record.Last_Touch_Date__c})`

4. Label: **Update Opportunity Attribution**.

---

## 6. Fault path (optional)

1. From **Get Primary Contact Role**, open the **fault connector** (small arrow or right-click) and add a **Fault** element that logs or sends an email.
2. Repeat for **Get Contact** and **Update Records** if you want fault handling on each.

---

## 7. Save and activate

1. **Save** the flow.
2. **Activate** it.

---

## 8. Retrieve the flow into the repo

From the project root (e.g. `vibecode-salesforce`), run:

```bash
sf project retrieve start --target-org vibecode --metadata "Flow:Opportunity_Attribution_From_Primary_Contact"
```

If the flow was created with a different API name, use that name in place of `Opportunity_Attribution_From_Primary_Contact`. The flow file will appear under `force-app/main/default/flows/`. Commit it so the flow is version-controlled.

---

## Quick test

1. Create a **Contact** with First/Last Touch fields filled.
2. Create an **Opportunity**.
3. Add that Contact as an **Opportunity Contact Role** and check **Primary**.
4. Save the Opportunity.
5. Open the Opportunity and confirm First/Last Touch match the Contact.
