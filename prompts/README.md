# AWS SAA-C03 Study Workspace — Instructions for AI Agents

Read this file before working in this folder. These instructions apply to every AI agent using this workspace.

## Workspace layout

```text
aws/
├── lessons/                  # Completed AWS lessons
├── questions/                # Exam/practice question sets and answer reviews
└── prompts/
    ├── README.md             # This file: agent workflow
    ├── AWS teacher.md        # Lesson-generation instructions
    ├── AWS questions.md      # Exam/practice-question instructions
    └── subjects.md           # Ordered SAA-C03 curriculum
```

The workspace is an Obsidian vault. Use Markdown files and preserve the existing Hebrew/English style. Do not put generated lessons or exams inside `prompts/`.

## Choose the correct prompt

- Use `prompts/AWS teacher.md` when the user asks to learn a topic, create a lesson, explain a service, continue the curriculum, or review lesson material.
- Use `prompts/AWS questions.md` when the user explicitly asks for exam questions, practice questions, a quiz, a mock exam, or an exam based on a topic.
- Use `prompts/subjects.md` as the authoritative order of the 41-lesson curriculum. The AWS teacher prompt contains additional topic coverage to check while preparing a lesson.

Do not silently turn a lesson request into an exam, or an exam request into a lesson. If a request asks for both, complete each part using its corresponding prompt and keep the outputs separate.

## Creating lessons

1. Read `AWS teacher.md` and `subjects.md` before creating a lesson.
2. Identify the requested curriculum topic. Teach only one topic at a time unless the user explicitly asks for a combined review.
3. Follow the teacher prompt's required structure, including the explanation, exam-critical points, comparisons, traps, and real-world scenario.
4. Save the completed lesson as a new Markdown file in `lessons/`.
5. Use a stable, readable filename with the curriculum number and topic, for example:

   `lessons/01 - AWS Fundamentals.md`

6. Do not overwrite an existing lesson without checking it first. If the topic already has a lesson, update it only when the user asks for an update or when creating a clearly named revision is more appropriate.
7. Keep the lesson self-contained. Add links to related lessons with Obsidian wikilinks when useful, for example `[[09 - VPC Fundamentals]]`.
8. Stop after the requested topic. Move to the next curriculum topic only when the user explicitly says “נושא הבא” or otherwise clearly asks to continue.

## Creating exams and practice sets

1. Read `AWS questions.md` before generating questions.
2. Generate an exam/practice set only when the user requests it. Do not create questions automatically after every lesson.
3. Unless the user explicitly requests another amount, create exactly 10 questions.
4. Save the question set as a new Markdown file in `questions/`, for example:

   `questions/2026-08-19 - VPC - Practice Set 01.md`

5. Before the user submits answers, include the questions and four options (A–D), but no answers, hints, explanations, or answer letters.
6. When the user submits answers, update the relevant question-set file with the score, explanations, exam tips, and analysis of mistakes according to `AWS questions.md`. Preserve the original questions and the user's answers.
7. Make questions original. Never claim that a question appeared on the real exam unless a reliable source verifies that claim. Use current official AWS documentation and the official exam guide when up-to-date information matters.
8. Keep exam content in `questions/`; do not mix it into lesson files.

## General operating rules

- Inspect existing files before creating or editing anything, and preserve the user's work.
- Prefer one new file per lesson or question set so the vault remains easy to navigate.
- Use the user's requested language. If no language is specified, retain the bilingual style of the existing prompts: technical terms in English with clear Hebrew explanations where appropriate.
- Do not invent progress, scores, sources, or completed lessons.
- If the request is ambiguous, make the smallest reasonable assumption and state it briefly in the generated file or response.
- Keep the prompts in `prompts/` as instructions/templates. Generated learning material belongs in `lessons/` or `questions/`.

## Quick routing examples

| User request | Action | Output folder |
|---|---|---|
| “Teach me VPC” / “צור שיעור על VPC” | Follow `AWS teacher.md` | `lessons/` |
| “נושא הבא” | Continue with the next item in `subjects.md` | `lessons/` |
| “Give me 10 hard S3 questions” | Follow `AWS questions.md` | `questions/` |
| “Check my answers” | Grade the matching question set and record the review | `questions/` |
| “Create a full mock exam” | Use the questions prompt, keeping the requested scope and format | `questions/` |
