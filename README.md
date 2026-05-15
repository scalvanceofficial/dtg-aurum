# Guide: Integrating Google Sheets with Enquiry Forms

This guide explains how the integration between your landing page forms and Google Sheets works, and how to maintain it.

## 1. Google Sheets Setup
- **Spreadsheet Name:** Any name (e.g., "DTG Contact Form").
- **Sheet (Tab) Name:** Must be exactly `DTG AURUM`.
- **Headers (Row 1):**
  - Column A: `Name`
  - Column B: `Number`
  - Column C: `Email ID`

## 2. Google Apps Script Code
Use this code in your Google Apps Script editor. This version is designed to be "bulletproof" and handles multiple data formats.

```javascript
function doPost(e) {
  try {
    var ss = SpreadsheetApp.getActiveSpreadsheet();
    var sheet = ss.getSheetByName("DTG AURUM");
    
    // 1. Extract data (handles both JSON and URL-encoded formats)
    var data = e.parameter; 
    if (e.postData && e.postData.contents) {
      try {
        data = JSON.parse(e.postData.contents);
      } catch (err) {
        // Not JSON, continue with e.parameter
      }
    }

    // 2. Append row to the sheet
    sheet.appendRow([
      data.name || "N/A",
      data.number || "N/A",
      data.email || "N/A"
    ]);

    // 3. Return a success response
    return ContentService
      .createTextOutput(JSON.stringify({ status: "success" }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch(error) {
    // Return error details if something goes wrong
    return ContentService
      .createTextOutput(JSON.stringify({ status: "error", message: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

## 3. Crucial Deployment Settings
Whenever you update the code, you **must** redeploy correctly:
1.  Click **Deploy** > **Manage deployments**.
2.  Click the **Pencil icon** (Edit).
3.  Choose **New Version** in the dropdown.
4.  **Execute as:** Set to **"Me"** (your email).
5.  **Who has access:** Set to **"Anyone"**.
6.  Click **Deploy**.
7.  Copy the **Web App URL** and ensure it matches the `SCRIPT_URL` in your `index.html`.

## 4. Frontend Integration (index.html)
The forms are integrated using a shared class and a single script block at the end of `index.html`.

### Form Requirements
Every form must have:
1.  The class `enquiry-form-shared` (for static forms) or the ID `mainEnquiryForm` (for the popup).
2.  Input fields with `name="name"`, `name="number"`, and `name="email"`.

### Submission Logic
```javascript
// Data is gathered from the form fields
const data = {
    name: currentForm.name.value,
    number: currentForm.number.value,
    email: currentForm.email.value
};

// Data is converted to URL parameters for maximum reliability
const params = new URLSearchParams(data);

// Request is sent to Google Apps Script
fetch(SCRIPT_URL, {
    method: "POST",
    mode: "no-cors",
    headers: {
        "Content-Type": "application/x-www-form-urlencoded"
    },
    body: params.toString()
});
```

## 5. Troubleshooting
- **Alert shows but no data:** Ensure the sheet name is exactly `DTG AURUM` (case sensitive).
- **Nothing happens on click:** Open the browser console (F12) to check for errors.
- **Old data keeps appearing:** Ensure you deployed as a **"New Version"** in Apps Script.
