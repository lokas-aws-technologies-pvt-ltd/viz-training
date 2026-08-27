# MyPrayers — prototype

A community prayer counter. Someone opens a **topic** — "Floods in Nepal" — and anyone who joins
prays for it by pressing a button, as many times as they want. Each person addresses whichever
divine they hold; the app counts every prayer twice over: once against that person, once against
the topic as a whole.

Open `index.html` in a browser. Nothing to install, no server, no build step.

> **Read [`Concept-Note.html`](Concept-Note.html) first.** It supersedes the framing below. This
> prototype implements the counting engine only; the concept note covers what the product is
> actually for — being there for someone when you can't be — and the parts still to be designed:
> the recipient's view, consent, the synchronised moment, and how a circle ends.

---

## What it does

| Screen | What happens there |
|---|---|
| **Onboarding** | Give a name, pick who you pray to — 13 traditions, ~35 named focuses, including a humanist "moment of intention". |
| **Topics** | Every open topic, its running total, and a stacked bar showing which traditions are carrying it. Your own count sits on any topic you've joined. |
| **Topic** | The counting screen. A big Pray button ringed by a mala; every press lights the next bead. Below it: what's being asked for, the faith breakdown, who has offered the most, and a live feed. |
| **New** | Open your own topic — title, place, the ask, and a kind. You join it automatically. |
| **You** | Your total across all topics, malas completed, topics joined, days praying, and where your prayers went. |

## How the counting works

Pressing the button once offers one prayer. **Hold it down** to keep praying at about eight a
second — the button is meant to be pressed repeatedly, so holding is the honest version of that.

Counting borrows the mala, which every counting tradition has some form of:

- **27 beads** make a round. The ring lights bead by bead and pulses when it closes.
- **4 rounds** make a full mala of **108**, which is what the "malas completed" figure counts.

Three tallies are kept, and all three are live on screen:

1. `topic.mine` — your prayers on this topic.
2. `topic.tally[tradition]` — the topic's total, broken out by tradition. Summing it gives the
   headline number.
3. `state.total` — your prayers everywhere, which drives the You screen.

You pray as your chosen focus by default, and can address someone else on a single topic without
changing your global choice ("Change" on the praying-as row). Prayers already counted keep the
tradition they were offered in — the tally is never rewritten retroactively.

## What is real and what is mocked

- **Real**: every count, the bead and mala arithmetic, topic creation, the per-topic faith
  override, and persistence. State is in `localStorage` under `myprayers.v1` and survives reloads.
  Storage failures (private windows, blocked site data) are caught — the app then runs from memory.
- **Mocked**: there is no server and no account. The other people praying are simulated — a
  background tick adds prayers from a fixed cast every 1.5–5 seconds, weighted toward whichever
  topic you're looking at so the screen stays alive while you're on it. Seed topics ship with a
  crowd already in place and a short backdated feed.

Everything lives in the one browser. Nothing is sent anywhere. "Reset this prototype" on the You
screen clears it.

## Design notes

- Limewash-and-verdigris ground, brass for anything being counted. Light and dark are both
  designed; the page follows the viewer's theme.
- Marcellus for titles (inscriptional, quiet), Karla for interface text, IBM Plex Mono for every
  tally — counts are the subject, so they get the counting face.
- Each tradition carries one hue, used consistently in the mix bar, the legend, the avatars and
  the rising "+1". In dark mode the hues are lifted through a single `--lift` token rather than
  being redefined thirteen times.
- The app's own mark is a mala, not any one tradition's symbol. No tradition is ranked, and the
  leaderboard sorts people by prayers offered, never by faith.

## If this gets built for real

Three things the prototype sidesteps:

- **A server and identity.** Counts have to be authoritative somewhere, and the tallies are the
  whole product. Increments want to be atomic and batched — a held button generates a lot of them.
- **Abuse.** A count that anyone can drive up by holding a button is trivially inflated. Real
  builds need rate limiting and some notion of a verified participant, or the number stops
  meaning anything.
- **Moderation.** Anyone can open a topic. Topics naming a living person, or a side in a conflict,
  need a review path and a way to close or hide one.
