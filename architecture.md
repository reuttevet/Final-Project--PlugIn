# ארכיטקטורה וזרימת מידע

## מבט על

המערכת בנויה סביב **Airtable** כמקור אמת יחיד, **n8n** כמנוע האוטומציה, ושכבת
**AI/RAG** (OpenAI + Qdrant) לצ'אטים ולמכירות. שלוש נקודות כניסה מפעילות את
המערכת: אפליקציית הניהול (Webhook), בוטים בטלגרם, וטריגרים מתוזמנים.

```mermaid
flowchart TB
    subgraph Clients["נקודות כניסה"]
        APP["אפליקציית ניהול\n(Lovable)"]
        TG1["בוט ניהול\n(Telegram)"]
        TG2["בוט שירות\n(Telegram)"]
        WEB["טפסי לידים / אתר"]
    end

    subgraph n8n["n8n — מנוע אוטומציה"]
        WF13["WF13 — Webhook לאפליקציה\nlist / create / update / chat"]
        WF9["WF9 — Manager Agent"]
        WF5["WF5 — Customer Service"]
        WF2["WF2 — Leads Intake"]
        WF3["WF3 — Cold Emails"]
        WF4["WF4 — Reply Tracking"]
        WF1["WF1 — Tax Validation"]
        WF8["WF8 — InvoiceMaker"]
        WF6["WF6 — Policies RAG"]
        WF7["WF7 — Products RAG"]
    end

    subgraph Data["נתונים ושירותים"]
        AT[("Airtable\nPlugIn base")]
        QD[("Qdrant\nProducts / Policies")]
        GD[("Google Drive\nחשבוניות")]
        OA["OpenAI\nGPT + embeddings"]
        GM["Gmail"]
    end

    APP --> WF13 --> AT
    WF13 --> QD
    TG1 --> WF9 --> AT
    TG2 --> WF5 --> QD
    WF5 --> WF2
    WEB --> WF2 --> AT
    WF2 --> GM
    WF3 --> AT
    WF3 --> GM
    WF4 --> GM
    WF4 --> AT
    WF1 --> AT
    WF8 --> AT
    WF8 --> GD
    WF6 --> QD
    WF7 --> QD
    WF5 --> OA
    WF9 --> OA
    WF13 --> OA
```

## זרימות מרכזיות

### מסע הליד
1. ליד נכנס דרך טופס/בוט → **WF2** בודק כפילות ב-Airtable ויוצר רשומה בסטטוס `New`.
2. **WF3** רץ כל 3 שעות, מוצא לידים בסטטוס `New`, מנסח מייל קר ב-AI, שולח, ומסמן `Contacted`.
3. **WF4** רץ כל 30 דקות, בודק תשובות ב-Gmail, ומסמן לידים שהגיבו כ-`Replied`.

### מסע החשבונית
1. **WF1** אוסף חשבוניות חדשות, מאמת (סכום > 0, יש לקוח), מחשב מע"מ וסה"כ,
   ממספר (`INV-XXXX`) ומסמן `Ready` (או `Invalid`).
2. **WF8** רץ כל דקה, מוצא חשבוניות `Ready` בלי PDF, בונה HTML, מעלה ל-Google Drive,
   כותב את הקישור ל-`PdfUrl` ומסמן `Issued`.

### שירות לקוחות (RAG)
- **WF6/WF7** טוענים מראש מדיניות ומוצרים ל-Qdrant.
- **WF5** (בוט טלגרם) עונה ללקוחות על סמך המידע שנשלף מ-Qdrant, ושומר לידים דרך WF2.

### ניהול
- **WF9** (בוט טלגרם, מוגבל ל-Chat ID של הבעלים) עונה על שאלות אנליטיקה
  (הכנסות, חשבוניות פתוחות) על סמך שליפה חיה מ-Airtable.
- **WF13** משרת את אפליקציית הווב: קריאה, יצירה ועדכון של רשומות, וצ'אט עם סוכן RAG.
