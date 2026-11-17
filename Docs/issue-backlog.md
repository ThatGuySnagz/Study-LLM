# Study LLM issue backlog

Reference for the open issues on [ThatGuySnagz/Study-LLM](https://github.com/ThatGuySnagz/Study-LLM) after the review edits. Issue 5 no longer exists. The next work is issues 1–3.

## What changed

- Deleted issue 5, "Add deck.schema.json for format version 1". Its schema work is now the body of issue 4. Issue 4 merges only when the valid-deck test is green, so `main` does not stay red.
- Issue 4 now requires `sourceRef` on every item. Issue 20 follows that rule instead of inventing its own.
- Issue 6 adds a second test for `decks/example-deck.json` and leaves the issue 4 fixture test passing.
- Issues 7 and 8 say they depend on the example deck from issue 6.
- Issues 9–12 place the tested functions in `core/` (for example `core/study.js`). Issue 12 names the progress file `decks/<deck-id>.progress.json`.
- Issues 13–17 call `core/` instead of reimplementing filters, scoring, edits, or progress. Their automated coverage is the `core/` tests. Issue 18 is the browser checklist. Those app pull requests do not need a new automated test unless the class requires one in every pull request.
- Issue 13 loads a deck through the file picker or another same-folder mechanism the browser allows. A page opened as a local file often cannot `fetch` another file on disk.
- Issue 19 documents `materials/inbox/` and `materials/extracted/` but does not create those folders.
- Issue 4 notes that a CI workflow that runs the tests can wait.
- Issues 1–3, 21, and 22 were left as they were.

## Milestone 1 — Repo contract

### Issue 1 — Write the download-and-study README

Replace the one-line README with the download-and-tell-your-agent flow from [starter-plan.md](starter-plan.md), and name the MIT license.

Done when a new reader can go from clone to telling their agent what to do without opening the plan file. The README names MIT and points at `LICENSE`. Materials go in `materials/inbox/`, the agent reads `AGENTS.md` and `PRIVACY.md`, the deck lands in `decks/`, and study happens in local `app/`.

### Issue 2 — Add PRIVACY.md

Add `PRIVACY.md` from the six-point outline in the starter plan.

Done when the notice says this is a local template with no account and no service operated by the template. It does not collect notes or keep model requests. Materials, extracted text, decks, and progress stay local unless the person copies them. Content leaves the machine only in a request to the model the person configures, and that provider's terms apply. A teacher who sends student work to a commercial model is responsible for school rules, including FERPA for US schools. The notice does not claim the template is FERPA-compliant, and it says this is not legal advice. Copying a deck file is the person's own act. The template has no server that receives it.

### Issue 3 — Ignore local study files

Keep personal study files out of git. `.DS_Store` is already ignored.

Done when `.gitignore` ignores `materials/inbox/`, `materials/extracted/`, `decks/*.json` except `decks/example-deck.json`, `decks/*.progress.json`, and secret files such as `.env`.

## Milestone 2 — Deck contract

### Issue 4 — Add the deck schema and a passing valid-deck test

Add a Node test runner and `schema/deck.schema.json` for format version 1. The first commit on the branch adds a failing test because the schema file does not exist yet. Later commits add the schema until a valid mixed-deck fixture passes. Merge only when that test is green. Do not add `decks/example-deck.json` here. Incomplete-item cases belong to issue 7. Rejecting a missing or non-1 `formatVersion` stays in issue 8.

The schema requires `formatVersion`, `id`, `title`, and `items`. The valid fixture uses `formatVersion` `1`. `authorLabel` is optional free text. Item `type` is `flashcard`, `multiple_choice`, or `written`. Every item requires `sourceRef`. There is no class id, student id, or roster field.

A CI workflow that runs the tests is out of scope.

### Issue 6 — Commit the public example deck

Depends on issue 4. Commit `decks/example-deck.json` with one flashcard, one multiple-choice item, and one written prompt. Use public sample sentences only.

Add a new test that loads `decks/example-deck.json` and validates it. Leave the fixture test from issue 4 passing. Each item is `included: true` and has a `sourceRef`.

### Issue 7 — Reject incomplete study items

Depends on issue 6. The example deck already validates, and this issue keeps that file passing.

Add failing tests, then update the schema, for a multiple-choice item with no `correctChoiceId`, a flashcard with no `answer`, and a written item with no `referenceAnswer`. `sourceRef` stays as decided in issue 4.

### Issue 8 — Accept only format version 1

Depends on issue 6. Add failing tests for a missing `formatVersion` and for a `formatVersion` other than `1`, then make the schema reject both. `decks/example-deck.json` still passes.

## Milestone 3 — Study core

These functions live in `core/` and do not read the DOM or the network. The app issues import this module.

### Issue 9 — List included items for one study mode

A pure function returns only items with `included: true` whose `type` is the requested mode (`flashcard`, `multiple_choice`, or `written`). An `included: false` item stays in the deck object and stays out of the list.

### Issue 10 — Score a multiple-choice answer

A chosen id equal to `correctChoiceId` records `good`. Any other id records `again`. The deck object is unchanged. The result is progress data. The multiple-choice page calls this function.

### Issue 11 — Keep deck edits and unknown fields

Tests change prompt text, the flashcard answer, choices, the correct choice, the reference answer, the rubric, and `included`. An unknown extra field on an item is still present after the edit. Setting `included` to `false` keeps the item in the file. The edit-and-save page calls these functions.

### Issue 12 — Store practice beside the deck

Practice lives in `decks/<deck-id>.progress.json`. The object has `deckId`, `updatedAt`, and per-item `lastResult` (`again` or `good`) plus `learnerAnswer`. Writing progress does not change deck items. Replacing progress with an empty object leaves the deck intact. The study pages call these functions.

## Milestone 4 — Local study app

`app/` calls `core/`. Rules are tested in issues 9–12. Issue 18 is the browser checklist. Pull requests for issues 13–17 do not need a new automated test unless the class requires one in every pull request. No server is added.

### Issue 13 — Open a deck in the local app

Show the deck title and the three modes. The mode list comes from the `core/` filter. Load `decks/example-deck.json` through the file picker or another same-folder mechanism the browser allows. Do not use `fetch` of a disk path as the way this issue is done.

### Issue 14 — Study flashcards

`prompt` is the front. A flip reveals `answer`. Again and Good write `lastResult` to `decks/<deck-id>.progress.json` through `core/`. The deck file does not gain a score.

### Issue 15 — Study multiple choice

Show the stem and choices. The page calls the `core/` scorer: `correctChoiceId` records `good`, any other choice records `again`. Show `explanation` after an answer when the item has one. Store the score in `decks/<deck-id>.progress.json`.

### Issue 16 — Study written prompts

Show `prompt`, `referenceAnswer`, and `rubric`. Save the attempt as `learnerAnswer` in `decks/<deck-id>.progress.json` through `core/`. The deck file does not contain the attempt. The app does not send the attempt to a model and does not grade it.

### Issue 17 — Edit a deck and save the file

Change prompt text, answers, choices, the correct choice, the reference answer, the rubric, and `included` by calling `core/`. Save with the File System Access API where the browser has it, and download the updated file otherwise. Unknown fields survive the save.

### Issue 18 — Manually pass the example deck

Check these in a browser before calling the app done:

- Open `decks/example-deck.json` with the file picker, or another same-folder mechanism the browser allows.
- Study one flashcard and record Again or Good.
- Answer the multiple-choice item once correctly and once incorrectly.
- Write an attempt on the written prompt and confirm it is not inside the deck file.
- Confirm practice was written to `decks/<deck-id>.progress.json`.
- Edit an item, save, reload, and see the edit.
- Remove the progress file and confirm the deck is unchanged and practice is reset.

## Milestone 5 — Agent workflow

### Issue 19 — Write AGENTS.md

Tell the person's agent to read `materials/inbox/`, write `materials/extracted/<slug>.json`, emit format version 1 items, treat the deck as a draft, and write only local files. The agent stops and asks for a transcription or paste when a configured model cannot read a photo. The agent leaves out accounts, a classroom backend, analytics, and a service run by this template.

Creating `materials/inbox/` and `materials/extracted/` in git, including a `.gitkeep`, can wait.

### Issue 20 — Write the draft-deck prompt

`prompts/draft-deck.md` tells the agent how to turn extracted text into a draft deck. It points at `schema/deck.schema.json` and `decks/example-deck.json`. It asks for `flashcard`, `multiple_choice`, and `written` items with `included: true`. It requires `sourceRef` because issue 4 made that field required. The result is a draft written to `decks/<slug>.json` on the machine.

### Issue 21 — Add the model config example

Add `config.example.json` with a model name and an optional base URL, and no API key. The README says the person's agent uses this config, or a local untracked config, when drafting. The README says `app/` can open and edit a deck when no model is configured.

## Decision

### Issue 22 — Decide what scale means before any backend

This issue does not block the local milestones. Record one choice in a comment:

- More students each open their own copy of a deck file.
- A later hosted service, which would need its own privacy update and a new milestone.

Accounts, a classroom, and a server stay unbuilt until that comment exists.
