# PlugIn — מערכת אוטומציה לעסק אלקטרוניקה

מערכת ניהול ואוטומציה מקצה-לקצה לעסק אלקטרוניקה קטן (**PlugIn**), הבנויה על n8n,
Airtable, OpenAI, Qdrant, Google Drive, Telegram ו-Gmail, עם אפליקציית ניהול
("קוקפיט בוקר") שנבנתה ב-Lovable.

המערכת מכסה את כל מסע העסק: קליטת לידים ומכירות, שירות לקוחות חכם (RAG),
הפקת חשבוניות אוטומטית, אימות מסמכי מס, ופאנל ניהול אחד שממנו בעל/ת העסק
רואה הכול ומתפעל את המערכת.

---

## 🔗 קישורים

| רכיב | קישור |
|---|---|
| אפליקציית הניהול (Lovable) | https://pixel-perfect-snap-684.lovable.app |
| בסיס הנתונים (Airtable — PlugIn) | https://airtable.com/invite/l?inviteId=inv2wjAWLwTe6odln&inviteToken=cd1abfc31726c8c3429df17f93d97bac71b05f9025852c14f6f331364ddbafc0 |

---

## 🧱 רכיבי המערכת

- **Airtable** — בסיס הנתונים המרכזי (base בשם `PlugIn`). מכיל את הטבלאות
  Invoices, Leads, Products, Tasks, Customers. פירוט מלא ב-[`docs/airtable-schema.md`](docs/airtable-schema.md).
- **n8n** — מנוע האוטומציה. עשרה workflows (ראו למטה) שמריצים את כל הלוגיקה
  העסקית ומחזיקים את כל ה-credentials במקום אחד.
- **OpenAI** — מודל השפה (`gpt-4o-mini`) לסוכני הצ'אט והמכירות, וכן יצירת
  embeddings לחיפוש הסמנטי.
- **Qdrant** — מסד וקטורי לחיפוש RAG. שתי אוספים (collections): `Products`
  ו-`Policies`.
- **Google Drive** — אחסון קובצי החשבוניות שמופקות אוטומטית.
- **Telegram** — שני בוטים: בוט שירות לקוחות ובוט ניהול לבעל/ת העסק.
- **Gmail** — שליחת מיילים קרים ללידים ומעקב אחרי תשובות.
- **אפליקציית הניהול (Lovable)** — ממשק ווב בעברית (RTL) שקורא וכותב לנתונים
  דרך נקודת כניסה אחת ב-n8n (`WF13`). ראו [`app/`](app/).

---

## 🔄 סקירת ה-Workflows

| # | Workflow | טריגר | תפקיד |
|---|---|---|---|
| 1 | Tax Document Validation | תזמון | ממספר ומאמת חשבוניות חדשות, מחשב מע"מ וסה"כ |
| 2 | Leads → Airtable | Webhook | קליטת ליד חדש, מניעת כפילויות, יצירת רשומה |
| 3 | Sales Cold Emails | תזמון (3 שעות) | שולח מייל קר מנוסח ב-AI ללידים חדשים |
| 4 | Sales Agent — Reply | תזמון (30 דק') | מזהה תשובות למיילים ומעדכן סטטוס ליד |
| 5 | Customer Service | Telegram | בוט שירות לקוחות מבוסס RAG + שמירת לידים |
| 6 | Policies RAG | טופס | הטמעת מסמכי מדיניות למאגר הווקטורי |
| 7 | Products RAG | טופס | הטמעת קטלוג המוצרים למאגר הווקטורי |
| 8 | InvoiceMaker | תזמון (דקה) | מפיק חשבונית HTML, מעלה ל-Drive, מסמן Issued |
| 9 | Manager Agent | Telegram | בוט אנליטיקה לבעל/ת העסק על נתוני החשבוניות |
| 13 | Webhook לאפליקציה | Webhook | נקודת הכניסה של אפליקציית הניהול (list/create/update/chat) |

תיאור מפורט של כל workflow — ב-[`docs/workflows.md`](docs/workflows.md).
תרשים ארכיטקטורה וזרימת מידע — ב-[`docs/architecture.md`](docs/architecture.md).

---

## 📁 מבנה הריפו

```
plugin-erp/
├── README.md                 ← המסמך הזה
├── workflows/                ← עשרת ה-workflows של n8n (JSON לייבוא)
│   ├── 01-tax-document-validation.json
│   ├── 02-leads-to-airtable.json
│   ├── 03-sales-cold-emails.json
│   ├── 04-sales-agent-reply.json
│   ├── 05-customer-service.json
│   ├── 06-policies-rag.json
│   ├── 07-products-rag.json
│   ├── 08-invoicemaker.json
│   ├── 09-manager-agent.json
│   └── 13-webhook-app.json
├── app/                      ← אפליקציית הניהול (קוקפיט בוקר)
│   ├── index.html
│   └── README.md
├── data/
│   └── products.csv          ← קטלוג המוצרים (מקור ל-RAG ול-Airtable)
├── docs/
│   ├── architecture.md       ← תרשים ארכיטקטורה וזרימת מידע
│   ├── airtable-schema.md    ← מבנה הטבלאות והשדות
│   └── workflows.md          ← תיאור מפורט לכל workflow
└── screenshots/              ← צילומי מסך (ראו ההנחיות בתיקייה)
```

---

## 🚀 התקנה והרצה

### ייבוא ה-Workflows ל-n8n
1. ב-n8n: **Workflows → Import from File**.
2. בחרו כל קובץ מתיקיית `workflows/`.
3. חברו את ה-credentials בכל workflow: Airtable (Personal Access Token),
   OpenAI, Qdrant, Google Drive, Telegram, Gmail.
4. הפעילו (Activate) כל workflow.

### הרצת אפליקציית הניהול
`app/index.html` היא אפליקציית עצמאית. בקובץ יש קבוע `WEBHOOK_URL` — יש להזין
בו את כתובת ה-Production של ה-Webhook מ-`WF13`. כל הקריאות והכתיבות עוברות דרכו.

### הגדרת ה-RAG
הריצו את `06-policies-rag` ו-`07-products-rag` פעם אחת דרך הטופס שלהם, כדי
לטעון את מסמכי המדיניות ואת קטלוג המוצרים ל-Qdrant. חשוב שמודל ה-embeddings
יהיה זהה בכל ה-workflows.

---

## 🔐 אבטחה

מפתחות ה-API (Airtable, OpenAI וכו') לעולם לא יושבים בקוד שרץ בדפדפן. כל
הגישה לנתונים עוברת דרך ה-Webhook של n8n (`WF13`), שמחזיק את ה-credentials
בצד השרת. אפליקציית הדפדפן לא מכירה את המפתחות כלל.
