# Igiver Training Platform — Design & Build Documentation

Design documentation for the application that runs the Igiver training programme: partner colleges
send candidates, we select and train them for about a year, certify those who attend enough, place
them in jobs, and pay the trainers and interview panellists who make it happen.

**This repository contains no code.** It is the specification the application gets built from.

---

## Start here

Read in this order. Each document assumes the one before it.

| # | Document | What it gives you | Time |
|---|---|---|---|
| 1 | **APP-Creator-Build-Deck.pdf** | The whole system explained from scratch — the programme, how it becomes forms, and the rules you must enforce. Written for developers new to this project. | 45 min |
| 2 | **APP-Wireframes-Storyboards.pdf** | All 23 screens and 10 user flows. Every screen lists the exact fields it reads and writes. | Reference |
| 3 | **APP-Data-Structure-Detailed.md** | The authoritative spec. All 28 tables, every field typed and described, with the rules and validation queries. | Reference |

If you are building a screen, you will live in documents 2 and 3. Document 1 is read once.

---

## Files

| File | Status | Notes |
|---|---|---|
| `APP-Creator-Build-Deck.pdf` / `.html` | **Current** | Build briefing, targeted at Zoho Creator. 26 slides. |
| `APP-Wireframes-Storyboards.pdf` / `.html` | **Current** | 23 annotated wireframes, 10 storyboards. |
| `APP-Data-Structure-Detailed.md` | **Current — authoritative** | 28 tables. The source of truth for all data questions. |
| `APP-Training-Deck.pdf` / `.html` | Optional | Same briefing, but platform-neutral. Useful if the stack ever changes; **ignore it if you are building in Creator** — its rules section describes database constraints Creator does not have. |
| `APP-Data-Structure-corrected.md` | **Superseded** | An earlier draft, kept for history. Do not build from this. |
| `APP Data Structure.pdf` | **Superseded** | The original design, kept for history. Contains errors that were fixed — see the changelog in the Detailed spec. Do not build from this. |

**On the HTML versions:** GitHub displays them as raw source, not as pages. Download the file and
open it in a browser to get the interactive deck (arrow keys to navigate). The PDFs are the same
content and open anywhere — use those unless you want the interactive version.

---

## Decisions already made

Settled. Do not reopen without discussion.

- **One Zoho Creator application**, with separate logins for admin and trainer via permission sets.
- **Staff log in; nobody else does.** Students, colleges and panel members have no accounts. Panel
  members score candidates on paper, and a coordinator enters the scores.
- **28 tables become roughly 24 Creator forms.** Creator's built-in users and permission sets
  replace two tables; its system fields replace the `id` / `created_at` / `updated_at` columns.
- **Exactly two subforms:** attendance inside a class session, and line items inside an invoice.
  Everything else is a separate form, because something else needs to look up to it.
- **Panels are always exactly three people.** Entrance weights always sum to 1; there is no
  partial-panel case.
- **Certification requires 80% attendance**, identical for every batch. `present` and `late` both
  count as attended; `excused` is excluded from the calculation entirely.
- **Trainers and panel members are both paid hourly** through the same invoicing forms.

---

## Open questions

Three affect the **structure** and should be answered before forms are built — changing a form's
shape after it holds data is painful:

- **Q4** — Does the Zoho vs non-Zoho track change what is taught? Nothing currently links a
  student's track to a set of subjects.
- **Q7** — Does the app generate certificate PDFs, or only record that one was issued? There is a
  template link but nowhere to store the finished file.
- **Q10** — What happens to a student's deposit if they drop out? Refunds are defined only for
  students who complete.

Seven further questions affect screens rather than structure and can be answered while building.
All ten are listed at the end of the wireframes document with the default each currently assumes.

---

## Two rules worth knowing before you write anything

**Creator will not enforce the rules for you.** A full database can refuse invalid data at the
storage layer. Creator cannot — so every rule in the spec lives in an `On Validate` script ending in
`cancel`. Two rules need the hidden-unique-key technique described in the build deck, because they
depend on two fields being unique together: one attendance mark per student per session, and one
class billed only once.

**Copy money, calculate everything else.** When an invoice line is created, copy the hourly rate
onto the line. If it reads the trainer's current rate instead, giving someone a raise silently
rewrites every invoice you have already paid them. Everything that is not a completed financial
record should be calculated from a single source, never stored twice.

---

## Build order

Build one complete journey at a time, not one layer at a time. Each slice is demoable.

1. **Forms and lookups** for institutions, entrance, enrolment and curriculum. No scripts yet.
2. **College → batch → entrance weights.** Proves logins and permission sets work.
3. **Entrance → scoring → selection → onboarding.**
4. **Timetable and attendance.** At this point the app is usable daily — get it right.
5. **Billing and certificates.** Needed monthly, not daily.
6. **Placement.** A year away for the first batch.

Get the lookups and the two subforms right in step 1. Converting a field into a subform later, once
there is data in it, is the one change that really hurts.
