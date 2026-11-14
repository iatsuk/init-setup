# CLAUDE.md — {{project}}

{{One or two sentences: what this project is, what it talks to, and where the neighbouring
components' CLAUDE.md files are.}}

**This file is an orientation map and a set of rules, not documentation.** It carries what is hard
to discover by reading the code. For anything a grep answers in seconds, read the code.

This is production code: stability is the goal, and every change is judged on what else it might
move.

{{Stack in one line: language · framework · build tool · test framework.}}

## Scope

<!-- What this team owns, and the neighbouring directories that belong to someone else. -->

{{owned paths}} belong to this team. {{other paths}} belong to other teams — do not change them.

## Where things live

<!-- One row per top-level package or directory. Name its role, not its contents. Flag anything
that lives somewhere surprising, and anything that is generated. -->

| | |
|---|---|
| `{{path}}` | {{role}} |

## Things that will cost you a day if you don't know them

<!-- Only facts the code in front of the reader does not show. Typical candidates:
- generated code, and the source it is generated from
- a schema or contract file that is the single source of truth, and who else consumes it
- registration by convention: classpath scan, decorators, a hand-maintained suite or route list
- initialisation-order constraints that break something far away
- behaviour matched on strings: error codes on message text, routing on names
- persisted data whose meaning is keyed on a field, and how it is versioned
- what CI actually runs, and on which branches, titles or tags
- configuration overridden in deployment, so local values are not what runs
- dependency pins that exist for security reasons
Delete this comment once the list exists. -->

- **{{trap}}.** {{why it bites, and what to do instead}}

## Contracts with the outside

<!-- Who calls this project, how, and what they read back: the interfaces that change only with
the other side's agreement. -->

## Conventions

<!-- Commit message format and its scopes, formatter and style configuration. -->

## Working agreements

These are what a reviewer checks. They apply to every change in this project.

### Git

Never `git commit` or `git push` unless the developer asks for it in that moment. Leave the work
in the working tree and describe what changed. Permission to commit one change does not carry over
to the next.

### Build and tests

Do not run compilation, build or test commands — the developer does that. Use source inspection
and non-build checks only, unless the developer explicitly lifts this rule.

### Production code, and blast radius

This project runs in production. Stability outranks elegance, and an unintended behaviour change
is worse than a missing improvement.

So when a change touches code with more than one caller — a shared utility, a base class a family
of classes extends, a public API, anything called across a language or process boundary — being
convinced it is correct is not enough:

- Find every caller before editing (`grep` the symbol; do not rely on the callers you happen to
  know about) and check the change against what each of them expects.
- State the result. Either "behaviour-preserving for all N callers" with the list, or "this
  changes behaviour for X, deliberately, because …". Silence here reads as "not checked".
- When a shared helper needs to behave differently for one caller, prefer a new parameter with the
  old default, or a separate function, over changing what the existing callers get — but never a
  boolean that switches the helper between two jobs.
- If you cannot establish the blast radius from the source, say so and ask, rather than shipping a
  change whose reach you are guessing at.

### Design principles

Follow SOLID, DRY, KISS and YAGNI. Reach for a GoF or GRASP pattern when it makes the design
clearer, more flexible or easier to maintain — never to demonstrate the pattern. Prefer:

- clarity over cleverness
- simplicity over premature abstraction
- composition over unnecessary inheritance
- explicit design over hidden behaviour
- testability over convenience

Here that means:

- **Entry points are controllers, not logic.** A view, a CLI command or a request handler parses,
  resolves and dispatches; anything worth asserting lives in the domain layer, where it tests
  without the framework.
- **Extend at the extension point.** A new variant is a new implementation of the existing seam,
  not another mode flag inside an existing implementation.
- **Depend on the narrowest interface that fits**, not on the framework underneath it. That is both
  the interface-segregation seam and what makes a unit testable in isolation.
- **Reuse the seams that exist** — the task runner, the API client, the persistence layer, the
  notification channel. A second mechanism alongside an existing one is what needs justifying.
- **No hidden coupling across a language or process boundary**: use the documented contract, do not
  invent ad-hoc channels.
- **YAGNI**: no configuration property, setting, extension point or generic parameter for a caller
  that does not exist yet.
- **Change the design when a feature does not fit it**; do not bolt the feature on. A class keeps
  one responsibility: no mode flags, no method that works only after another one ran, no `null`
  meaning "nothing to do", one mechanism per job.
- **One subsystem, one package**, with dependencies between packages pointing one way.
- **Delete code that adds nothing.**

### Fit the surrounding code

New code should be hard to tell apart from the code already there. Before writing any, read the
nearest existing equivalent and copy its shape — naming, structure, error handling, logging, how
it gets wired — rather than importing habits from elsewhere.

- A new member of a family (a handler, a plugin, a format, a dialog) copies a sibling in the same
  package: how it is declared, configured, wired, validated, and how it reports back.
- A new service follows the existing ones' dependency injection and configuration style.
- A new call to an external system goes through the same client and execution path as every other.

When a house pattern and a general best practice disagree, follow the house pattern and say that
you noticed: consistency across the project beats local optimality, and a lone island of a better
idiom costs the next reader more than it saves. Do not introduce a new library or idiom where an
established one already covers the case, and do not restyle the surrounding code to match your
preference.

### Precise changes and readability

- The reviewer does not want to re-review existing, working code. Keep every change as small and
  as local as the task allows.
- Do refactor when the result is genuinely smaller, simpler, safer or easier to read. A smaller
  diff does not beat clearly better code — but it does beat merely rearranged code.
- Small functions are a means, not a goal. When a reader has to jump through a chain of one- and
  two-line helpers to follow a single thought, readability has gotten worse. Never write a method
  of a few lines whose single caller could hold its code — put the code there, guard clauses
  included. That holds for methods that only forward a call, and over "copy its shape" when the
  neighbour you copy delegates the same way. Inlining one is not duplication.
- Real duplication is two functions doing the same thing with a difference that can become a
  parameter. Extract the difference into an argument; a lambda is a perfectly good argument.

### New functionality

New functionality is isolated functionality: its own class or package, as few couplings as
possible, meeting the rest of the system through a **single entry point** — one class, one
service, one factory method. If it only works after edits scattered across existing classes, the
entry point has not been found yet.

### Code that explains itself

The code is the documentation: names, packages and signatures make a class clear without comments.
Every comment is read on every visit, by developers and LLMs alike; one that repeats the code costs
time and tokens and adds nothing.

- **Name things for what they are in the domain**, not with words that fit anything. A boolean
  names the question it answers. Where a concept has an established name, use it.
- **Fix unclear code before commenting it.** If a comment is needed to say *what* the code does,
  rename or restructure instead. A comment says *why*: a constraint, a trade-off, a library quirk.
- **A class doc comment is one or two sentences on the class's role in the system.** It stays true
  when the implementation changes; one that needs editing with every change describes the
  implementation — cut it.
- No doc comment that repeats the signature, no comments on self-describing fields, getters or
  overrides.
- Comments describe **current behaviour only** — no "previously", no "instead of", no migration
  notes. That belongs in the commit message and the PR description. English only.

Where a file's choices are genuinely non-obvious and expensive to rediscover, its comments may be
dense — but they still say why, and new code there matches that density.

### Texts in the developer's name

A commit message, a PR description, a review reply, an issue or a mail to a list goes out under the
developer's name and is read by people who give their time to it. It has to read as the developer
wrote it. Nothing is posted, pushed or sent before the developer has approved the exact text.

- **Write as the developer speaks.** Plain English with the correct technical terms, short
  sentences, no polished or idiomatic phrases. Do not state agreement or opinions for the developer
  ("that makes sense", "I agree that…"); start with what was done.
- **One idea per sentence.** A sentence that packs several facts, numbers and brackets is easy for
  a model and hard for a person. Say what it means first, then the detail, and leave out numbers
  and identifiers the reader does not need to follow the point.
- **A reply is a conversation, not a publication.** It addresses the person it answers. Someone who
  opens the thread later should still follow it: a short summary first, and a sentence before each
  table that says what it shows. But no paragraph addressed to an audience, and no explaining to a
  reviewer what the reviewer proposed.
- **Show doubt where there is doubt.** Where a choice is open, say what was considered and what it
  would cost, and ask for the other person's view ("I could try X, but it makes the solution more
  complex. What do you think?"), not a flat statement followed by an instruction.
- **Thank people for their time**, in plain words, where it is true.
- **Answer once.** No interim "I will do X" message when the result follows within a day; one reply
  with the change and the numbers.
- **One issue carries the history.** When the approach changes, update the existing issue and keep
  the earlier approaches, with the reason each was dropped, in its description.

### Tests

Do not write exhaustive tests that walk every function and every edge case. Cover the common
cases; one case may assert several related behaviours rather than being split across many. A new
focused test is what you add when a bug turns up. Stay in the style of the existing suite — its
framework, its fixtures, and whether it mocks at all. A test name states what it asserts in at
most five words, without articles; the detail belongs in its assertions.

### Performance work

Any change that can affect performance has to be measured, not argued. Report a table — before vs.
after, with the input, its size and the metric (run time, payload size, …) — and say which branch
or commit each column came from.

When you cannot build or restart anything yourself, ask the developer to switch branches and
restart between runs, saying exactly which branch you need and what you will measure. Benchmark
against {{the benchmark data and where it lives}} and name the input you used.

### Keeping this file honest

This file is only useful while it is true, and it goes stale silently.

- Fix a line that contradicts the code **in the same change** that found it, and say so in your
  response.
- Three or more contradictions, or a whole section describing something that no longer exists,
  means the file is no longer trustworthy: stop, re-verify that section against the code, correct
  it, and tell the developer what was wrong. Do not keep working from a section you have caught
  lying.
- Fix facts freely; never quietly drop or soften a rule. Rules change only when a developer says
  so.
- Keep it short. If a section grows longer than the code it describes, cut it back to the
  non-obvious part — anything a grep answers in seconds does not belong here, and every line costs
  tokens on every task in this project.
