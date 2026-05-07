# מסמך דרישות - תיעוד מערכת PaymentsAid

## מבוא

מערכת PaymentsAid היא אפליקציית Windows Forms בשפת C# המשמשת לניהול תשלומי סיוע לאוכלוסיות מיוחדות במשרד הרווחה (מוסא). המערכת מטפלת בתשלומים עבור חמש תוכניות סיוע: עיוורים, חירשים, מונשמים, כלבי נחייה לעיוורים וטלפון לחירשים. המערכת בנויה בארכיטקטורת 3-Tier קלאסית ומתממשקת עם מערכת מרכבה (ERP ממשלתי) ומערכת חשבות לביצוע תשלומים בפועל.

מסמך זה נועד לשמש כמדריך קליטה מקיף למפתח חדש המצטרף לצוות.

## מילון מונחים

- **PaymentsAid_UI**: שכבת ממשק המשתמש (Windows Forms) - פרויקט `PaymentsAid`, namespace: `Mosa.PaymentsAid.Win`
- **PaymentsAid_BL**: שכבת הלוגיקה העסקית - פרויקט `PaymentsAidBL`, namespace: `Mosa.PaymentsAid.BL`
- **PaymentsAid_BO**: שכבת האובייקטים העסקיים - פרויקט `PaymentsAidBO`, namespace: `Mosa.PaymentsAid.BO`
- **PaymentsAid_DAL**: שכבת גישה לנתונים - פרויקט `PaymentsAidDAL`, namespace: `Mosa.PaymentsAid.Dal`
- **מרכבה**: מערכת ה-ERP הממשלתית לניהול כספים, תקציבים ותשלומים
- **חשבות (Hashavut)**: מערכת החשבות הכללית לביצוע תשלומים בפועל
- **מט"ס (Mattas)**: מערכת מט"ס לניהול תשלומים לחירשים ועיוורים
- **מרגנית (Marganit)**: מערכת מספור אישי למטופלים
- **חסמות (Hasamot)**: שירות Web חיצוני לבדיקת זכאויות והסמכות רפואיות
- **תוכנית_סיוע**: אחת מחמש תוכניות הסיוע: עיוורים (5), חירשים (6), מונשמים (7), כלבי נחייה (8), טלפון (9)
- **חודש_עיבוד**: החודש שעבורו מתבצע חישוב ותשלום הסיוע, בפורמט MM/YYYY
- **בקשת_תשלום**: רשומת בקשה לתשלום סיוע עבור מטופל בחודש עיבוד מסוים
- **סעיף_תקציבי**: סעיף תקציבי במרכבה שממנו מתבצע התשלום
- **סוג_עזרה (AidType)**: סיווג סוג העזרה הניתנת למטופל (עובד/לא עובד, גיל פנסיה, תעריף מינימום)
- **עיבוד_שוטף**: תהליך חישוב תשלומים חודשי רגיל
- **תשלום_רטרואקטיבי**: תשלום עבור תקופה קודמת שלא שולמה בזמנה
- **ביטול_עיבוד**: ביטול עיבוד חודשי שכבר בוצע
- **BaseEntity**: מחלקת בסיס אבסטרקטית בשכבת BO המספקת מיפוי DataRow לאובייקטים
- **Active_Directory**: שירות ספריות של Windows לניהול הרשאות משתמשים
- **תובחה (Tovcha)**: תהליך העברת נתונים למרכבה
- **סטטוס_חיזון**: סטטוס חיצוני שמתקבל ממרכבה על מצב הבקשה

## דרישות

### דרישה 1: ארכיטקטורת המערכת - מבנה שכבות (3-Tier)

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את מבנה השכבות של המערכת, כדי שאוכל לנווט בקוד ולהבין את זרימת הנתונים.

#### קריטריוני קבלה

1. THE PaymentsAid_UI SHALL תלוי בשלוש שכבות: PaymentsAid_BL, PaymentsAid_BO ו-PaymentsAid_DAL כפרויקטים מקושרים (ProjectReference)
2. THE PaymentsAid_BL SHALL תלוי בשתי שכבות: PaymentsAid_BO ו-PaymentsAid_DAL כפרויקטים מקושרים
3. THE PaymentsAid_BO SHALL תלוי בשכבה אחת: PaymentsAid_DAL כפרויקט מקושר
4. THE PaymentsAid_DAL SHALL לא תלוי באף פרויקט פנימי אחר ומשמשת כשכבה הבסיסית ביותר
5. THE PaymentsAid_UI SHALL מוגדרת כ-WinExe (אפליקציית Windows) עם נקודת כניסה `Mosa.PaymentsAid.Win.Program`
6. THE PaymentsAid_BL SHALL מוגדרת כספריית מחלקות (Library) עם namespace `Mosa.PaymentsAid.BL`
7. THE PaymentsAid_BO SHALL מוגדרת כספריית מחלקות (Library) עם namespace `Mosa.PaymentsAid.BO`
8. THE PaymentsAid_DAL SHALL מוגדרת כספריית מחלקות (Library) עם namespace `Mosa.PaymentsAid.Dal`
9. THE PaymentsAid_UI SHALL מתייחסת לספריות חיצוניות: MosaGeneralDataBL, MosaGeneralDataBO, MosaGeneralDataDal (נתונים כלליים של מוסא), ActiveReports3 (דוחות), Microsoft.ReportViewer (הצגת דוחות), XPCommonControls (פקדי UI)

### דרישה 2: שכבת האובייקטים העסקיים (BO) - מודל הנתונים

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את מודל הנתונים של המערכת, כדי שאוכל להבין את הישויות העסקיות ואת הקשרים ביניהן.

#### קריטריוני קבלה

1. THE BaseEntity SHALL מספקת מחלקת בסיס אבסטרקטית שכל אובייקט עסקי יורש ממנה, עם מתודה אבסטרקטית `PopulateFromDataRow(DataRow dr)` למיפוי שורת נתונים לאובייקט
2. THE BaseEntity SHALL מספקת מתודה גנרית סטטית `PopulateList<T>(DataTable dt)` ליצירת רשימת אובייקטים מטבלת נתונים
3. THE BaseEntity SHALL מספקת מתודות עזר `GetNullableValueFromColum<T>` ו-`GetValueFromColum<T>` לטיפול בערכי null מבסיס הנתונים
4. WHEN אובייקט עסקי נוצר מ-DataRow, THE BaseEntity SHALL ממפה כל עמודה לשדה המתאים באובייקט באמצעות המתודה `PopulateFromDataRow`
5. THE PaymentsAid_BO SHALL מכילה את הישויות העסקיות הבאות: PatientDetails (פרטי מטופל), Requests (בקשות תשלום), SupportApplication (בקשות תמיכה), Blind (עיוורים), Deaf (חירשים), Respirator (מונשמים), BlindDog (כלבי נחייה), HashavutPayments (תשלומי חשבות), MattasPayments (תשלומי מט"ס), MonthsProcess (חודשי עיבוד), AidType (סוגי עזרה), Eligible (זכאויות), EntitlementDecision (החלטות זכאות), DoublePayments (תשלומים כפולים), TimingInterface (תזמון ממשקים), Users (משתמשים)
6. THE PatientDetails SHALL מכילה שדות: סוג זהות, מספר זהות, סמל מערכת, שם פרטי, שם משפחה, מין, תאריך לידה, כתובת, עיר, מיקוד, רשות מקומית, בנק, סניף, חשבון בנק, מספר מרגנית, סטטוס מרכבה, מספר מרכבה, פרטי מקבל פנסיה
7. THE Requests SHALL מכילה שדות לניהול בקשת תשלום כולל: סוג זהות, מספר זהות, סמל מערכת, חודש דיווח, חודש עיבוד, סטטוס בקשה, סטטוס תשלום, סכום תשלום, סוג עזרה, סיווג תקציבי, סעיף תקציבי, סטטוס מרכבה
8. THE SupportApplication SHALL מכילה שדות לניהול בקשת תמיכה במרכבה כולל: מספר בקשה, שנת בקשה, מספר רשומה, אסמכתא, תקנה, מרכז מימון, סכום הוצאה, סטטוס חיזון מרכבה, שובר מרכבה

### דרישה 3: שכבת הגישה לנתונים (DAL) - תבנית גישה לנתונים

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין כיצד המערכת ניגשת לבסיס הנתונים, כדי שאוכל לתחזק ולהרחיב את שאילתות הנתונים.

#### קריטריוני קבלה

1. THE PaymentsAid_DAL SHALL משתמשת בתבנית Typed DataSet של .NET עם קבצי XSD המגדירים את מבנה הנתונים ו-TableAdapters לגישה לבסיס הנתונים
2. WHEN שאילתת נתונים מתבצעת, THE PaymentsAid_DAL SHALL משתמשת ב-Stored Procedures (פרוצדורות מאוחסנות) בבסיס הנתונים, כפי שמשתקף בשמות כמו `usp_APP_PATIENT_DETAILS_Select`, `usp_APP_REQUESTS_Select`
3. THE PaymentsAid_DAL SHALL מתחברת לבסיס נתונים SQL Server בשם `SA_RSA_PAYMENTS_AID` באמצעות Windows Integrated Security
4. THE PaymentsAid_DAL SHALL מכילה DAL נפרד לכל ישות עסקית: PatientDetailsDal, RequestsDal, SupportApplicationDal, BlindDal, DeafDal, RespiratorDal, BlindDogDal, HashavutPaymentsDal, MattasPaymentsDal, MonthsProcessDal, ועוד
5. WHEN קובץ XSD מעודכן, THE PaymentsAid_DAL SHALL מייצרת אוטומטית קוד Designer.cs באמצעות MSDataSetGenerator
6. THE PaymentsAid_UI SHALL מתחברת גם לבסיס נתונים `MOSA_GENERAL_DATA` עבור נתונים כלליים של מוסא (רשויות, ערים וכדומה)

### דרישה 4: שכבת הלוגיקה העסקית (BL) - תבנית עבודה

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את תבנית הלוגיקה העסקית, כדי שאוכל להוסיף לוגיקה חדשה בצורה עקבית.

#### קריטריוני קבלה

1. THE PaymentsAid_BL SHALL מממשת כל מחלקת BL כיורשת של מחלקת BO המתאימה (לדוגמה: `PatientDetailsBL : PatientDetails`, `RequestsBL : Requests`, `SupportApplicationBL : SupportApplication`)
2. THE PaymentsAid_BL SHALL מספקת בכל מחלקת BL מתודות CRUD: Select (שליפה), Insert (הוספה), Update (עדכון), Delete (מחיקה) לפי הצורך
3. WHEN מתודת Select מופעלת, THE PaymentsAid_BL SHALL יוצרת TableAdapter מה-DAL, מפעילה את מתודת GetData המתאימה, וממירה את התוצאה לרשימת אובייקטים באמצעות `BaseEntity.PopulateList<T>`
4. WHEN מתודת InsertUpdate מופעלת, THE PaymentsAid_BL SHALL יוצרת TableAdapter מה-DAL ומעבירה את כל שדות האובייקט כפרמטרים ל-Stored Procedure המתאים
5. THE PaymentsAid_BL SHALL מכילה מחלקת `Common` עם הגדרות קבועות, enums ופונקציות עזר המשמשות את כל המערכת
6. THE Common SHALL מגדירה enums עבור: Gender (מין), PensionAge (גיל פנסיה), RequestStatus (סטטוס בקשה: שוטף=1, רטרואקטיבי=2, חריג=3, לא שולם=4, ניכוי=5, ביטול=6), PaymentType (סוג תשלום), PaymentStatus (סטטוס תשלום: ממתין=1, עבר=2, לא לעיבוד=3), UserGroup (קבוצת משתמש: צפייה=1, הרצה=2), PaymentTypeSymbol (סמל סוג תשלום), ProgramSymbol (סמל תוכנית: עיוורים=5, חירשים=6, מונשמים=7, כלבי נחייה=8, טלפון=9)
7. THE Common SHALL מספקת פונקציות עזר לעיצוב תאריכים ומחרוזות SQL: `MakeStringSQL`, `MakeStringYYYYMM`, `MakeStr4DateSQL`, `MakeStr4DateFrom`, `MakeStr4DateTo`
8. THE Common SHALL מספקת פונקציות להחלפת שפת מקלדת בין עברית לאנגלית באמצעות `LoadKeyboardLayout`

### דרישה 5: אימות משתמשים והרשאות

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את מנגנון ההרשאות, כדי שאוכל להבין מי יכול לבצע פעולות במערכת.

#### קריטריוני קבלה

1. WHEN המערכת מופעלת, THE PaymentsAid_UI SHALL בודקת את זהות המשתמש באמצעות `WindowsIdentity.GetCurrent()` ומחלצת את שם המשתמש מה-Active Directory
2. WHEN המשתמש שייך לקבוצת `AVODA_NT\SiyoaR`, THE PaymentsAid_UI SHALL מקצה הרשאת צפייה בלבד (View)
3. WHEN המשתמש שייך לקבוצת `AVODA_NT\SiyoaWrite`, THE PaymentsAid_UI SHALL מקצה הרשאת הרצה (Run) המאפשרת ביצוע פעולות
4. IF המשתמש אינו שייך לאף אחת מהקבוצות המורשות, THEN THE PaymentsAid_UI SHALL סוגרת את האפליקציה מיד ללא הצגת ממשק
5. WHILE המשתמש בעל הרשאת צפייה בלבד, THE PaymentsAid_UI SHALL מנטרלת את הקישור להכנת עיבוד (xpLinkedIbudPreparation) ומונעת ביצוע פעולות כתיבה
6. THE PaymentsAid_UI SHALL בודקת מופע קודם של האפליקציה באמצעות `CheckForPreviousInstance` ומונעת הפעלה כפולה

### דרישה 6: מסך ראשי וניווט

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את מבנה המסך הראשי, כדי שאוכל להבין את זרימת הניווט במערכת.

#### קריטריוני קבלה

1. THE MainForm SHALL מוגדרת כטופס MDI (Multiple Document Interface) המכילה SplitContainer עם פאנל ניווט שמאלי ואזור תוכן ימני
2. THE MainForm SHALL מציגה קישורי ניווט לחמישה מודולים: הכנת עיבוד (RequestForm), תשלום רטרואקטיבי (RetroPaymentForm), דוחות (Reports), שאילתות (QueryForm), היסטוריית תשלומים רטרואקטיביים (RetroPaymentHistory), ועדכון תעריפים (TaarifimForm)
3. WHEN המשתמש לוחץ על קישור ניווט, THE MainForm SHALL יוצרת מופע חדש של הטופס המתאים, מגדירה אותו כטופס-בן MDI, ומנטרלת את פאנל הניווט
4. WHEN טופס-בן נסגר, THE MainForm SHALL מפעילה מחדש את פאנל הניווט
5. THE MainForm SHALL מציגה מספר גרסה בשורת הסטטוס (גרסה נוכחית: 3.18.05.06 - Staging)
6. WHEN המשתמש לוחץ על קישור יציאה, THE MainForm SHALL סוגרת את כל טפסי הבנים ומסיימת את האפליקציה

### דרישה 7: תהליך עיבוד שוטף (הכנת עיבוד - RequestForm)

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את תהליך העיבוד השוטף, כדי שאוכל לתחזק את הלוגיקה המרכזית של המערכת.

#### קריטריוני קבלה

1. THE RequestForm SHALL מאפשרת בחירת תוכנית סיוע פעילה באמצעות checkboxes: עיוורים, חירשים, מונשמים, כלבי נחייה, טלפון
2. THE RequestForm SHALL מציגה את חודש העיבוד הנוכחי לכל תוכנית סיוע מטבלת `MonthsProcess`
3. THE RequestForm SHALL מבצעת תהליך עיבוד רב-שלבי הכולל: שלב 0 - קידום חודש עיבוד, שלב 1 - בדיקת מטופלים לעיבוד שוטף, שלב 2 - ביצוע עיבוד, שלב 3 - העברת בקשות תמיכה למרכבה, שלב 4 - בדיקת סטטוס בקשות תמיכה, שלב 5 - העברת דרישות תשלום למרכבה, שלב 6 - בדיקת סטטוס דרישות תשלום
4. WHEN שלב בדיקת מטופלים (שלב 1) מופעל עבור תוכנית עיוורים, THE RequestForm SHALL מפעילה את `CheckPationtForBlind()` הבודקת: קיום פרטי מטופל, קיום פרטי בנק, החלטת זכאות, גיל פנסיה, סטטוס תעסוקה, ובודקת זכאות מול שירות חסמות
5. WHEN שלב בדיקת מטופלים (שלב 1) מופעל עבור תוכנית חירשים, THE RequestForm SHALL מפעילה את `CheckPationtForDeaf()` הבודקת: קיום פרטי מטופל, קיום פרטי בנק, החלטת זכאות, גיל פנסיה, סטטוס תעסוקה, האם גם עיוור, ובודקת זכאות מול שירות חסמות
6. WHEN שלב בדיקת מטופלים (שלב 1) מופעל עבור תוכנית מונשמים, THE RequestForm SHALL מפעילה את `CheckPationtForRespirator()` הבודקת: קיום פרטי מטופל, קיום פרטי בנק, סטטוס טרכאוסטומיה, זכאות ביטוח לאומי, היתר עובד זר
7. WHEN שלב בדיקת מטופלים (שלב 1) מופעל עבור תוכנית כלבי נחייה, THE RequestForm SHALL מפעילה את `CheckPationtForDogs()` הבודקת: קיום פרטי מטופל, קיום פרטי בנק, תאריכי תוקף כלב נחייה
8. WHEN שלב חישוב (שלב 2) מופעל, THE RequestForm SHALL מחשבת סכום תשלום לכל מטופל לפי סוג העזרה, תעריפים, וסטטוס תעסוקה, ויוצרת רשומות בקשות תשלום (Requests)
9. WHEN שלב העברה למרכבה (שלב 3) מופעל, THE RequestForm SHALL יוצרת רשומות SupportApplication ומעבירה אותן למרכבה באמצעות ממשק התובחה
10. WHEN שלב בדיקת סטטוס (שלב 4) מופעל, THE RequestForm SHALL בודקת את סטטוס החיזון שהתקבל ממרכבה עבור כל בקשת תמיכה
11. THE RequestForm SHALL מציגה DataGridView עם רשימת בקשות התשלום, בקשות התמיכה, ובעיות שנמצאו בעיבוד
12. WHEN העיבוד מסתיים, THE RequestForm SHALL מייצרת קבצי פלט לחשבות (ExportFileHashavut) ולמט"ס (ExportFileMattas) בפורמט קבוע

### דרישה 8: חישוב סכומי תשלום

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את לוגיקת חישוב התשלומים, כדי שאוכל לתחזק ולעדכן את החישובים.

#### קריטריוני קבלה

1. WHEN חישוב מתבצע עבור עיוור, THE RequestForm SHALL קובעת סוג עזרה (PaymentTypeSymbol) לפי: עיוור עובד (51), עיוור לא עובד (52), תעריף מינימום (53), עיוור בגיל פנסיה (54)
2. WHEN חישוב מתבצע עבור חירש, THE RequestForm SHALL קובעת סוג עזרה לפי: חירש עובד (61), חירש לא עובד (62), חירש בגיל פנסיה עובד (63), חירש בגיל פנסיה לא עובד (64)
3. WHEN חישוב מתבצע עבור מונשם, THE RequestForm SHALL משתמשת בסוג עזרה מונשם (71)
4. WHEN חישוב מתבצע עבור כלב נחייה, THE RequestForm SHALL משתמשת בסוג עזרה כלבי נחייה (81)
5. THE RequestForm SHALL מחשבת גיל פנסיה לפי מין: גבר - 67, אישה - 62 (PensionAge), עם תמיכה בגיל פנסיה ישן: גבר - 65, אישה - 60 (OldPensionAge)
6. THE RequestForm SHALL מחשבת גיל פנסיה לעיוורים בנפרד: גבר - 67, אישה - 62 (PensionAgeBlind)
7. WHEN סכום התשלום חורג מהתקציב, THE RequestForm SHALL מסמנת את הבקשה כחריגה (IfExceeding) עם סטטוס: חריגה ממתינה לחריג (0), חריגה ממתינה לאנלה (1), לא חורגת (9)
8. THE RequestForm SHALL בודקת תשלומים כפולים באמצעות `checkDoublePayment()` ושולחת התראה במייל כאשר מזוהים תשלומים כפולים

### דרישה 9: תשלום רטרואקטיבי (RetroPaymentForm)

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את תהליך התשלום הרטרואקטיבי, כדי שאוכל לטפל בבקשות תשלום עבור תקופות קודמות.

#### קריטריוני קבלה

1. THE RetroPaymentForm SHALL מאפשרת חיפוש מטופל לפי סוג זהות ומספר זהות
2. THE RetroPaymentForm SHALL מציגה את פרטי המטופל האישיים, פרטי בנק, ופרטים ספציפיים לתוכנית הסיוע
3. WHEN מטופל נמצא, THE RetroPaymentForm SHALL טוענת את היסטוריית הבקשות שלו ומציגה אותן ב-DataGridView
4. THE RetroPaymentForm SHALL מאפשרת בחירת תוכנית סיוע (עיוורים, חירשים, מונשמים, כלבי נחייה) ומציגה שדות רלוונטיים לתוכנית הנבחרת
5. WHEN תשלום רטרואקטיבי מתבצע עבור עיוור, THE RetroPaymentForm SHALL מפעילה את `DoCalculationForBlind()` הבודקת זכאות, מחשבת סכום, ויוצרת בקשת תשלום עם סטטוס רטרואקטיבי (RequestStatus=2)
6. WHEN תשלום רטרואקטיבי מתבצע עבור חירש, THE RetroPaymentForm SHALL מפעילה את `DoCalculationForDeaf()` הבודקת זכאות, מחשבת סכום, ויוצרת בקשת תשלום
7. WHEN תשלום רטרואקטיבי מתבצע עבור מונשם, THE RetroPaymentForm SHALL בודקת תנאים מוקדמים באמצעות `CheckBeforeRespiratorOrder()` לפני ביצוע החישוב
8. THE RetroPaymentForm SHALL בודקת זכאות מול שירות חסמות (Hasamot) באמצעות `CheckEntitlementFromHasamot()` לפני אישור תשלום
9. THE RetroPaymentForm SHALL מאפשרת מחיקת בקשת תשלום רטרואקטיבי קיימת
10. THE RetroPaymentForm SHALL בודקת שהעיבוד השוטף הושלם לפני שמאפשרת תשלום רטרואקטיבי באמצעות `checkIfIbudIsDone()`

### דרישה 10: ביטול עיבוד (CancelIbud)

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את תהליך ביטול העיבוד, כדי שאוכל לטפל במקרים שבהם נדרש לבטל עיבוד שבוצע.

#### קריטריוני קבלה

1. THE CancelIbud SHALL מאפשרת שני סוגי ביטול: ביטול מלא (MakeFullCancelation) של כל העיבוד לחודש מסוים, וביטול למטופל ספציפי (MakePationCancelation)
2. THE CancelIbud SHALL מאפשרת בחירת תוכנית סיוע (עיוורים, חירשים, כלבי נחייה) לביטול
3. WHEN ביטול מלא מתבצע, THE CancelIbud SHALL מפעילה את `RequestsBL.CancelIbud()` המבטלת את כל בקשות התשלום לחודש העיבוד הנבחר
4. WHEN ביטול למטופל ספציפי מתבצע, THE CancelIbud SHALL מאפשרת חיפוש מטופל לפי תעודת זהות ומפעילה את `RequestsBL.CancelIbudForPation()`
5. WHEN ביטול מתבצע, THE CancelIbud SHALL בודקת תחילה שלא בוצע ביטול קודם באמצעות `RequestsBL.checkCancelation()`

### דרישה 11: ממשק מרכבה (Merkava Interface)

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את הממשק עם מערכת מרכבה, כדי שאוכל לטפל בבעיות ממשק ולהבין את זרימת הנתונים.

#### קריטריוני קבלה

1. THE PaymentsAid_UI SHALL מתממשקת עם מרכבה באמצעות תהליך תובחה (Tovcha) להעברת בקשות תמיכה ודרישות תשלום
2. THE TimingInterface SHALL מנהלת את תזמון הממשקים עם מרכבה כולל: קוד מערכת שולחת (SendSystemCode=7), קוד מערכת מקבלת (ReceiveSystemCode=6), קוד ממשק תמיכה (InterfaceCodeSupport=12), קוד ממשק דרישה (InterfaceCodeRequest=13)
3. THE SupportApplication SHALL מייצגת בקשת תמיכה במרכבה עם שדות: מספר בקשה, שנת בקשה, תקנה, מרכז מימון, סכום הוצאה, סטטוס חיזון, שובר מרכבה
4. THE HashavutPayments SHALL מייצגת דרישת תשלום לחשבות עם שדות: סעיף תקציבי, סכום נטו, סכום שוטף, סכום רטרואקטיבי, תקנה, מרכז מימון, מספר דרישה, אסמכתא, סטטוס חיזון מרכבה
5. WHEN סטטוס חיזון מתקבל ממרכבה, THE PaymentsAid_BL SHALL מעדכנת את סטטוס בקשת התמיכה ודרישת התשלום בהתאם
6. THE PaymentsAid_UI SHALL מציגה את סטטוס החיזון של מרכבה ב-DataGridView ייעודי (dgRequestStatusHizon9)

### דרישה 12: ממשק חסמות (Hasamot Web Service)

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את הממשק עם שירות חסמות, כדי שאוכל להבין כיצד נבדקות זכאויות.

#### קריטריוני קבלה

1. THE PaymentsAid_UI SHALL מתחברת לשירות Web של חסמות בכתובת `http://adabas/MosaIntegrationWS/Hasamot.asmx`
2. WHEN בדיקת זכאות מתבצעת, THE RequestForm SHALL מפעילה את `CheckEntitlementFromHasamot()` השולחת בקשה לשירות חסמות עם פרטי המטופל וחודש הדיווח
3. THE PaymentsAid_UI SHALL מקבלת מחסמות אובייקט `HasamaWithMisgeretInfo` המכיל מידע על הסמכה רפואית ומסגרת
4. WHEN חסמות מחזירה שהמטופל אינו זכאי, THE RequestForm SHALL מסמנת את המטופל כבעייתי ומוסיפה אותו לרשימת הבעיות

### דרישה 13: ייצוא קבצי תשלום

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את תהליך ייצוא קבצי התשלום, כדי שאוכל לתחזק את הממשקים עם מערכות חיצוניות.

#### קריטריוני קבלה

1. WHEN עיבוד מסתיים עבור עיוורים או כלבי נחייה, THE RequestForm SHALL מייצרת קובץ חשבות (ExportFileHashavut) וקובץ מט"ס לעיוורים (ExportFileMattasBlind) בפורמט קבוע
2. WHEN עיבוד מסתיים עבור חירשים, THE RequestForm SHALL מייצרת קובץ מט"ס לחירשים (ExportFileMattasDeaf) בפורמט קבוע
3. WHEN עיבוד מסתיים עבור מונשמים, THE RequestForm SHALL מייצרת קובץ חשבות למונשמים (ExportFileHashavutRespiration) בפורמט קבוע
4. THE RequestForm SHALL שומרת את קבצי הפלט בתיקיית `Output` שנוצרת אוטומטית בהפעלת המערכת
5. THE PaymentsAid_UI SHALL מאפשרת הגדרה בקובץ app.config האם לייצר קובץ חשבות (`MakeHashavutFile=True/False`)

### דרישה 14: דוחות (Reports)

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את מערכת הדוחות, כדי שאוכל להוסיף ולתחזק דוחות.

#### קריטריוני קבלה

1. THE Reports SHALL מאפשרת בחירת תוכנית סיוע (עיוורים, חירשים, מונשמים, כלבי נחייה, טלפון) ובחירת סוג דוח
2. THE Reports SHALL מאפשרת סינון דוחות לפי טווח תאריכים, רשות מקומית, וסטטוס
3. THE PaymentsAid_UI SHALL משתמשת ב-Microsoft ReportViewer להצגת דוחות מ-SQL Server Reporting Services (SSRS) בכתובת המוגדרת ב-app.config (`ReportServer`, `ReportPath`)
4. THE Reports SHALL מציגה דוחות שונים לכל תוכנית סיוע כולל: דוח סטטיסטי, דוח תשלומים, דוח בעיות
5. THE GeneralFunc SHALL מספקת פונקציה `RenderProblemPatientReport()` ליצירת דוח מטופלים בעייתיים ושליחתו במייל

### דרישה 15: מערכת דואר אלקטרוני

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את מנגנון שליחת המיילים, כדי שאוכל לתחזק את ההתראות האוטומטיות.

#### קריטריוני קבלה

1. THE MailHelper SHALL מאפשרת שליחת מיילים עם קבצים מצורפים באמצעות SMTP
2. THE MailHelper SHALL תומכת ברינדור דוחות SSRS לקבצי PDF וצירופם למייל
3. THE MailHelper SHALL שומרת רשומת מייל בבסיס הנתונים באמצעות `MailsBL.InsertMails()` כולל: כתובת שולח, כתובת נמען, נושא, גוף הודעה, תאריך קבלה, סטטוס שליחה
4. THE MailHelper SHALL שומרת קבצים מצורפים בבסיס הנתונים באמצעות `MailsAttachmentsBL.InsertUpdateMailsAttachments()`
5. WHEN תשלומים כפולים מזוהים, THE GeneralFunc SHALL שולחת מייל התראה עם טבלת HTML המפרטת את המטופלים עם תשלומים כפולים, לכתובות המוגדרות ב-app.config (`DoublePaymentsProgram5And8MailAdress`, `DoublePaymentsProgram6MailAdress`)
6. THE MailHelper SHALL משתמשת בכתובת שולח `MerkavaTovchaUser@molsa.gov.il` לשליחת מיילים אוטומטיים

### דרישה 16: ניהול תעריפים (TaarifimForm)

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את ניהול התעריפים, כדי שאוכל לעדכן תעריפי תשלום.

#### קריטריוני קבלה

1. THE TaarifimForm SHALL מציגה טבלת תעריפים ב-DataGridView עם אפשרות סינון לפי תוכנית סיוע
2. THE TaarifimForm SHALL מאפשרת עדכון תעריף חדש עבור סוג עזרה נבחר
3. THE TaarifimForm SHALL מאפשרת הצגת פרטי תעריף כולל: סוג עזרה, תיאור, סכום נוכחי, סכום חדש
4. THE TaarifimForm SHALL מאפשרת הדפסת טבלת התעריפים
5. THE TaarifimForm SHALL מאמתת שהתעריף החדש הוא מספרי בלבד באמצעות `OnlyNum()` KeyPress handler

### דרישה 17: שאילתות וחיפוש (QueryForm)

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את מודול השאילתות, כדי שאוכל לחפש מידע על מטופלים ובקשות.

#### קריטריוני קבלה

1. THE QueryForm SHALL מאפשרת חיפוש מטופלים ובקשות תשלום לפי פרמטרים שונים
2. THE QueryForm SHALL מציגה תוצאות חיפוש ב-DataGridView
3. THE QueryForm SHALL מאפשרת סינון לפי תוכנית סיוע, תקופה, סטטוס בקשה, וסטטוס תשלום

### דרישה 18: תוכניות סיוע - מודל נתונים ספציפי

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את המודל הספציפי לכל תוכנית סיוע, כדי שאוכל לטפל בלוגיקה הייחודית לכל תוכנית.

#### קריטריוני קבלה

1. THE Blind SHALL מכילה שדות ספציפיים לעיוורים: סטטוס תעסוקה, החלטת זכאות, תאריך אישור זכאות, תאריך תחילת זכאות, תאריכי עבודה, סטטוס שר"ם, האם גם חירש
2. THE Deaf SHALL מכילה שדות ספציפיים לחירשים: סטטוס תעסוקה, החלטת זכאות, תאריך אישור זכאות, תאריך תחילת זכאות, תאריכי עבודה, האם גם עיוור
3. THE Respirator SHALL מכילה שדות ספציפיים למונשמים: האם טרכאוסטומיה, זכאות ביטוח לאומי (BTL), היתר עובד זר, תאריך פטירה, בעלות חשבון בנק, פרטי מקבל תשלום (אפוטרופוס), פקס, טלפון, מייל
4. THE BlindDog SHALL מכילה שדות ספציפיים לכלבי נחייה כולל פרטי מטופל, פרטי בנק, ותאריכי תוקף
5. THE MattasPayments SHALL מכילה שדות לתשלומי מט"ס: סכום חודשי, מהות נזקקות (QualityNeed), סוג עזרה, סיווג תקציבי
6. THE HashavutPayments SHALL מכילה שדות לתשלומי חשבות: סעיף תקציבי, סכום נטו, סכום שוטף, סכום רטרואקטיבי, תקנה, מרכז מימון, מספר דרישה, אסמכתא, סטטוס חיזון מרכבה

### דרישה 19: טיפול בשגיאות ובעיות

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין כיצד המערכת מטפלת בשגיאות ובעיות, כדי שאוכל לאבחן ולתקן בעיות.

#### קריטריוני קבלה

1. WHEN בעיה מזוהה במטופל במהלך עיבוד, THE RequestForm SHALL מוסיפה את המטופל לרשימת בעיות (ProblemReason) ומציגה אותו ב-DataGridView ייעודי
2. WHEN עיבוד מסתיים עם בעיות, THE GeneralFunc SHALL מייצרת דוח מטופלים בעייתיים ושולחת אותו במייל לכתובת המוגדרת ב-app.config
3. THE PaymentsAid_BO SHALL מספקת שדה `ErrorString` ב-BaseEntity לתיעוד שגיאות ברמת האובייקט
4. THE SupportApplication SHALL מספקת שדות `ErrorFiledDesc` ו-`ErrorDesc` לתיעוד שגיאות מרכבה
5. IF שגיאת חיזון מתקבלת ממרכבה, THEN THE PaymentsAid_BL SHALL מעדכנת את שדה `MerkavaErrorDesc` בדרישת התשלום ומציגה את השגיאה למשתמש

### דרישה 20: הגדרות תצורה (Configuration)

**סיפור משתמש:** כמפתח חדש, אני רוצה להבין את הגדרות התצורה של המערכת, כדי שאוכל להגדיר סביבות עבודה שונות.

#### קריטריוני קבלה

1. THE PaymentsAid_UI SHALL קוראת הגדרות חיבור לבסיס נתונים מקובץ app.config כולל: connection string ל-SA_RSA_PAYMENTS_AID ול-MOSA_GENERAL_DATA
2. THE PaymentsAid_UI SHALL קוראת הגדרות שרת דוחות מקובץ app.config: `ReportServer` (כתובת שרת SSRS) ו-`ReportPath` (נתיב דוחות)
3. THE PaymentsAid_UI SHALL קוראת כתובת שירות חסמות מהגדרות האפליקציה: `PaymentsAid_Hasamot_Hasamot`
4. THE PaymentsAid_UI SHALL קוראת כתובות מייל להתראות מקובץ app.config: `MailAdress`, `DoublePaymentsProgram5And8MailAdress`, `DoublePaymentsProgram6MailAdress`
5. THE PaymentsAid_UI SHALL קוראת הגדרת `MakeHashavutFile` מקובץ app.config לקביעה האם לייצר קובץ חשבות
6. THE PaymentsAid_UI SHALL בנויה על .NET Framework 2.0 עם תמיכה ב-Windows Integrated Security לאימות
