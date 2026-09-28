# אפליקציית הניהול — קוקפיט בוקר

ממשק ווב פנימי בעברית (RTL) לניהול העסק: דשבורד, טבלאות (לקוחות, לידים,
הזמנות, חשבוניות, מוצרים, משימות), טפסי יצירה, ופאנל צ'אט עם עוזר.

הגרסה המקורית נבנתה ב-Lovable: https://pixel-perfect-snap-684.lovable.app
הקובץ `index.html` כאן הוא גרסת קוקפיט עצמאית (single-file) לצורך תיעוד וגיבוי.

## חיבור לנתונים
כל הקריאות והכתיבות עוברות דרך נקודת כניסה אחת ב-n8n (`WF13`), בגוף אחיד:
```
{ action, table, payload }
```
- קריאה: `{ action: "list", table, payload: {} }`
- יצירה: `{ action: "create", table, payload: { fields } }`
- עדכון: `{ action: "update", table, payload: { id, fields } }`
- צ'אט: `{ action: "chat", message }`

בקובץ `index.html` יש קבוע `WEBHOOK_URL` — יש להזין בו את כתובת ה-Production
של ה-Webhook מ-`WF13`. אין מפתחות Airtable באפליקציה — הכול עובר דרך n8n.
