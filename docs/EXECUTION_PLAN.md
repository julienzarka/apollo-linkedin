# Execution plan

This is a plan, not implemented functionality. Task status lives **only** in
[TASKS.md](../TASKS.md). [SPEC.adoc](SPEC.adoc) defines behavior;
[ARCHITECTURE.adoc](ARCHITECTURE.adoc) defines implementation boundaries.
Do not duplicate their requirements into a second mutable specification.

## How to execute

Work one task at a time on a feature branch. Do not implement everything in one
pass. For behavioral work: write the named failing test, run it and record the
expected assertion/import failure, implement the smallest change, run targeted
tests, then the accumulated suite/lint/build. Keep provider access tests separate
from offline tests. Each completed task gets a concise evidence report.

These commands become available after T01; they are not runnable in the current
documentation-only repository:

```text
uv sync --locked --group dev
uv run pytest tests/unit/test_normalize.py -q
uv run pytest -q
uv run ruff check .
uv run ruff format --check .
uv build
git diff --check
git status --short
codemap --diff --ref master
```

Pin supported dependencies during T01. Implement standard CLI help first, not
every command as an empty stub. Configure pytest's offline suite so accidental
network calls fail; live checks require an explicit separate command/marker and
must never run in public CI. Ignore private logs and data.

## Dependency order and gates

```text
T00: account probes and real contract evidence ---------------------+
T01 -> T02 -> T03                                                  |
           -> T04                                                 |
T01 -> T05 -> T06 (Apollo) <----------------------------------------+
           -> T07 -> T08 (Gmail) <----------------------------------+
           -> T09 (LinkedIn snapshots) <----------------------------+
T02 + T04 + verified export shapes -> T10                           |
T03 + T04 + T06 + T08 + T09 -> T11 -> T12 -> T13                    |
T09 + T11 + measured freshness/event support -> T14 <---------------+
T00 through T14 -> T15
```

T00 may remain pending while T01–T05 and synthetic adapter work proceed. Adapter
tasks cannot be called live verified without their T00 evidence. T10 fallback
and T14 freshness are part of the delivery; neither may silently replace a
missing required capability. Do not spawn agents unless the owner requests it.

Milestones: (1) tested domain/store; (2) all three adapters offline verified;
(3) usable local table; (4) live capability verification and release readiness.
Milestones do not replace the finer remaining-task list.

## BOOT — Documentation and repository bootstrap

**Why:** give the execution agent a stable, reviewable contract before code.

**Deliver:** researched feasibility and Five Whys; adapted English agent rules;
remaining-only tasks; specification; access checklist; design; task cards;
handoff; MIT license; private-data exclusions; public GitHub repository.

**Verify:** render AsciiDoc without errors; check local links and cross-task
references; review terminology and dependencies; inspect staged files for data
or credentials; verify remote visibility and license after pushing documentation.
Do not invent application tests or claim integrations work. Record first-commit
limitations for Codemap. Remove BOOT only after the actual publication checks.

## T00 — Verify access and freeze adapter contracts

**Why / what:** eliminate the largest uncertainty before an agent invents API
fields or builds a scraper. Follow [ACCESS.adoc](ACCESS.adoc), all four sections.

**Own:** private capability evidence; sanitized field-path tables in ACCESS;
independently synthetic fixtures in `tests/fixtures/` when T01 exists.

**Steps:**

1. Obtain owner-controlled setup for the scoped Apollo key, Gmail desktop OAuth,
   and LinkedIn Member product. Never ask for token values in a chat message.
2. Probe the documented read routes with a small list and small page size.
3. Prove current sequence membership, sender scope, timestamp semantics, matching
   identifiers, and each provider's pagination/termination behavior.
4. Compare known existing invitations, connections, direct messages, and Sales
   Navigator items; measure or explicitly leave unknown source freshness.
5. Inspect exported CSV schemas only when supplied; record a supported schema
   version/header signature, not an assumed filename contract.
6. Record per-capability results. Update the relevant adapter card only when
   evidence requires it. Keep unverified capabilities and their blockers visible.

**Verification:** private evidence names endpoint, status, shape, scope, paging,
date semantics, and sample comparison; shared reports contain only counts and
field paths. An unavailable permission is not an empty list. Documentation-only
contract discovery does not require a manufactured failing test.

**Stop locally:** if authentication/consent is unavailable, report the precise
missing setup and continue T01–T05. Do not switch to an undocumented API.

## T01 — Minimal package and reproducible checks

**Depends on:** none. **Own:** `pyproject.toml`, `uv.lock`, minimal package entry
point, `tests/acceptance/test_cli_help.py`, test configuration, README setup notes.

**RED:** run an installed/module CLI help test in a subprocess; assert exit 0,
English usage, and that it needs no private config or network. It fails because
the package/entry point is absent. Record this failure before implementing help.

**GREEN:** create a Python 3.12+ package named `apollo-linkedin`, version `0.1.0`,
src layout and argparse help. Add only HTTPX, Google's auth libraries, and needed
dev tooling; defer unused runtime imports until their tasks. One version source.
Lock dependencies. A small test fixture rejects unintended network access.

**Verify:** targeted help test, full suite, Ruff, build, install built wheel into
an isolated environment and run `--help`. No real provider requests. The initial
package should not expose unimplemented commands as successful operations.

## T02 — Normalized records and coverage

**Depends on:** T01. **Own:** `models.py`, `normalize.py`,
`tests/unit/test_normalize.py`, `tests/unit/test_coverage.py`.

**RED:** parameterize tests for UTC conversion, date-only retention, invalid dates,
DST-ambiguous naive times, whitespace in email, alias non-equivalence, canonical
public URLs, unknown Sales Navigator IDs, and complete-empty versus unavailable.

**GREEN:** small typed records and pure normalization functions. Return useful
validation issues without dumping raw records. Keep source time, observation
time, precision, and coverage separate. Validate provider boundaries; retain
unknown fields only when required for an explicit evidence decision.

**Verify:** focused tests plus all current checks. Round-trip normalized dates
through serialization. Strong input errors must not silently coerce to null.
**Maps to:** S01, S04, S07, S08.

## T03 — Matching and reconciliation

**Depends on:** T02. **Own:** `reconcile.py`,
`tests/unit/test_matching.py`, `tests/unit/test_projection.py`.

**RED:** two same-name contacts; duplicate URLs; contradictory email and URL;
manual mapping persistence input; Apollo and newer Gmail send; newer incoming
email; draft; delivered-only record; incoming invitation then connection;
outgoing invitation then later connection; vanished pending request; same-day
date-only ambiguity; six out-of-order messages; group message; older reimport.

**GREEN:** pure functions accepting normalized records and explicit owner
identities. Implement unique-identifier matching, current-membership projection,
latest-known email, invitation evidence, and last-five direct messages exactly
as SPEC. Do not perform I/O or guess identifiers in these functions.

**Verify:** assert both desired outputs and forbidden transitions (no name-only
merge; no acceptance from absence; no false “never emailed”). Stable tie-breaks
make input ordering irrelevant. **Maps to:** S02–S07.

## T04 — SQLite state, transactions, and forgetting

**Depends on:** T02. **Own:** `store.py`,
`tests/integration/test_store.py`, `tests/integration/test_idempotency.py`.

**RED:** import the same batch twice; inject a mid-transaction error; apply an
older archive; fail page two of list membership; replace a complete membership
set; store date-only values; conflict with a second writer; forget one contact
and verify no cached preview/mapping remains; reject unsupported schema version.

**GREEN:** explicit schema and parameterized SQL, foreign keys, transaction per
complete source/domain, unique event identities, separate failed-run records.
Use one small schema version mechanism, not a migration framework. Add mapping
and deletion methods with tests; CLI exposure comes later. A reset is explicit.

**Verify:** real temporary SQLite files, reopen after writes, inspect counts and
foreign-key consistency. Test rollback by querying the database, not by mocking
`commit`. **Maps to:** S01, S03, S04, S07–S09.

## T05 — Configuration, safe HTTP, and bounded failures

**Depends on:** T01. **Own:** `config.py`, `http.py`, synthetic config example,
`tests/unit/test_config.py`, `tests/unit/test_http.py`.

**RED:** missing secret names without values; secret-bearing error body; invalid
provider host; cross-host next link and redirect; 401/403 no retry; 429 Retry-After
in seconds and HTTP-date; long Retry-After; 503 then success; timeout exhaustion;
request budget exhaustion; malformed JSON; missing field; forbidden write route.

**GREEN:** fixed provider bases and method/path allowlists, request limits and
timeouts from ARCHITECTURE. Inject the HTTP client, clock, and wait function.
Use MockTransport; never sleep in tests. Store only redacted error categories.
Read secrets from environment/private OAuth file; no secret flags or .env loader.

**Verify:** capture stdout/stderr/logs and assert sentinel tokens never appear.
Verify attempts, elapsed-wait decisions, and exit-level error categories.
Do not broaden allowed hosts to make a test pass. **Maps to:** S08–S10.

## T06 — Apollo list, memberships, and sent activity

**Depends on:** T02, T04, T05; T00 for field contracts. **Own:**
`adapters/apollo.py`, `tests/integration/test_apollo.py`, synthetic Apollo fixtures.

**RED:** two contact pages; identical IDs across pages; empty list; bad label ID;
website missing; duplicate sequence names; removed enrollment; incomplete
50-event timeline; permission failure; search cap; wrong sender; scheduled email;
actual sent-then-bounced email; email with only completion/delivery time.

**GREEN:** send exact documented methods and parameter locations. Retrieve the
catalog once. Read only necessary details; join email evidence by verified IDs.
Use current-membership fields/export verified in T00; do not reconstruct complete
membership from an arbitrary activity window. Preserve unavailable fields.

**Verify:** request assertions in MockTransport plus returned domain records;
private live comparison on the chosen small list. If the contract is not proved,
report “offline implementation verified; live mapping blocked” and retain the
remaining live-verification line. **Maps to:** S01–S03, S08.

## T07 — Gmail consent and token lifecycle

**Depends on:** T05; owner setup from T00. **Own:** `adapters/gmail_auth.py`,
`tests/unit/test_gmail_auth.py`, private-token setup documentation.

**RED:** no token; valid token; expired token with refresh; revoked refresh token;
consent denial; corrupt token file; scope mismatch; secret in exception; token
file permissions; failed token write leaves previous valid file intact.

**GREEN:** integrate established Google OAuth/auth libraries using desktop
loopback flow, state/PKCE support, offline refresh, and atomic private storage.
Print a consent URL for the user rather than driving their browser. Request only
`gmail.readonly`. Do not introduce a hosted callback service or collect passwords.

**Verify:** mocked auth-boundary tests, scope and permission checks; an explicit
owner-driven live login and refresh. Testing-mode expiry and re-consent are
documented. No credentials in CI. **Maps to:** S08, S09.

## T08 — Gmail sent metadata and combined email coverage

**Depends on:** T02–T05, T07; T00 schema. **Own:** `adapters/gmail.py`,
`tests/integration/test_gmail.py`, combined-email tests in `test_projection.py`.

**RED:** multiple page tokens; unordered search results; same message found for
two aliases; exact versus near-matching recipient; CC/BCC; false-positive query
result; wrong sender; incoming reply; malformed Date header; valid internalDate;
missing/deleted message between list/get; 429; Gmail failure with cached Apollo.

**GREEN:** enumerate SENT message IDs for validated contact addresses, fetch
metadata only, validate actual From and recipient headers, cache by message ID,
and compute qualifying timestamps. Validate/quote query values and keep all
pages; do not assume result ordering or Gmail UI alias expansion. Gmail failures
must leave combined coverage partial, including when Apollo succeeds.

**Verify:** contract tests assert no bodies/snippets are requested and both
provider timestamps are retained. Live verify one direct Gmail email and one
Apollo email. **Maps to:** S02, S07–S09.

## T09 — LinkedIn snapshots

**Depends on:** T02, T04, T05; T00 domain/paging evidence. **Own:**
`adapters/linkedin.py`, `tests/integration/test_linkedin_snapshot.py`.

**RED:** each verified domain fixture; actual no-data/processing distinction;
misleading total count; several pages; repeated page; unexpected next host;
missing scope; expired token; wrong-version error; missing URL; same-day date;
group message; known standard inbox versus unverified Sales Navigator coverage.

**GREEN:** fetch only required domains with the documented version header and
verified traversal rules. Produce normalized facts and source-generation time
only when known. Filter/minimize unrelated data before persistence. Do not label
a successful fetch as real-time or automatically infer that missing data is empty.

**Verify:** all paging requests and full normalized results with MockTransport;
private owner comparison per domain. Unknown scope/freshness remains explicit.
**Maps to:** S04–S09.

## T10 — Explicit archive and CSV import

**Depends on:** T02, T04; T00 verified export headers. **Own:**
`adapters/csv_import.py`, `tests/integration/test_csv_import.py`, synthetic CSVs.

**RED:** BOM; known preamble; reordered columns; quoted multiline message;
missing domain file; unknown header; bad date; date-only precision; duplicate
file digest; older archive; missing Apollo stable IDs; huge input; file path
outside the selected import directory; formula and HTML text; partially invalid rows.

**GREEN:** parse verified header sets with standard csv tools. Accept extracted
CSV directories/files only; no ZIP parser. Hash files, preserve date precision,
report issue counts, and import transactionally. Failed rows mean partial
semantic coverage; do not claim a clean complete import after silent drops.

**Verify:** real temporary files and database; two imports have stable event
counts; older imports cannot regress projections. If real schemas are unavailable,
support only the documented synthetic format and keep provider-format validation
remaining. **Maps to:** S01, S03, S05–S09.

## T11 — Refresh orchestration

**Depends on:** T03, T04, T06, T08, T09; T10 for import integration.
**Own:** `sync.py`, `tests/integration/test_sync.py`.

**RED:** all three sources succeed; Gmail fails; LinkedIn one-domain failure;
Apollo membership incomplete; a second contact page fails; budget exhaustion;
rerun; restart after crash; concurrent refresh; complete empty list; explicit
subset refresh; output errors after successful source commit.

**GREEN:** orchestrate independent source/domain fetches and transactions;
reconcile only committed evidence; preserve prior complete snapshots; record
counts, coverage, and failures; translate result into the SPEC exit contract.
Do not erase good state on a source failure or silently import an old archive.

**Verify:** actual store contents after each failure, not just returned statuses;
one offline all-source fixture scenario. A useful partial refresh returns 3,
not 0. **Maps to:** S01, S07–S09.

## T12 — Private table output

**Depends on:** T03, T04, T11. **Own:** `render.py`,
`tests/unit/test_render.py`, synthetic output used in acceptance tests.

**RED:** all required columns; stable contact and sequence ordering; nested
message JSON; date-only output; unknown versus empty; stale data banner; inferred
acceptance label; HTML script text; javascript URL; CSV formulas including
leading whitespace; Unicode; multiline content; interrupted write; output path
equals an input; explicit message-text omission.

**GREEN:** CSV writer and small HTML renderer with escaped text, validated links,
no JavaScript/external assets, and local styling. Render primary contacts and
unmatched recent network activity separately. Atomic output replacement. Keep
storage values unchanged when neutralizing spreadsheet formulas for export.

**Verify:** read CSV back with csv.DictReader; parse HTML and inspect dangerous
text/links; confirm no network resources. View synthetic output in Firefox with
a temporary profile. No brittle pixel snapshots. **Maps to:** S01–S10.

## T13 — User commands and complete offline workflow

**Depends on:** T07, T10–T12. **Own:** `cli.py`, `__main__.py`,
`tests/acceptance/test_workflow.py`, `tests/acceptance/test_cli_errors.py`.

**RED:** command workflow from temporary config/database through mocked providers
to exported files; offline import/export; review/mapping conflict; forget; reset
without confirmation; no secrets in help/errors; invalid flags; version from
installed package metadata; failure/partial exit statuses.

**GREEN:** wire the commands in ARCHITECTURE. Keep business logic out of argparse
handlers. `sync` produces outputs and a concise coverage report; `export` works
offline. Manual mapping validates identifiers. Forget/reset affect local state
only and describe reimport/revocation limits.

**Verify:** subprocess tests using temporary paths and controlled transports,
then clean-wheel installation and synthetic end-to-end use. Update runnable
README instructions now that commands exist. **Maps to:** S08–S10.

## T14 — Prove and improve LinkedIn freshness

**Depends on:** T00, T09, T11. **Own:** freshness decision in ACCESS; if needed,
changelog functions in `adapters/linkedin.py`, checkpoint storage,
`tests/integration/test_linkedin_changelog.py`.

**Decision first:** compare snapshot coverage/freshness with the owner's actual
on-demand workflow. Close with measured snapshot adequacy only if it satisfies
the agreed behavior; do not manufacture an undocumented refresh endpoint. If
needed and supported, implement changelog ingestion. If neither route meets the
need, retain the precise limitation and owner decision as remaining work.

**RED if implementing:** inclusive cursor boundary; two events sharing processed
time; more than one page; duplicate activity ID; failed event then successful
replay; delete/update; unrelated resource; interrupted page; cursor rollback;
empty page; out-of-order event; history gap; baseline overlap; expired token.

**GREEN:** implement only verified relevant event shapes. Deduplicate replays,
commit data and cursor together, retain failure evidence, and never advance
through incomplete traversal. Gap recovery refetches snapshots but preserves
the history-gap warning. Explicitly classify event versus processing timestamps.

**Verify:** deterministic replay against real temporary SQLite; private known-event
comparison across two reads, without generating outreach. No promise of exact
real-time acceptance. **Maps to:** S05–S08.

## T15 — CI, setup documentation, and live acceptance

**Depends on:** all required task evidence. **Own:** `.github/workflows/ci.yml`,
README usage, account setup notes, sanitized release report, implementation PR.

**CI:** least-privilege `contents: read`; clean checkout; fixed Python version;
locked dependencies; offline tests; Ruff check/format; package build. Choose and
pin reviewed action revisions at implementation time. No provider secrets,
private datasets, or prospect-table artifacts in public CI.

**Acceptance:** use one owner-selected list. Check all SPEC columns against the
sources, direct Gmail plus Apollo sends, two current sequence memberships,
outgoing versus incoming invitations, connection precision, both message
directions, Sales Navigator coverage, one unresolved match, repeat-run stability,
and a simulated provider failure. Include request counts and freshness evidence.
No messages, invitations, or enrollments are sent by these checks.

**Finish:** full offline gates and clean-wheel smoke; document exact live checks,
unknowns, and setup steps; inspect staged files; open a feature PR with concrete
behavior and validation. Do not merge or call live synchronization complete if
required evidence is still blocked. Remove only verified remaining-task lines.

## Regression scenarios that must survive refactoring

| Scenario | Expected evidence |
| --- | --- |
| Two same-name people | No false auto-link; a visible unresolved item |
| Exact URL contradicts exact email | Conflict remains visible |
| New direct Gmail email | Combined latest-known date advances with Gmail provenance |
| Gmail read fails | Previous facts retained; partial/stale coverage |
| Sequence feed reaches 50 events | No claim of complete current membership from the feed |
| Invitation disappears | No invented acceptance |
| Date-only connection | Original date precision survives output |
| Same archive twice | Stable rows/events and no fake new activity |
| Snapshot says total 1 but has more pages | Verified termination rule retrieves all pages |
| Changelog cursor ties/replays | No loss, no duplicates, no cursor past a failed page |
| Malicious imported text | Inert HTML/CSV and no agent/tool execution |
| A source fails during refresh | Its last good state survives; independent success remains usable |

## Reporting and escalation

Use [HANDOFF.adoc](HANDOFF.adoc). Escalate a contract discrepancy with endpoint,
redacted status, expected/observed shape, reproducible synthetic case, and the
smallest proposed spec change. Never silently replace the access strategy or
weaken a test. Ask for owner setup only when the step actually depends on it.
