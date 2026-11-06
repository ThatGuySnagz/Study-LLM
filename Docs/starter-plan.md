# Study LLM — product plan

A GitHub starter any agent or LLM can extend. A person downloads the repo, tells their AI to use it for studying, and provides materials. This repository is the base. Their agent builds on it. Nothing here is a hosted product.

## Purpose

Study LLM turns materials the person already has into a local study deck with three modes: flashcards, multiple choice, and written prompts. An agent drafts the items. The student or teacher edits the draft and studies on their own machine.

The deck is a file. Studying that file does not need an account, a class server, or a network connection.

## Who it is for

Same tool, two audiences:

- **Student.** Studying their own notes. They keep materials and the deck on their machine.
- **Teacher.** Building a deck for students from materials on the teacher's machine. They share the deck by giving students that file. There is no hosted classroom, roster, or join code.

## Download and tell your agent

1. Download the repo (GitHub template, clone, or zip) onto the machine where the materials live.
2. Open that folder with the agent the person already uses.
3. Put source files in `materials/inbox/`, or paste text and have the agent save the paste as a file there.
4. Tell the agent to use this template. Example: "Use this Study LLM template. Read AGENTS.md. Draft a local deck from `materials/inbox/` and leave it in `decks/`."
5. The agent reads `AGENTS.md` and `PRIVACY.md`, extracts text locally, calls only the model the person configured, and writes `decks/<slug>.json`.
6. The person opens `app/` locally, loads that deck, studies, and edits. Edits save back to the deck file on the machine.
7. A teacher copies the deck file to students by hand (USB, email attachment, a drive they already use). Each student opens it in their own copy of this template.

The template authors do not receive the materials, the deck, or the model request.

## Template vs what the person's agent adds

| In the template (scaffold this) | The person's agent adds |
| --- | --- |
| `README.md` with the flow above | Reading the person's actual files |
| `LICENSE` (MIT) | — |
| `PRIVACY.md` from the outline below | Calling the model the person configured (their API or a local model) |
| `AGENTS.md` rules for draft, edit, and no upload | The draft items for that material |
| Deck schema and one example deck with no personal notes | Course or note-specific decks |
| Static local study app: three modes, edit, save | Fixes when extraction is thin (ask for a clearer photo or a paste) |
| `config.example.json` (model name and base URL only) | Secrets and any local-model setup, kept out of git |
| `.gitignore` for materials, personal decks, progress, secrets | — |

The agent must not add accounts, a classroom backend, analytics, or a service run by this template.

## Inputs

Accepted sources, all read on the device:

| Input | Local form | `sourceRef` examples |
| --- | --- | --- |
| PowerPoint | `.pptx` in `materials/inbox/` | `slides.pptx slide 4` |
| PDF | `.pdf` in `materials/inbox/` | `chapter.pdf page 2` |
| Word | `.docx` in `materials/inbox/` | `notes.docx heading "Mitosis"` |
| Photos of handwritten notes | `.jpg`, `.png`, `.webp`, `.heic` in `materials/inbox/` | `page1.jpg` |
| Pasted text | Agent saves it as `materials/inbox/pasted-<short>.txt` before drafting | `pasted-chapter3.txt` |

Extraction writes `materials/extracted/<slug>.json` and stays on disk:

```json
{
  "sources": [
    {
      "file": "materials/inbox/chapter.pdf",
      "kind": "pdf",
      "parts": [{ "ref": "chapter.pdf page 2", "text": "..." }]
    }
  ]
}
```

`kind` is `pptx`, `pdf`, `docx`, `image`, or `paste`.

Handwritten photos need a vision-capable model the person configured. If the configured model cannot read images, the agent stops and asks the person to transcribe or paste the text. It does not drop the photo silently and does not send the image anywhere except that model request.

The study app does not re-upload inbox files. After the deck exists, study and edit work from the deck file alone.

## Study modes

Every item has `type`: `flashcard`, `multiple_choice`, or `written`. A deck may mix all three. The app offers one mode at a time and lists only items of that type with `included: true`.

**Flashcards.** `prompt` is the front. `answer` is the back. The person flips to reveal. No score leaves the machine.

**Multiple choice.** `prompt` is the stem. `choices` is a list of `{ "id", "text" }` (aim for 3–5, at least 2). `correctChoiceId` is the source of truth. Optional `explanation` shows after the person answers. Session right/wrong counts live in the progress sidecar, not in the shared deck.

**Written prompts.** `prompt` is the question. `referenceAnswer` is the draft answer the teacher or student can edit. Optional `rubric` is a short checklist. The person's attempt is stored in the progress sidecar, not inside the shared deck file. Checking an attempt against the reference is a later, explicit request to the person's own agent. The first scaffold does not send attempts to a model by itself.

## Generate, then edit

1. Agent reads inbox files and writes the extraction JSON locally.
2. Agent sends material text (and images, when the model can read them) only in the model request the person configured. The template does not keep a copy of that request.
3. Agent writes a **draft** deck: `decks/<slug>.json`. Every item starts `included: true`.
4. The person reviews in the app or in the JSON. They can change prompt text, answers, choices, the correct choice, the reference answer, the rubric, and whether the item stays in the deck.
5. Save writes the same deck file. `included: false` drops an item from study and keeps it in the file so it can be restored.
6. Nothing in this loop syncs to a server.

AI text is a draft. The person is the editor.

## Local file layout

```
Study-LLM/
  README.md
  PRIVACY.md
  AGENTS.md
  .gitignore
  config.example.json
  schema/
    deck.schema.json
  prompts/
    draft-deck.md
  app/
    index.html
    styles.css
    study.js
  materials/                 # contents gitignored
    inbox/
    extracted/
  decks/
    example-deck.json        # committed; no personal notes
                             # personal decks gitignored
```

`.gitignore` ignores `materials/inbox/`, `materials/extracted/`, `decks/*.json` except `decks/example-deck.json`, `decks/*.progress.json`, and `.env` / secret files.

`config.example.json` names the model and an optional OpenAI-compatible base URL. It has no API key. The person's agent reads a local untracked config or environment variable when it drafts. The study app does not need that config to open a deck.

### Deck file (`schema/deck.schema.json`)

```json
{
  "formatVersion": 1,
  "id": "uuid",
  "title": "Chapter 3",
  "authorLabel": "",
  "items": [
    {
      "id": "uuid",
      "type": "flashcard",
      "included": true,
      "prompt": "Front text",
      "answer": "Back text",
      "sourceRef": "chapter.pdf page 2"
    },
    {
      "id": "uuid",
      "type": "multiple_choice",
      "included": true,
      "prompt": "Stem",
      "choices": [
        { "id": "a", "text": "..." },
        { "id": "b", "text": "..." }
      ],
      "correctChoiceId": "a",
      "explanation": "",
      "sourceRef": "slides.pptx slide 4"
    },
    {
      "id": "uuid",
      "type": "written",
      "included": true,
      "prompt": "Question",
      "referenceAnswer": "Draft answer the person can edit",
      "rubric": "",
      "sourceRef": "pasted-chapter3.txt"
    }
  ]
}
```

`authorLabel` is optional free text the person types. It is not an account. There is no class id, student id, or roster field.

`formatVersion` stays `1` until a later change breaks the shape. Unknown fields are preserved on save so a person's agent can add notes without the app stripping them.

### Progress sidecar (not shared)

`decks/<deck-id>.progress.json` is gitignored and omitted when a teacher copies a deck:

```json
{
  "deckId": "uuid",
  "updatedAt": "ISO-8601",
  "items": {
    "<itemId>": {
      "lastResult": "again",
      "learnerAnswer": ""
    }
  }
}
```

`lastResult` for flashcards and multiple choice is `again` or `good`. `learnerAnswer` is the written attempt. Deleting the sidecar resets practice and leaves the deck intact.

## How a teacher shares a deck

There is no server. Sharing is copying one JSON file.

1. Teacher finishes edits so `decks/<slug>.json` matches what students should see.
2. Teacher copies that file only. Leave `materials/`, `materials/extracted/`, and `*.progress.json` on the teacher's machine.
3. Teacher sends the file by a channel they already trust (USB, email, school drive, and similar). The template does not provide a link, login, or upload step.
4. Student puts the file in their own `decks/` folder and opens it in `app/`.
5. The student's practice stays in their progress sidecar. Handing the original deck file back does not include those attempts.

Default deck contents are study items, not the original student submissions. A source excerpt belongs in an item only when the teacher chooses to put it there, because that text travels inside the file they send.

## Privacy notice outline

Ship a short `PRIVACY.md` in the repo. It is a local-first, no-account notice. It is not a SaaS privacy policy. Use plain language:

1. **What this is.** A template the person runs on their own machine. No account. No service operated by the template.
2. **Notes.** The template does not collect notes. It does not keep a copy of model requests.
3. **Local files.** Materials, extracted text, decks, and progress stay in local folders unless the person copies them somewhere else.
4. **AI calls.** Content leaves the machine only in a request to the model the person configures (their API, or a local model). That provider's terms apply to the call. The template does not store the request.
5. **Teachers and student work.** A teacher who sends student work to a commercial model is responsible for school rules, including FERPA for US schools. Do not claim the template is FERPA-compliant. State that this is not legal advice.
6. **Sharing.** Copying a deck file is the person's own act. The template has no server that receives it.

## License

The repository uses the **MIT License**. Put the full MIT text in `LICENSE`, and name MIT in the README.

MIT matches a starter other people and their agents download and change, including for class. They can use and modify it without asking first. The requirement is to keep the copyright and license notice.

Do not add a commercial license or a terms of service. This template is not a hosted service.

Apache 2.0 would add a patent grant. GPL would require modified copies to stay open. Neither is the choice for this template. If a school or employer will own the repo, they may require a different license before publish. This is not legal advice.

## Non-goals

- Accounts, sign-in, or user profiles.
- A cloud classroom, roster, assignments inbox, or live session.
- A hosted backend or upload API run for this template.
- A SaaS privacy policy, telemetry, or ads.
- Storing model requests in the repo or the app.
- A claim that the template is FERPA-compliant, and any legal advice.
- Automatic grading. Written attempts are local; a model sees them only if the person later asks their agent.
- Requiring an LMS.

## Scaffold first

Build in this order when implementation starts:

1. **Repo contract.** `README.md` (the download-and-tell-your-agent steps), `LICENSE` (MIT), `PRIVACY.md` from the outline above, and `.gitignore` for materials, personal decks, progress files, and secrets.
2. **Deck format.** `schema/deck.schema.json` and `decks/example-deck.json` with one flashcard, one multiple-choice item, and one written prompt. Use public sample sentences only.
3. **Agent rules.** `AGENTS.md` plus `prompts/draft-deck.md`: accept the five inputs, write extraction JSON, emit `formatVersion: 1` items, treat output as a draft, write only local files, and do not add a backend or accounts.
4. **Local study app.** Static `app/` that opens a deck JSON, runs the three modes, edits prompt text, answers, choices, and `included`, and saves the deck. Use the file picker (File System Access API where the browser has it) and a download-the-file fallback. Do not add a server to save files. Progress goes to the sidecar.
5. **Inbox convention.** `materials/inbox/` and `materials/extracted/`, documented in `AGENTS.md`, including the stop-and-ask rule when a photo cannot be read.
6. **Model config example.** `config.example.json` with model name and optional base URL, no secrets. Document that the person's agent uses it when drafting. The study app must still open and edit a deck with no model configured.
