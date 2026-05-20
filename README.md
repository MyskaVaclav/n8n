# BESS n8n Workflows

This repository contains exported JSON workflow files for implementing automated BESS data acquisition and evaluation in n8n.

The workflows read operational data from the MARFY system, store the collected values in Google Sheets, evaluate predefined operating limits, and generate a daily PDF report.

## Repository Contents

- `Proces vyčítání dat.json` – workflow for data acquisition
- `Proces vyhodnocení dat.json` – workflow for daily data evaluation and report generation
- `Retry logika.json` – workflow for automatic retry handling after workflow execution failures

## Data Acquisition Workflow

The `Proces vyčítání dat` workflow reads BESS operational data from the MARFY system every 30 minutes.

The workflow logs into the MARFY web interface, extracts the required antiforgery token and session cookies, requests data for the last completed 30-minute time window, transforms the returned data into a structured format, and appends the results to Google Sheets.

The stored values include:

- minimum and maximum cell voltage
- minimum and maximum module voltage
- minimum and maximum cell temperature
- minimum and maximum module temperature
- state of charge (SOC)
- identifiers of cells and modules with extreme values

## Data Evaluation Workflow

The `Proces vyhodnocení dat` workflow runs once every 24 hours.

It loads the stored data from Google Sheets, evaluates the previous day according to the `Europe/Prague` time zone, checks operating limits, identifies violation intervals, and creates a summary of BESS behavior.

The workflow evaluates:

- cell and module temperature limits
- SOC limits
- safe cell voltage range
- voltage differences between cells and modules
- temperature differences between cells and modules
- duration of limit violations
- occurrence of cells and modules at extreme values

The result is rendered into an HTML template, converted into a PDF report, uploaded to Google Drive, and sent by e-mail.

## Retry Workflow

The `Retry logika` workflow provides automatic retry handling for failed workflow executions.

The workflow is triggered using the n8n `Error Trigger` node whenever the main data acquisition workflow fails. It calculates the last completed 30-minute acquisition window, waits 10 minutes, and then re-executes the data acquisition workflow with the original frozen time window parameters.

This approach prevents data loss caused by temporary connectivity issues, unavailable services, or short-term failures of the MARFY system.

The retry workflow preserves:

- original start timestamp
- original end timestamp
- acquisition window alignment
- correct 30-minute data intervals

The retry process is fully automated and integrated directly into the n8n error workflow system.

## Technologies Used

- n8n
- JavaScript
- Google Sheets
- Google Drive
- Gmail
- HTML/CSS
- PDF

## Requirements

Before running the workflows, configure the required n8n credentials:

- Google Sheets account
- Google Drive account
- Gmail account
- CustomJS account for HTML-to-PDF conversion

The MARFY login credentials, Google Sheets document ID, Google Drive folder ID, and target e-mail address must also be configured in the imported workflows.

## Import into n8n

1. Open n8n.
2. Import the JSON workflow files.
3. Configure all required credentials.
4. Check the MARFY login settings.
5. Check the Google Sheets and Google Drive IDs.
6. Activate the workflows.