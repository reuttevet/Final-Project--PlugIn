# סכימת Airtable — base "PlugIn"

Base ID: `appqXVC2uospFtYbh`

הבסיס מכיל חמש טבלאות. שמות השדות משמשים ישירות את ה-workflows ואת אפליקציית
הניהול, ולכן חשוב לשמור עליהם מדויקים.

## Invoices — חשבוניות
| שדה | סוג | הערות |
|---|---|---|
| InvoiceNumber | Single line text | שדה ראשי; פורמט `INV-0001` |
| CustomerId | Single line text | מזהה/שם הלקוח |
| Amount | Currency (₪) | סכום לפני מע"מ |
| VatAmount | Currency (₪) | מע"מ (18%) |
| Total | Currency (₪) | סה"כ כולל מע"מ |
| Status | Single select | Draft / Ready / Issued / Paid / Invalid |
| PdfUrl | URL | קישור לחשבונית ב-Google Drive |
| Created | תאריך | מועד יצירה |
| Notes | Long text | הערות חופשיות |

## Leads — לידים
| שדה | סוג | הערות |
|---|---|---|
| Name | Single line text | שדה ראשי |
| Email | Email | |
| Company | Single line text | |
| Phone | Phone | |
| Status | Single select | New / Contacted / Replied / Won / Lost |
| Interest | Long text | תחום/מוצר שהלקוח התעניין בו |
| Created | Created time | |

## Products — מוצרים ושירותים
| שדה | סוג | הערות |
|---|---|---|
| Name | Single line text | שדה ראשי |
| Category | Single line text | אוזניות, מסכים, שירותים וכו' |
| Price | Currency (₪) | |
| Description | Long text | מפרט מלא |
| InStock | Checkbox | במלאי כן/לא |

## Tasks — משימות
| שדה | סוג | הערות |
|---|---|---|
| Title | Single line text | שדה ראשי |
| Status | Single select | Todo / Doing / Done |

## Customers — לקוחות
| שדה | סוג | הערות |
|---|---|---|
| Name | Single line text | שדה ראשי |
| Phone | Phone | |
| Email | Email | |
| City | Long text | |
| CustomerId | Number | מזהה לקוח |
| Created | Created time | |
