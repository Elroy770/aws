# AWS SAA-C03 Study Workspace — Instructions for AI Agents

Read this file before working in this folder. These instructions apply to every AI agent using this workspace.

## Workspace layout

```text
AWS/
├── AWS_Certified_Solutions_Architect_Slides.md   # מאגר המקור: שקפי הקורס המלאים (~16.5K שורות, 878 שקפים)
├── lessons/                  # 41 שיעורים ערוכים ומורחבים — חומר הלימוד הראשי
├── questions/                # מאגרי שאלות תרגול ובחינות + סקירות תשובות
└── prompts/
    ├── README.md             # הקובץ הזה: workflow לסוכנים
    ├── AWS teacher.md        # הוראות ליצירת שיעור
    ├── AWS questions.md      # הוראות ליצירת שאלות/מבחנים
    └── subjects.md           # תוכנית הלימודים המסודרת (41 שיעורים)
```

The workspace is an Obsidian vault. Use Markdown files and preserve the existing Hebrew/English style.
Do not put generated lessons or exams inside `prompts/`.

---

## מאגר המקור — `AWS_Certified_Solutions_Architect_Slides.md`

בשורש הוולט נמצא קובץ שקפי הקורס המלא (חומר אישי ללימוד עצמי).
זהו **מקור האמת** להרחבת שיעורים — 878 שקפים לפי כותרות `# `.

**איך לעבוד מולו:**

- לאתר את הנושא: `grep -n "^# " AWS_Certified_Solutions_Architect_Slides.md` ואז grep על מונח.
- לקרוא טווח: `sed -n 'X,Yp' AWS_Certified_Solutions_Architect_Slides.md`.
- **לנסח מחדש בעברית במילים שלך.** אין להעתיק שקפים מילה במילה או להדביק בלוקים שלמים.
  שולפים מהשקפים את **העובדות** (מספרים, מגבלות, התנהגות, השוואות) ובונים מהן מערך שיעור.
- אין להעתיק הערות `<!-- Page N -->` או כותרות שקף לקובץ הסופי.
- אין לערוך את קובץ השקפים עצמו — הוא read-only.

בסוף כל שיעור מופיעה שורת **שקפי מקור** עם טווח השורות ששימש אותו, לצורך חזרה או הרחבה עתידית.

---

## פורמט השיעור (מחייב — כל 41 השיעורים אחידים)

כל קובץ ב-`lessons/` בנוי במבנה הבא. שיעור חדש או עדכון של קיים חייב לשמור עליו.

```text
frontmatter: lesson / title / domain / services / tags

# NN — Title
> [!abstract] בשורה אחת
## 🗺️ מפת השיעור          — טבלת תחנות + מונחי מפתח
## 1. 🎯 הבעיה והפתרון
## 2. ⚙️ איך זה עובד        — תתי-סעיפים 2.1, 2.2 ... + דיאגרמות טקסט
## 3. 🔍 פירוק מפורט         — פרמטרים, סוגים, מגבלות, ברירות מחדל
## 4. 💰 עלות ותמחור         — על מה מחייבים / מה זול ומה יקר / עלויות נסתרות / טיפי חיסכון
## 5. ⚖️ השוואות מכריעות     — X vs Y בטבלה
## 6. 🏛️ Well-Architected    — ששת ה-Pillars, ספציפית לנושא
## 7. 🪤 מלכודות במבחן        — מילות מפתח → תשובה + טעויות נפוצות
## 8. 🏗️ Scenario מהעולם האמיתי
## 9. 🚫 מה לא צריך לדעת למבחן
## 10. ⚡ Cheat Sheet
## 11. ✅ בדיקת הבנה          — 3 שאלות + תשובות ב-<details>
## 🔗 קישורים                 — wikilinks + שקפי מקור
```

### שני הסעיפים שאסור לדלג עליהם

**סעיף 4 — עלות ותמחור.**
כל שיעור מסביר על מה בדיוק מחייבים, מה העלויות הנסתרות (data transfer, requests, IOPS,
provisioned capacity, minimum storage duration), ומה זול יותר מול מה יקר יותר.
**אין לתת מחירים מוחלטים בדולרים** — הם משתנים לפי Region ולאורך זמן.
נותנים **יחסים** ("היקר ביותר", "עד ~90% הנחה", "0 עלות", "פי ~X").

**סעיף 6 — ששת ה-Pillars.**
טבלה עם Operational Excellence, Security, Reliability, Performance Efficiency,
Cost Optimization, Sustainability.
התוכן חייב להיות **ספציפי לנושא הנלמד**. טקסט גנרי שאפשר להדביק בכל שיעור — נכשל.

### כללי קריאוּת

- אסור פסקאות ארוכות שמפרידות רעיונות בפסיקים. כל רעיון = שורה או בולט משלו.
- שורה ריקה בין כל בלוק (כותרת / פסקה / טבלה / רשימה / code fence).
- קו הפרדה `---` בין סעיפים ראשיים.
- מעדיפים טבלאות, בולטים, callouts של Obsidian (`> [!tip]`, `> [!warning]`, `> [!info]`)
  ודיאגרמות טקסט ב-code fence — על פני "קיר טקסט".
- אמוג'י רק בכותרות הסעיפים לפי המבנה למעלה. לא בתוך הטקסט.
- אורך יעד: 320–600 שורות לשיעור.
- כתיבת קובץ שיעור: בכלי Write. **לא** דרך heredoc ב-shell — עברית עם backticks ו-`$` נשברת שם.

---

## Choose the correct prompt

- Use `prompts/AWS teacher.md` when the user asks to learn a topic, create a lesson, explain a service,
  continue the curriculum, or review lesson material.
- Use `prompts/AWS questions.md` when the user explicitly asks for exam questions, practice questions,
  a quiz, a mock exam, or an exam based on a topic.
- Use `prompts/subjects.md` as the authoritative order of the 41-lesson curriculum.
  The AWS teacher prompt contains additional topic coverage to check while preparing a lesson.

Do not silently turn a lesson request into an exam, or an exam request into a lesson.
If a request asks for both, complete each part using its corresponding prompt and keep the outputs separate.

---

## Creating or updating lessons

1. Read `AWS teacher.md` and `subjects.md`, plus the lesson-format section above.
2. Identify the requested curriculum topic. Teach only one topic at a time unless the user explicitly
   asks for a combined review.
3. Pull the source material from `AWS_Certified_Solutions_Architect_Slides.md` and rewrite it — never paste it.
4. Follow the mandatory 11-section structure. Sections 4 (cost) and 6 (pillars) are not optional.
5. Save the lesson in `lessons/` with a stable, readable filename carrying the curriculum number:

   `lessons/01 - AWS Fundamentals.md`

6. Do not overwrite an existing lesson without reading it first. Preserve correct content that is already
   there — expand it, don't discard it.
7. Keep the lesson self-contained. Link related lessons with Obsidian wikilinks, e.g. `[[09 - VPC Fundamentals]]`.
   Verify the exact filename before linking — a wrong name is a broken link.
8. Stop after the requested topic. Move to the next curriculum topic only when the user explicitly says
   "נושא הבא" or otherwise clearly asks to continue.

---

## Creating exams and practice sets

1. Read `AWS questions.md` before generating questions.
2. Generate an exam/practice set only when the user requests it. Do not create questions automatically
   after every lesson.
3. Unless the user explicitly requests another amount, create exactly 10 questions.
4. Save the question set as a new Markdown file in `questions/`, for example:

   `questions/2026-08-19 - VPC - Practice Set 01.md`

5. Before the user submits answers, include the questions and four options (A–D), but no answers, hints,
   explanations, or answer letters.
6. When the user submits answers, update the relevant question-set file with the score, explanations,
   exam tips, and analysis of mistakes according to `AWS questions.md`.
   Preserve the original questions and the user's answers.
7. Make questions original. Never claim that a question appeared on the real exam unless a reliable source
   verifies that claim. Use current official AWS documentation and the official exam guide when up-to-date
   information matters.
8. Keep exam content in `questions/`; do not mix it into lesson files.

---

## Working in parallel (multi-agent)

כשמריצים כמה סוכנים במקביל על הוולט:

- כל סוכן מקבל **טווח שיעורים בלעדי** (5–10 שיעורים) ואסור לו לגעת בקבצים של אחרים.
- קובץ השקפים הוא read-only לכולם — קריאה מקבילה בטוחה.
- `prompts/` וה-README מתעדכנים רק על ידי הסוכן המתאם, לא על ידי סוכני המשנה.
- כל הסוכנים חייבים לקבל את אותו מפרט פורמט, אחרת הוולט יוצא לא עקבי.

---

## General operating rules

- Inspect existing files before creating or editing anything, and preserve the user's work.
- Prefer one new file per lesson or question set so the vault remains easy to navigate.
- Use the user's requested language. If no language is specified, retain the bilingual style of the existing
  material: technical terms in English with clear Hebrew explanations.
- Do not invent progress, scores, sources, or completed lessons.
- Do not invent AWS numbers, limits, or prices. If the slides conflict with current AWS behavior,
  write the correct version and flag the discrepancy.
- If the request is ambiguous, make the smallest reasonable assumption and state it briefly.
- Keep the prompts in `prompts/` as instructions/templates. Generated learning material belongs in
  `lessons/` or `questions/`.

---

## Quick routing examples

| User request | Action | Output folder |
|---|---|---|
| "Teach me VPC" / "צור שיעור על VPC" | Follow `AWS teacher.md` + the lesson format above | `lessons/` |
| "נושא הבא" | Continue with the next item in `subjects.md` | `lessons/` |
| "הרחב את שיעור X" | Pull more material from the slides file, keep the same structure | `lessons/` |
| "Give me 10 hard S3 questions" | Follow `AWS questions.md` | `questions/` |
| "Check my answers" | Grade the matching question set and record the review | `questions/` |
| "Create a full mock exam" | Use the questions prompt, keeping the requested scope and format | `questions/` |
