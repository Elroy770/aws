# 04 — IAM Advanced & Organizations

## 1. מה זה?

זהו שילוב של IAM policy evaluation, STS, IAM Identity Center, AWS Organizations ו-Service Control Policies ‏(SCPs).

## 2. למה צריך את זה?

ארגון צריך גישה מרכזית ורב-חשבונית, credentials זמניים ו-guardrails שלא ניתן לעקוף בעזרת admin מקומי.

## 3. איך זה עובד?

STS מנפיק credentials זמניים כאשר role נלקח. Identity Center מחבר workforce לזהות מרכזית ולחשבונות AWS. Organizations מארגן accounts ב-OUs; SCP קובע את התקרה המותרת לחשבון/OUs. Explicit deny גובר על allow; SCP אינו מעניק הרשאות בפני עצמו.

## 4. הדברים שחייבים לדעת למבחן

- Policy evaluation: explicit deny > allow > implicit deny.
- Cross-account access: trust policy ב-role + permissions אצל ה-caller.
- SCP מגביל את maximum permissions, גם ל-admin בחשבון חבר.
- IAM Identity Center מתאים ל-SSO של עובדים.

## 5. ההבדלים החשובים

| מנגנון | תפקיד |
|---|---|
| IAM policy | מעניקה/דוחה הרשאה לזהות |
| SCP | גבול הרשאות לארגון/חשבון |
| Permission boundary | גבול להרשאות של principal מסוים |
| Trust policy | מי רשאי לקחת role |

## 6. מלכודות במבחן

SCP עם Allow לבדו לא מספיק. גם לאחר הסרת deny, עדיין חייבת להיות identity/resource policy שמאפשרת את הפעולה.

## 7. Scenario מהעולם האמיתי

הארגון אוסר יצירת משאבים מחוץ לאזור מאושר. הצמד SCP ל-OU שמדחה פעולות מחוץ ל-Region, ואז נהל SSO לעובדים דרך Identity Center.

## 8. מה לא צריך לדעת

לא נדרש לשנן את מסכי הקונסולה או כל condition key של Organizations.

## 9. סיכום

- STS = credentials זמניים.
- Trust ו-permission הם שני צדדים של role.
- Explicit deny תמיד מנצח.
- SCP מגביל, לא מעניק.
- SSO מרכזי: Identity Center.

## 10. בדיקת הבנה

1. האם SCP מעניק הרשאה?
2. מה נדרש ל-cross-account role?
3. מי גובר: explicit deny או allow?

## 10. הרחבה: תכנון לפי ששת ה-Pillars

Operational Excellence: Identity Center, permission sets ו-CloudTrail מקטינים עבודה ידנית. Security: SCP ו-boundaries יוצרים guardrails, אך אינם תחליף ל-IAM policy. Reliability: הרשאות cross-account ל-backup ול-DR חייבות להיבדק מראש. Performance Efficiency: federation ו-STS מפחיתים proxy ושירותים מיותרים. Cost Optimization: tagging, budgets ו-account separation מראים מי צורך; centralized audit/logging מוסיף חיובי ingestion ו-retention. Sustainability: כיבוי sandbox accounts ומשאבים לא בשימוש חוסך compute ואנרגיה. Multi-account בדרך כלל יקר יותר תפעולית מחשבון יחיד, אך זול יותר כאשר isolation, compliance ו-blast-radius חשובים.
