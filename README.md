# Airtable-Connected Dynamic Form Builder

This repository contains a MERN-style application to build dynamic forms from Airtable tables, apply conditional logic, save responses to Airtable and MongoDB, and keep the DB in sync using Airtable webhooks.

This README contains:
- Setup and run instructions
- Airtable OAuth setup guide
- Data model explanation
- Conditional logic explanation
- Webhook configuration
- How to run tests

Quick summary of what's included in this scaffold:
- Backend: Express + Mongoose (models provided)
- Utility: conditional logic function + unit tests (Jest)
- API contract and route outlines
- sample.env.example for required env vars

Note: UI is not included in this scaffold; you'll wire a React frontend using the API endpoints described below.

## Setup (backend)

1. Copy env example:
   cp sample.env.example .env

2. Install dependencies:
   npm install
   cd frontend && npm install (if you create the frontend)

3. Run backend in dev:
   npm run dev

4. Run tests:
   npm test

## Airtable OAuth Setup (high level)

1. Create an Airtable developer app and set OAuth redirect URI to your backend callback URL:
   e.g. https://your-backend.example.com/auth/airtable/callback

2. Add client ID and secret to your .env:
   AIRTABLE_CLIENT_ID, AIRTABLE_CLIENT_SECRET

3. OAuth flow (high-level):
   - GET /auth/airtable -> redirect user to Airtable authorize URL
   - Airtable redirects back to /auth/airtable/callback?code=...
   - Exchange code for access token & refresh token
   - Save user profile, tokens, timestamp in MongoDB

## API Contract (important endpoints)

Auth / OAuth
- GET /auth/airtable
  - Redirects user to Airtable OAuth authorize URL.
- GET /auth/airtable/callback?code=...
  - Exchange code for tokens, fetch user profile, save in DB, create session / JWT.

Forms / Builder
- POST /api/forms
  - Create a form definition (owner from auth)
  - Body: { baseId, tableId, title?, questions: [...] }
- GET /api/forms/:formId
  - Get stored form definition
- GET /api/user/bases
  - Use user's Airtable token to list bases (used by builder UI)
- GET /api/user/bases/:baseId/tables
  - List tables in a base
- GET /api/airtable/:baseId/:tableId/fields
  - Fetch Airtable fields (backend validates supported field types)

Form Viewer & Submissions
- GET /api/forms/:formId  (reuse above)
- POST /api/forms/:formId/submit
  - Validate against form schema (required fields, choice membership)
  - Create a record in Airtable via API
  - Save response in MongoDB with airtableRecordId

Responses
- GET /api/forms/:formId/responses
  - Returns all responses stored in MongoDB for that form

Webhooks
- POST /api/webhooks/airtable
  - Handle Airtable events: record.updated, record.deleted
  - Update local DB accordingly (mark deletedInAirtable for deletes)

## Data Models (summary)

User
- airtableUserId: string
- profile: {...}
- accessToken, refreshToken
- lastLoginAt: Date

Form
- owner: ObjectId (User)
- title: string
- baseId: string
- tableId: string
- questions: [
  {
    questionKey: string,
    fieldId: string,         // Airtable field id
    label: string,
    type: 'short_text'|'long_text'|'single_select'|'multi_select'|'attachment',
    required: boolean,
    options?: string[],      // for single/multi select
    conditionalRules?: { logic: 'AND'|'OR', conditions: [{ questionKey, operator, value }] }
  }
]

Response
- formId
- airtableRecordId
- answers (JSON map questionKey -> value)
- createdAt, updatedAt
- deletedInAirtable: boolean

## Conditional Logic

The pure function `shouldShowQuestion(rules, answersSoFar)` is implemented in `utils/conditionalLogic.ts`. It returns true if rules is null. It evaluates operators: equals, notEquals, contains. It tolerates missing values.

## Webhook configuration

- Configure Airtable webhook to POST to your backend endpoint:
  POST https://your-backend.example.com/api/webhooks/airtable
- Verify and secure using your AIRTABLE_WEBHOOK_SECRET; validate incoming signature if provided by Airtable.

## Running tests

- Tests are implemented with Jest for the conditional logic function.
- Run:
  npm test

## What's next
- Implement OAuth routes and session/JWT creation
- Implement Airtable API calls: list bases/tables/fields, create record
- Create a minimal React UI for Form Builder and Form Viewer
- Deploy backend and frontend
