# תיאור מפורט של ה-Workflows

כל הקבצים בתיקיית [`../workflows/`](../workflows/), מוכנים לייבוא ל-n8n
(**Import from File**). כולם עובדים מול אותו base ב-Airtable (`PlugIn`).

---

## 1 — Tax Document Validation
**קובץ:** `01-tax-document-validation.json` · **טריגר:** Schedule (כל 10 שניות)

ממספר ומאמת חשבוניות חדשות.
- שולף מ-Invoices רשומות ללא `InvoiceNumber`.
- לולאה על הרשומות; לכל חשבונית ה-`If` בודק שהיא תקינה (`Amount > 0` ויש `CustomerId`).
- תקינה → סופר כמה חשבוניות כבר ממוספרות, מקצה מספר רץ (`INV-XXXX`),
  מחשב מע"מ (18%) וסה"כ, ומעדכן את הרשומה לסטטוס `Ready`.
- לא תקינה → מסמן `Invalid`.

## 2 — Leads → Airtable
**קובץ:** `02-leads-to-airtable.json` · **טריגר:** Webhook (POST)

נקודת קליטת לידים.
- מקבל ליד (שם / אימייל / טלפון / תחום עניין).
- מחפש ב-Leads אם כבר קיים ליד עם אותו אימייל (מניעת כפילויות).
- אם חדש → יוצר רשומה בסטטוס `New` ושולח מייל התראה דרך Gmail.

## 3 — Sales Cold Emails
**קובץ:** `03-sales-cold-emails.json` · **טריגר:** Schedule (כל 3 שעות)

סוכן מכירות למיילים קרים.
- שולף עד 10 לידים בסטטוס `New`.
- מנסח לכל אחד מייל קר קצר בעברית (chain LLM) לפי השם, החברה ותחום העניין.
- שולח דרך Gmail ומעדכן את הליד ל-`Contacted`.

## 4 — Sales Agent — Reply
**קובץ:** `04-sales-agent-reply.json` · **טריגר:** Schedule (כל 30 דקות)

מעקב אחרי תשובות.
- שולף הודעות מ-Gmail, מוצא לידים בסטטוס `Contacted`,
  ומסמן `Replied` את מי שהגיב.

## 5 — Customer Service
**קובץ:** `05-customer-service.json` · **טריגר:** Telegram

בוט שירות לקוחות מבוסס RAG.
- מקבל הודעת לקוח בטלגרם.
- סוכן AI (gpt-4o-mini) עונה בעברית באמצעות שני כלי חיפוש ב-Qdrant:
  `Products` (קטלוג) ו-`Policies` (מדיניות).
- כשלקוח מתעניין — אוסף שם + פרט קשר ושומר ליד (קורא ל-WF2).
- זיכרון שיחה לפי `chat.id`.

## 6 — Policies RAG
**קובץ:** `06-policies-rag.json` · **טריגר:** Form

הטמעת מסמכי מדיניות.
- טופס להעלאת קובץ מדיניות/נהלים.
- יוצר embeddings (OpenAI) ומכניס לאוסף `Policies` ב-Qdrant (mode = insert).

## 7 — Products RAG
**קובץ:** `07-products-rag.json` · **טריגר:** Form

הטמעת קטלוג המוצרים.
- טופס להעלאת קובץ CSV של מוצרים.
- מפצל לטקסט (chunks של 800 עם חפיפה 100), יוצר embeddings ומכניס לאוסף
  `Products` ב-Qdrant.

## 8 — InvoiceMaker
**קובץ:** `08-invoicemaker.json` · **טריגר:** Schedule (כל דקה)

הפקת חשבוניות ל-Drive.
- שולף חשבוניות בסטטוס `Ready` שאין להן `PdfUrl`.
- בונה חשבונית HTML בעברית (RTL) עם מספר, לקוח, סכום, מע"מ, סה"כ ותאריך.
- ממיר לקובץ, מעלה ל-Google Drive (תיקיית `PlugIn- Invoices`),
  כותב את קישור הצפייה ל-`PdfUrl` ומסמן `Issued`.

## 9 — Manager Agent
**קובץ:** `09-manager-agent.json` · **טריגר:** Telegram

בוט אנליטיקה לבעל/ת העסק.
- `If` מוודא שההודעה הגיעה מ-Chat ID של הבעלים בלבד.
- סוכן AI עונה בעברית על שאלות אנליטיקה (הכנסות, מספר חשבוניות, סכומים
  שלא שולמו) על סמך שליפה חיה מ-Airtable (כלי `Search records`).

## 13 — Webhook לאפליקציה
**קובץ:** `13-webhook-app.json` · **טריגר:** Webhook (POST `/erp-app`)

נקודת הכניסה היחידה של אפליקציית הניהול. גוף אחיד: `{ action, table, payload }`.

| action | גוף הבקשה | מה חוזר |
|---|---|---|
| `list` | `{ action, table, limit }` | `{ records: [{ id, fields }] }` |
| `create` | `{ action, table, payload }` | `{ ok, record }` |
| `update` | `{ action, table, payload }` (עם `id`) | `{ ok, record }` |
| `chat` | `{ action, message }` | `{ ok, reply }` |

- `Switch` מנתב לפי `action`.
- `chat` → סוכן RAG (Qdrant Products/Policies) שמחזיר תשובה בעברית.
- `list/create/update` → פעולות Airtable מול הטבלה שנשלחה בשדה `table`.
- כל ה-credentials נשמרים כאן בצד השרת; האפליקציה לא מכירה אותם.
