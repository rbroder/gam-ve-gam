# הוראות הגדרה: טופס → Google Sheet → GROW

מסמך זה מסביר איך לחבר את טופס ההרשמה לגיליון Google שמשמש כ-CRM, ולהפנות את המשתמש לתשלום ב-GROW לאחר השליחה.

זמן הקדשה: כ-15 דקות, פעם אחת.

---

## שלב 1: יצירת Google Sheet

1. היכנסו ל-[sheets.google.com](https://sheets.google.com) ופתחו גיליון חדש.
2. שנו את שמו ל-״הרשמות לסדנה - אני מנהל.ת, וגם״.
3. בשורה הראשונה, הוסיפו את הכותרות הבאות (עמודה אחר עמודה):

```
Timestamp | Name | Phone | Email | Marketing Consent
```

---

## שלב 2: יצירת Apps Script

1. בתוך הגיליון, לחצו על תפריט **Extensions → Apps Script** (״הרחבות → Apps Script״).
2. ייפתח חלון חדש עם קוד ריק. מחקו את כל הקוד הקיים.
3. הדביקו את הקוד הבא:

```javascript
function doPost(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    const data = JSON.parse(e.postData.contents);

    sheet.appendRow([
      data.timestamp || new Date().toISOString(),
      data.name || '',
      data.phone || '',
      data.email || '',
      data.marketingConsent || 'no'
    ]);

    return ContentService
      .createTextOutput(JSON.stringify({ result: 'success' }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ result: 'error', message: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet() {
  return ContentService.createTextOutput('OK');
}
```

4. שמרו (Ctrl+S או Cmd+S). תנו לפרויקט שם, למשל ״Workshop Form Handler״.

---

## שלב 3: פריסה כ-Web App

1. בפינה הימנית העליונה, לחצו על **Deploy → New deployment** (״פריסה → פריסה חדשה״).
2. ליד ה-Settings (גלגל שיניים), בחרו **Web app**.
3. הגדירו:
   - **Description:** Workshop form handler
   - **Execute as:** Me (your-email@gmail.com)
   - **Who has access:** **Anyone** (חובה - אחרת הטופס לא יוכל לכתוב לגיליון)
4. לחצו **Deploy**.
5. בפעם הראשונה תופיע בקשת אישור - לחצו **Authorize access**, בחרו את חשבון Google שלכם, ואז ״Advanced → Go to (Project name)״ → **Allow**.
6. תקבלו **Web app URL** - זה הקישור שצריך. העתיקו אותו (משהו כמו `https://script.google.com/macros/s/AKfycb.../exec`).

---

## שלב 4: הדבקה בקובץ `index.html`

1. פתחו את `index.html` בעורך טקסט.
2. חפשו את השורה:
   ```javascript
   const APPS_SCRIPT_URL = 'PASTE_YOUR_APPS_SCRIPT_URL_HERE';
   ```
3. החליפו את `PASTE_YOUR_APPS_SCRIPT_URL_HERE` בקישור שהעתקתם.
4. חפשו את השורה:
   ```javascript
   const GROW_PAYMENT_URL = 'PASTE_YOUR_GROW_PAYMENT_URL_HERE';
   ```
5. החליפו ב-URL של עמוד התשלום שלכם ב-GROW.
6. שמרו את הקובץ והעלו לאתר.

---

## בדיקה

1. פתחו את האתר.
2. מלאו את הטופס בפרטי בדיקה (כולל סימון תיבת ההסכמה).
3. שלחו.
4. וודאו ש:
   - השורה הופיעה בגיליון Google, עם `yes` בעמודת Marketing Consent.
   - הופניתם לעמוד התשלום ב-GROW.

---

## עדכון הסקריפט בעתיד

אם תרצו לשנות את הקוד (להוסיף שדה, להחליף סדר וכו׳):

1. ערכו את הקוד ב-Apps Script.
2. שמרו.
3. **חשוב:** Deploy → Manage deployments → לחצו על אייקון העיפרון → Version: **New version** → Deploy.
4. ה-URL נשאר זהה, אין צורך לעדכן את ה-HTML.

---

## טיפים

- **התראות במייל:** כדי לקבל הודעה במייל על כל הרשמה חדשה, בגיליון לחצו **Tools → Notification settings → Notify me when any changes are made**.
- **גישה מטלפון:** אפליקציית Google Sheets לטלפון מאפשרת לכם לראות הרשמות בזמן אמת.
- **ייצוא ל-Excel/CRM:** בכל רגע **File → Download → Microsoft Excel (.xlsx)**.

---

## פתרון בעיות

**הטופס לא נשלח / שגיאה ב-Console:**
- ודאו שה-URL מתחיל ב-`https://script.google.com/`.
- ודאו שב-Deploy בחרתם **Who has access: Anyone**.

**שורות לא מופיעות בגיליון:**
- חזרו לסקריפט ולחצו **Deploy → Manage deployments → ערוך** → ודאו שהגרסה האחרונה פרוסה.
- אם שיניתם את הקוד, חובה **לפרוס מחדש כגרסה חדשה** (New version).

**לא הופניתי לתשלום:**
- ודאו שה-URL של GROW מלא ותקין (מתחיל ב-`https://`).
