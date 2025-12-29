# Google Sheets Integration Setup for CareerPivot Waitlist

## Step-by-Step Instructions

### STEP 1: Create a Google Form
1. Go to [Google Forms](https://forms.google.com)
2. Click **"+ Blank"** to create a new form
3. Name it: "CareerPivot Waitlist"
4. Add these fields:
   - **Email** (Short answer, Required)
   - **Phone Number** (Short answer, Required)
   - **Current Role** (Short answer, Not required)
   - **Dream Role / Direction** (Short answer, Not required)
   - **SMS Consent** (Checkboxes or Multiple choice: "I consent to SMS messages")

---

### STEP 2: Connect Form to Google Sheet
1. In your Google Form, click the **"Responses"** tab
2. Click the **"Create Spreadsheet"** icon (green sheet icon)
3. Choose:
   - **Create in a new spreadsheet** (recommended)
   - Name it: "CareerPivot Waitlist Responses"
4. Click **"Create"**
5. A new Google Sheet will open automatically with response headers

---

### STEP 3: Get Your Form Submission URL
1. Go back to your Google Form
2. Click the **"⋮"** (three dots menu) in the top right
3. Select **"Get pre-filled link"**
4. This shows you the form URL structure

**Alternative Method (Recommended for API):**
1. Open the form in edit mode
2. Click **"⋮"** → **"Get embed code"** or inspect the form
3. The form ID is in the URL: `https://docs.google.com/forms/d/{FORM_ID}/viewform`

---

### STEP 4: Set Up Form Submission via JavaScript

The easiest method is using **Google Apps Script + CORS proxy** or **Formspree** (simpler).

#### OPTION A: Using Formspree (EASIEST - Recommended)

1. Go to [Formspree.io](https://formspree.io)
2. Sign up with your Google account
3. Click **"New Form"**
4. Select **"Create a new form"**
5. Name: "CareerPivot Waitlist"
6. It will give you an endpoint URL like: `https://formspree.io/f/xyzabc123`
7. Copy this URL

**Then in `index.html`, find the form submission code (around line 1050) and replace:**

```javascript
// In the form submit event listener, replace:
try {
    // Option 1: If using Formspree
    const response = await fetch('https://formspree.io/f/YOUR_FORM_ID', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
    });
    
    if (response.ok) {
        console.log('Form submitted successfully');
        successMessage.style.display = 'block';
        form.style.display = 'none';
        setTimeout(() => {
            form.reset();
            form.style.display = 'block';
            successMessage.style.display = 'none';
            submitBtn.disabled = true;
        }, 5000);
    } else {
        alert('There was an error. Please try again.');
    }
}
```

Replace `YOUR_FORM_ID` with your actual Formspree ID (e.g., `xyzabc123`).

---

#### OPTION B: Using Google Apps Script (Advanced)

1. Go to [script.google.com](https://script.google.com)
2. Click **"New Project"**
3. Replace the default code with:

```javascript
function doPost(e) {
  const sheet = SpreadsheetApp.getActiveSheet();
  const data = e.parameter;
  
  sheet.appendRow([
    new Date(),
    data.email,
    data.phone,
    data.role || '',
    data.dream || '',
    data.consent === 'on' ? 'Yes' : 'No'
  ]);
  
  return ContentService.createTextOutput(JSON.stringify({
    success: true,
    message: 'Data received'
  })).setMimeType(ContentService.MimeType.JSON);
}
```

4. Click **"Deploy"** → **"New Deployment"**
5. Select **"Type"** → **"Web app"**
6. Set:
   - Execute as: Your Google account
   - Who has access: "Anyone"
7. Click **"Deploy"**
8. Copy the **Deployment URL** (looks like: `https://script.googleapis.com/macros/d/{SCRIPT_ID}/usercript`)

**Then in `index.html`, update the form submission:**

```javascript
const response = await fetch('YOUR_DEPLOYMENT_URL', {
    method: 'POST',
    body: new URLSearchParams(formData)
});
```

---

### STEP 5: Update the HTML Form

Find the form submission section in `index.html` (around line 1050-1080) and uncomment/update:

```javascript
try {
    // Using Formspree
    const response = await fetch('https://formspree.io/f/YOUR_FORMSPREE_ID', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
    });

    if (response.ok) {
        successMessage.style.display = 'block';
        form.style.display = 'none';
        
        setTimeout(() => {
            form.reset();
            form.style.display = 'block';
            successMessage.style.display = 'none';
            submitBtn.disabled = true;
        }, 5000);
    } else {
        throw new Error('Form submission failed');
    }
} catch (error) {
    console.error('Error submitting form:', error);
    alert('There was an error submitting your form. Please try again.');
}
```

---

### STEP 6: Test the Integration

1. Open `index.html` in a browser
2. Scroll to the **"Start Your Transition Today"** section
3. Fill out the form:
   - Email: `test@example.com`
   - Phone: `(555) 123-4567`
   - Current Role: `Finance Manager` (optional)
   - Dream Role: `Product Manager` (optional)
   - Check the SMS consent checkbox
4. Click **"Join Waitlist"**
5. You should see: ✓ **"Welcome! Check your email for next steps."**

---

### STEP 7: Verify Data in Google Sheets (Formspree)

1. Log in to [Formspree.io](https://formspree.io)
2. Go to your form
3. Click **"Submissions"** tab
4. You should see all submitted data

**OR** (if using Google Apps Script):
1. Go to [script.google.com](https://script.google.com)
2. Find your deployment
3. Click on the linked Google Sheet
4. View all responses in the spreadsheet

---

## Quick Comparison: Which Option to Use?

| Feature | Formspree | Google Apps Script |
|---------|-----------|-------------------|
| **Setup Time** | 5 min | 15 min |
| **Cost** | Free tier (50 submissions/month) | Free (unlimited) |
| **Email Confirmations** | ✓ Built-in | ✗ Need to configure |
| **Custom Logic** | Limited | ✓ Full control |
| **CORS Issues** | ✗ No issues | ✗ No issues |

**RECOMMENDATION: Use Formspree for quick setup, Google Apps Script if you need unlimited submissions.**

---

## Troubleshooting

### "Form submission failed" error
- Check your Formspree ID is correct
- Ensure HTTPS (if on a secure domain)
- Check browser console (F12) for CORS errors

### Data not appearing in Google Sheets
- Verify Google Apps Script deployment is set to "Anyone"
- Check the script logs (Apps Script Editor → "Executions" tab)
- Ensure form field names match the `e.parameter` keys

### Email validation not working
- Check that email field has `type="email"`
- Browser will block submission if invalid

---

## Next Steps

After confirming submissions are working:

1. **Add auto-reply email** (Formspree Pro feature or custom Gmail)
2. **Set up SMS notifications** (Twilio integration)
3. **Create automation** (Zapier/Make to send welcome email)
4. **Track analytics** (Google Analytics 4 for form interactions)
5. **Build backend** (Node.js/Python to process signups)

---

**Questions?** Check the inline code comments in `index.html` around line 1050.
