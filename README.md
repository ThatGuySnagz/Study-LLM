# Study LLM

A GitHub starter any agent can extend. A person downloads this repo, tells their AI to use it for studying, and provides materials. This repository is the base. Their agent builds on it. Nothing here is a hosted product.

Study LLM turns materials the person already has into a local study deck with three modes: flashcards, multiple choice, and written prompts. An agent drafts the items. The student or teacher edits the draft and studies on their own machine.

The deck is a file. Studying that file does not need an account, a class server, or a network connection.

This project is licensed under the MIT License. See [LICENSE](LICENSE).

## Download and tell your agent

1. Download the repo (GitHub template, clone, or zip) onto the machine where the materials live.
2. Open that folder with the agent the person already uses.
3. Put source files in `materials/inbox/`, or paste text and have the agent save the paste as a file there.
4. Tell the agent to use this template. Example: "Use this Study LLM template. Read AGENTS.md. Draft a local deck from `materials/inbox/` and leave it in `decks/`."
5. The agent reads `AGENTS.md` and `PRIVACY.md`, extracts text locally, calls only the model the person configured, and writes `decks/<slug>.json`.
6. The person opens `app/` locally, loads that deck, studies, and edits. Edits save back to the deck file on the machine.
7. A teacher copies the deck file to students by hand (USB, email attachment, or a drive they already use). Each student opens it in their own copy of this template.

The template authors do not receive the materials, the deck, or the model request.

## Who it is for

Same tool, two audiences:

- **Student.** Studying their own notes. They keep materials and the deck on their machine.
- **Teacher.** Building a deck for students from materials on the teacher's machine. They share the deck by giving students that file. There is no hosted classroom, roster, or join code.

## What to put in the inbox

Accepted sources, all read on the device:

- PowerPoint (`.pptx`)
- PDF (`.pdf`)
- Word (`.docx`)
- Photos of handwritten notes (`.jpg`, `.png`, `.webp`, `.heic`)
- Pasted text, saved by the agent as a `.txt` file in `materials/inbox/`
