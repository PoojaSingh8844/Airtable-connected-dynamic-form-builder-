Airtable-Connected Dynamic Form Builder 
Estimated Time: 2–3 Days
Tech Stack: MongoDB, Express, React, Node.js

📌 Objective
Build a small full-stack application that:
Allows users to log in using Airtable OAuth.


Lets users create custom form definitions using Airtable fields.


Applies conditional logic to form questions.


Saves form responses into both Airtable and your database.


Displays all saved responses in the app.


Syncs your database when Airtable data changes using Webhooks.


UI design is not the priority — focus on correctness, data modeling, and logic.

✅ Core Features
1. Log in with Airtable using OAuth
Implement the Airtable OAuth flow.
After successful login:
Save the user’s Airtable profile data and access token in MongoDB.


Store:


Airtable userId or basic profile


OAuth access token & refresh token
Login timestamp



2. Create a Form (Form Builder)
Authenticated user should be able to:
Select an Airtable Base from their account.


Select a Table from that Base.


Fetch all fields from that table.


Choose which fields will appear in the form.


Rename question labels. (optional)


Mark fields as required or optional.  (optional)


Define conditional logic rules (see below).


Store the form schema in MongoDB, including:
Form owner


Airtable base id


Airtable table id


List of questions:


internal questionKey


Airtable fieldId


label


type


required


conditional rules


No fancy UI required — simple inputs are fine.

3. Supported Question Types
Only support the following Airtable field types:
Short text


Long text


Single select


Multi select


Attachment (file upload)


You can skip all other Airtable field types (checkbox, date, formula, etc.)
On the backend:
Reject unsupported Airtable field types automatically.


Validate incoming form data against Airtable rules.



4. Apply Conditional Logic
Allow the user to configure visibility rules for each question.
Example:
 Show githubUrl only if role = Engineer
Each question can contain:
Multiple conditions


Logic operator to combine them: AND or OR


Condition structure example:
type Operator = "equals" | "notEquals" | "contains";

interface Condition {
  questionKey: string;
  operator: Operator;
  value: any;
}

Rules object example:
interface ConditionalRules {
  logic: "AND" | "OR";
  conditions: Condition[];
}

Implement a pure function:
function shouldShowQuestion(
  rules: ConditionalRules | null,
  answersSoFar: Record<string, any>
): boolean

Rules:
If rules = null, return true


Evaluate each condition


Combine using AND / OR


Missing values should not crash evaluation


This function must work without UI dependencies and should be testable.

5. View and Fill the Form (Form Viewer)
Route example:
 /form/:formId
Requirements:
Form definition should load from your backend using the form ID.


Display fields and labels as configured.


Apply conditional logic in real-time as the user fills answers.


Validate required fields before allowing submission.


UI can be minimal. Logic is more important than styling.

6. Save Responses to Airtable + Database
On form submission:
Validate responses against the form definition:


Required fields


Single-select choices


Multi-select array options


Save a new Airtable record to the selected table.


Save the response in MongoDB.


Suggested fields for DB:
formId


airtableRecordId


answers (raw JSON)


createdAt


updatedAt

7. Response Listing (Database Only)
Add a simple responses view:
 Route example: /forms/:formId/responses
Backend endpoint:
 GET /forms/:formId/responses
This page should list all responses stored in MongoDB:
submission ID


created timestamp


status


compact answers preview


Do not fetch directly from Airtable.

8. Keep DB in Sync with Airtable Using Webhooks
If Airtable responses change (update/edit/delete), your database should update.
Hint: Airtable webhooks.
When Airtable sends a webhook event:
 Call a backend endpoint:
 POST /webhooks/airtable
Handle events:
When Airtable record is updated:


Update the corresponding DB record


When a record is deleted:


Mark DB record as deletedInAirtable


Do not hard delete locally — only flag it.

🎁 Bonus (Optional)
Basic form validation in UI


Dashboard of all created forms


Form preview mode


Export responses (CSV/JSON)





📦 Deliverables

Live deployed version
Frontend on Vercel/Netlify
Backend on Render/Railway

GitHub repository containing:
Frontend (React)


Backend (Express + MongoDB)


sample.env.example including:


Airtable OAuth keys


MongoDB connection


Webhook config


README.md
Include:
Setup instructions (frontend & backend)


Airtable OAuth setup guide


Data model explanation


Conditional logic explanation


Webhook configuration


How to run the project


Screenshots or demo video 

🧰 Helpful Resources
🔗 Create Airtable Account
🔗 Airtable OAuth Quickstart 
🔗 Airtable REST API
🔗 Airtable Webhooks


