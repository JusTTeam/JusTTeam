**Solution Overview**

Since HubSpot’s standard workflow actions cannot dynamically assign contacts to Salesforce campaigns based on the cid, we need to explore alternative approaches. One effective solution is to **leverage Salesforce’s Flow Builder** to automate this process within Salesforce.

**Here’s how the solution will work:**

1. **In HubSpot:**

•  **Capture the** cid **value** when a contact submits a form and **sync it to a field in Salesforce**.

•  **Ensure the contact or lead is created or updated in Salesforce** with the cid value.

2. **In Salesforce:**

•  **Create a Flow** that triggers when a lead or contact is created or updated.

•  **Use the** cid **value** to find the corresponding Salesforce campaign.

•  **Add the lead or contact to the campaign** with the desired campaign member status.

##
**Step-by-Step Implementation Guide**

**Prerequisites**

•  **HubSpot-Salesforce Integration** is properly set up and syncing contacts/leads between the two systems.

•  cid **Field in HubSpot and Salesforce:**

•  In HubSpot, the cid is stored in a contact property.

•  In Salesforce, you have a corresponding field on the Lead and Contact objects to store the cid.


##
**Part 1: HubSpot Configuration**

  

**Step 1: Ensure cid Is Synced to Salesforce**

  

1. **In HubSpot:**

•  Verify that the form captures the cid value and stores it in a contact property, e.g., **“SFDC Campaign ID”**.

2. **In HubSpot-Salesforce Field Mapping:**

•  Go to **Settings > Integrations > Salesforce > Field Mappings**.

•  Map the HubSpot cid contact property to a custom field on the Salesforce Lead and Contact objects.

•  **Salesforce Field:** Create a custom field on both Lead and Contact objects, e.g., **“HubSpot CID”** (Text field).

•  **Mapping:** Map the HubSpot cid property to the Salesforce **“HubSpot CID”** field.

•  Ensure that the **sync direction** is set appropriately (typically **HubSpot to Salesforce**).

3. **Test the Sync:**

•  Create a test contact in HubSpot with a cid value.

•  Verify that the contact is synced to Salesforce and that the **“HubSpot CID”** field is populated.

**Part 2: Salesforce Configuration**

  

**Step 2: Create Custom Fields (If Not Already Done)**

  

1. **Create “HubSpot CID” Field:**

•  **Object:** Lead

•  **Field Label:** HubSpot CID

•  **Field Name:** HubSpot_CID__c

•  **Data Type:** Text

•  **Repeat for the Contact object.**

2. **Create “Campaign ID Lookup” Field (Optional):**

•  If you prefer to use a lookup relationship to the Campaign object.


**Step 3: Create the Salesforce Flow**

  

We’ll create a **Record-Triggered Flow** that triggers when a Lead or Contact is created or updated with a cid value.

  

**Note:** The steps below are for Salesforce Lightning Experience.


**Step 3.1: Access Flow Builder**

1. **Navigate to:** **Setup** (Gear icon) > **Setup**.

2.  In the Quick Find box, type **“Flows”**.

3.  Click on **Flows** under **Process Automation**.

4.  Click **New Flow**.

**Step 3.2: Choose Flow Type**

1.  Select **“Record-Triggered Flow”**.

2.  Click **Create**.


**Step 3.3: Configure Trigger**

1. **Object:**

•  **For Leads:**

•  Select **Lead**.

•  **For Contacts:**

•  You may need to create a separate flow or handle both objects in one flow using Flow variants.

2. **Trigger the Flow When:**

•  **A record is created or updated.**

3. **Set Entry Conditions:**

•  **Condition Requirements:**

•  **All Conditions Are Met (AND).**

•  **Conditions:**

•  **HubSpot CID** **Is Null** **False**

•  This means the **“HubSpot CID”** field is **not empty**.

•  **Optional:** Add additional conditions to ensure the cid is valid.

4. **Optimize the Flow For:**

•  **Actions and Related Records.**

5.  Click **Done**.




**Step 3.4: Add a Decision Element (Optional)**

  

If you want to check whether the Lead or Contact is already a member of the campaign to avoid duplicates.

1. **Element:** **Decision**

2. **Label:** **“Is Lead/Contact Already in Campaign?”**

3. **Outcomes:**

•  **Yes (Already Member):**

•  **Condition:** Use a **Get Records** element to check if a Campaign Member record exists for this Lead/Contact and Campaign.

•  **No (Not a Member):**

•  Proceed to add the Lead/Contact to the Campaign.



**Step 3.5: Get the Campaign Record**

1. **Element:** **Get Records**

2. **Label:** **“Get Campaign by CID”**

3. **Object:** **Campaign**

4. **Conditions:**

•  **Id** **Equals** **{!$Record.HubSpot_CID__c}**

•  This assumes that the cid value is the Salesforce Campaign ID.

5. **Store Only the First Record Fetched**

6. **When No Records Are Found:**

•  Decide whether to handle this case (e.g., send an error notification or skip).

7.  Click **Done**.



**Step 3.6: Create Campaign Member Record**

1. **Element:** **Create Records**

2. **Label:** **“Add Lead/Contact to Campaign”**

3. **Create One Record**

4. **Object:** **Campaign Member**

5. **Set Field Values:**

•  **CampaignId:** **{!Get_Campaign_by_CID.Id}**

•  **LeadId or ContactId:**

•  **For Leads:** **{!$Record.Id}**

•  **For Contacts:** **{!$Record.Id}**

•  **Status:** **“Responded”** (or desired status)

6.  Click **Done**.



**Step 3.7: Connect the Flow Elements**

1. **Start Element** → **Get Campaign by CID**

2. **Get Campaign by CID:**

•  If campaign is found:

•  **Add Lead/Contact to Campaign**

•  If no campaign found:

•  Optionally, handle the case (e.g., send an alert).

3. **End Flow**



**Step 3.8: Save and Activate the Flow**

1.  Click **Save**.

2.  Provide a **Flow Label**, e.g., **“Add Lead to Campaign by CID”**.

3.  Click **Activate**.



**Handling Both Leads and Contacts in One Flow (Optional):**

•  You can create a **Record-Triggered Flow** on the **Campaign Member** object instead.

•  Or, create separate flows for Leads and Contacts.


**Step 4: Test the Salesforce Flow**

  

1. **Create a Test Lead or Contact in HubSpot:**

•  Ensure it has a cid value that corresponds to an existing Campaign ID in Salesforce.

2. **Verify Sync to Salesforce:**

•  Check that the Lead or Contact is created or updated in Salesforce with the HubSpot CID field populated.

3. **Check the Campaign Membership:**

•  Verify that the Lead or Contact has been added to the Campaign with the correct status.

4. **Handle Errors:**

•  If the flow fails (e.g., because the cid doesn’t match any Campaign), ensure there are mechanisms to catch and handle errors.



**Additional Considerations**

  

**1. Ensuring Data Integrity**

  

•  **Valid** cid **Values:**

•  The cid values provided must correspond to valid Salesforce Campaign IDs.

•  Consider adding validation to ensure that invalid cid values are flagged.

  

**2. Error Handling and Notifications**

  

•  **Add Fault Paths:**

•  In your flow, you can add fault paths to handle exceptions.

•  **Send Email Alerts:**

•  Configure the flow to send email notifications to admins if errors occur.

  

**3. Avoiding Duplicate Campaign Members**

  

•  **Check for Existing Membership:**

•  Use a **Get Records** element to check if the Lead/Contact is already a member of the Campaign before creating a new Campaign Member record.

  

**4. Handling Updates**

  

•  **Trigger on Update Events:**

•  Ensure the flow triggers when the HubSpot CID field is updated, in case the cid changes for existing Leads/Contacts.

  

**5. Permissions**

  

•  **User Permissions:**

•  Ensure that the user context under which the flow runs has the necessary permissions to create Campaign Member records.

  

**6. Bulk Data Handling**

  

•  **Large Data Volumes:**

•  Be cautious with bulk data operations, as flows have limits on the number of records processed.

•  **Batch Apex (Advanced):**

•  For large-scale operations, consider using Batch Apex.



**Alternative Approach: Use Apex Triggers (Advanced)**

  

If you require more complex logic or need to handle higher volumes of data, you could implement an **Apex Trigger** on the Lead and Contact objects.

•  **Pros:**

•  Greater control and flexibility.

•  Can handle complex logic and error handling.

•  **Cons:**

•  Requires Salesforce development expertise.

•  Involves writing and maintaining code.


**Summary**

  

•  **HubSpot Limitations:**

•  The standard HubSpot workflow actions cannot dynamically assign Salesforce campaigns based on the cid.

•  **Salesforce Flow Builder Solution:**

•  Use Salesforce Flow Builder to automate adding Leads/Contacts to Campaigns based on the cid value.

•  **Benefits:**

•  **Campaign-Agnostic:** Works for any campaign without manual updates.

•  **Scalable:** One flow can handle all cases.

•  **Maintainable:** Reduces the need for multiple workflows or manual processes.
