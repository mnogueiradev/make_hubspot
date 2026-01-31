# Make Blueprint – HubSpot + Google Sheets

This folder contains a Make (formerly Integromat) scenario blueprint exported as JSON.

- **Blueprint file**: `Make_HubSpot.json`
- **Scenario name in the export**: `Integration Google Sheets`

## What this scenario does
The scenario watches for new/updated rows in a Google Sheet, fetches the lead website, converts the HTML to text, asks an AI model to score the lead, and then routes the lead:

- If the AI response contains a score of `10`, it creates a Deal in HubSpot and updates the Google Sheet row as approved.
- Otherwise, it updates the Google Sheet row as rejected.

There is also an optional HTTP notification step (a GET request to a `trigger.macrodroid.com` URL) used as a “success” alert.

## Google Sheet expected columns
The blueprint is configured to read a table with headers in columns **A to F**:

- **A**: Company name (`Nome da Empresa`)
- **B**: Website URL (`Site (URL)`)
- **C**: Contact name (`Nome do Contato`)
- **D**: Status (written by Make)
- **E**: AI analysis (written by Make)
- **F**: Email draft (not filled in by the current blueprint)

## Modules (high-level flow)
- **Google Sheets – Watch Rows**: watches the sheet for rows.
- **HTTP – Make a request (GET)**: requests the website URL from column B.
- **Tools/RegExp – HTML to Text**: converts the page HTML into plain text.
- **AI/ML API – Get Model Response**: runs an LLM prompt that analyzes the site text and returns a result.
- **Router**:
  - **Route “Lead Rico”** (rich lead): creates a HubSpot Deal, then updates the sheet status to approved, then sends the optional HTTP notification.
  - **Route “Lead Pobre”** (poor lead): updates the sheet status to rejected and writes the AI response into the “AI analysis” column.

## Important note about the prompt
In the exported blueprint, the AI prompt includes a **test rule** forcing the model to always respond with “company is Large” and “Score: 10”, regardless of the website content.

If you want real scoring, remove that test rule in the AI module prompt.

## Prerequisites
You will need accounts/connections for:

- **Google Sheets** (Google connection in Make)
- **HubSpot CRM** (HubSpot connection in Make)
- **AI/ML API** (the module in the blueprint is `aimlapi:GetModelResponse`)

## How to import the blueprint into Make
1. Open **Make**.
2. Go to **Scenarios**.
3. Click **Create a new scenario**.
4. Use the **Import Blueprint** option.
5. Select `Make_HubSpot.json`.

After import, Make will show missing connections. Reconnect each module.

## Required configuration after import
Because blueprints store connection IDs from the exporting account, you must reconfigure these items:

- **Google Sheets modules** (`watchRows`, `updateRow`)
  - Set your Google connection.
  - Select the spreadsheet and sheet tab.
  - Confirm headers and the `A1:F1` header range (or adapt to your sheet).

- **HubSpot module** (`hubspotcrm:createDeal`)
  - Set your HubSpot connection.
  - Confirm pipeline/stage settings if your HubSpot account differs.

- **AI/ML API module** (`aimlapi:GetModelResponse`)
  - Set your AI/ML API connection.
  - Confirm the model id (currently `gpt-3.5-turbo`) is available in your provider.

- **Optional notification HTTP module**
  - The URL is currently hard-coded to a `trigger.macrodroid.com` endpoint and includes a phone number + message in query parameters.
  - Replace or remove this module if you do not use it.

## How to test
- Add a new row to the configured Google Sheet with:
  - Company name
  - Website URL
  - Contact name
- Run the scenario once manually.
- Check:
  - Column **D** (Status) updated to approved/rejected
  - Column **E** (AI analysis) updated when rejected
  - HubSpot deal created for approved leads

## Customization ideas
- Write the AI response into the sheet also for approved leads.
- Generate an email draft and write it into column **F**.
- Replace the “contains `10`” filter with a structured JSON output and numeric comparison.
- Add deduplication in HubSpot (search company/domain before creating a deal).
