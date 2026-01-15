# Google Apps Script Setup for Form Submissions

This guide will help you set up a Google Apps Script to receive form submissions from your website, save them to a Google Sheet, and send you email notifications.

## Step 1: Create a Google Sheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Create a new spreadsheet
3. Name it something like "Oceanside Blinds - Leads"
4. In Row 1, add these headers:
   - A1: `Timestamp`
   - B1: `Type`
   - C1: `Email`
   - D1: `Name`
   - E1: `Phone`
   - F1: `Appointment Type`
   - G1: `Preferred Date`
   - H1: `How Heard About Us`
   - I1: `Message`

## Step 2: Create the Google Apps Script

1. In your Google Sheet, go to **Extensions > Apps Script**
2. Delete any code in the editor
3. Paste the following code:

```javascript
// Configuration - Update this email address
const NOTIFICATION_EMAIL = 'coastalreservellc@outlook.com';
const SHEET_NAME = 'Oceanside Blinds - Leads'; // Change if your sheet has a different name

// Handle both GET and POST (redirects can convert POST to GET)
function doGet(e) {
  return handleRequest(e);
}

function doPost(e) {
  return handleRequest(e);
}

function handleRequest(e) {
  try {
    // Get sheet - try by name first, fall back to first sheet
    let sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_NAME);
    if (!sheet) {
      sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    }

    // Get form data from parameters (handle case where e or e.parameter is undefined)
    const data = (e && e.parameter) ? e.parameter : {};

    // Skip if no data received
    if (!data.email && !data.type) {
      return ContentService
        .createTextOutput(JSON.stringify({ success: false, error: 'No data received' }))
        .setMimeType(ContentService.MimeType.JSON);
    }

    // Add row to sheet
    const row = [
      data.timestamp || new Date().toISOString(),
      data.type || 'unknown',
      data.email || '',
      data.name || '',
      data.phone || '',
      data.appointmentType || '',
      data.preferredDate || '',
      data.hearAbout || '',
      data.message || ''
    ];

    sheet.appendRow(row);

    // Send email notification
    sendNotificationEmail(data);

    return ContentService
      .createTextOutput(JSON.stringify({ success: true }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    // Log error for debugging
    console.error('Error in handleRequest:', error);
    return ContentService
      .createTextOutput(JSON.stringify({ success: false, error: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function sendNotificationEmail(data) {
  let subject, body;

  if (data.type === 'newsletter') {
    subject = '📧 New Newsletter Signup - Oceanside Blinds';
    body = `
New newsletter subscription!

Email: ${data.email}
Time: ${data.timestamp}

---
This is an automated message from your website.
    `;
  } else if (data.type === 'contact') {
    subject = '📞 New Contact Form Submission - Oceanside Blinds';
    body = `
New contact form submission!

Name: ${data.name}
Email: ${data.email}
Phone: ${data.phone}
Message: ${data.message}

Time: ${data.timestamp}

---
This is an automated message from your website.
    `;
  } else if (data.type === 'estimate') {
    subject = '🏠 New Consultation Request - Oceanside Blinds';
    body = `
New consultation request!

Name: ${data.name}
Email: ${data.email}
Phone: ${data.phone}
Appointment Type: ${data.appointmentType || 'Not specified'}
Preferred Date: ${data.preferredDate || 'Not specified'}
How They Heard About Us: ${data.hearAbout || 'Not specified'}
Message: ${data.message || 'No message'}

Time: ${data.timestamp}

---
This is an automated message from your website.
    `;
  } else {
    subject = '🔔 New Form Submission - Oceanside Blinds';
    body = `
New form submission!

Data: ${JSON.stringify(data, null, 2)}

---
This is an automated message from your website.
    `;
  }

  MailApp.sendEmail({
    to: NOTIFICATION_EMAIL,
    subject: subject,
    body: body
  });
}

// Test function - run this to verify the full flow (sheet + email) works
function testFullSubmission() {
  const testData = {
    parameter: {
      type: 'estimate',
      name: 'Test User',
      email: 'test@example.com',
      phone: '555-1234',
      appointmentType: 'In-home estimate',
      preferredDate: '2025-02-01',
      hearAbout: 'Google',
      message: 'This is a test submission',
      timestamp: new Date().toISOString()
    }
  };

  const result = handleRequest(testData);
  Logger.log('Result: ' + result.getContent());
}

// Test function - run this to verify email works
function testEmail() {
  sendNotificationEmail({
    type: 'newsletter',
    email: 'test@example.com',
    timestamp: new Date().toISOString()
  });
}
```

4. Click **Save** (disk icon) and name the project "Oceanside Blinds Form Handler"

## Step 3: Deploy as Web App

1. Click **Deploy > New deployment**
2. Click the gear icon next to "Select type" and choose **Web app**
3. Fill in:
   - Description: "Form submission handler"
   - Execute as: **Me**
   - Who has access: **Anyone**
4. Click **Deploy**
5. Click **Authorize access** and follow the prompts to allow the script
6. **Copy the Web App URL** - it will look like:
   `https://script.google.com/macros/s/AKfycbw.../exec`

## Step 4: Update Your Website Config

1. Open `config.json` in your website folder
2. Find the line with `"googleScriptUrl": "PLACEHOLDER_GOOGLE_SCRIPT_URL"`
3. Replace `PLACEHOLDER_GOOGLE_SCRIPT_URL` with your Web App URL

Example:
```json
"googleScriptUrl": "https://script.google.com/macros/s/AKfycbwXXXXXXXXXXXXXXXXXXXXXXXXXXX/exec",
```

## Step 5: Test It

1. Open your website
2. Enter an email in the newsletter signup form
3. Submit the form
4. Check your Google Sheet - a new row should appear
5. Check your email - you should receive a notification

## Troubleshooting

**Form submits but nothing happens:**
- Make sure the Web App URL is correct in config.json
- Check that the script is deployed as "Anyone" can access

**No email received:**
- Run the `testEmail()` function in Apps Script to verify email works
- Check your spam folder
- Make sure the NOTIFICATION_EMAIL is correct in the script

**To update the script after changes:**
1. Make your changes in the Apps Script editor
2. Click **Deploy > Manage deployments**
3. Click the pencil icon to edit
4. Change version to "New version"
5. Click **Deploy**

## Adding More Forms (Optional)

The script already supports different form types. To add a contact form or estimate request form to your site, submit data with these fields:

```javascript
// Contact form
{
  type: 'contact',
  email: 'user@example.com',
  name: 'John Doe',
  phone: '555-1234',
  message: 'I need help with...',
  timestamp: new Date().toISOString()
}

// Consultation/Estimate request form
{
  type: 'estimate',
  email: 'user@example.com',
  name: 'John Doe',
  phone: '555-1234',
  appointmentType: 'In-home estimate',  // or 'Showroom appointment'
  preferredDate: '2025-02-15',
  hearAbout: 'Google',  // or 'Yelp', 'TV', 'Referral', 'Other: Friend recommendation'
  message: 'I have 5 windows in my living room...',
  timestamp: new Date().toISOString()
}
```
