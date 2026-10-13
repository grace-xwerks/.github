# Grace Engineering

A precision CNC shop in Memphis, Michigan, that writes its own software where
the off-the-shelf option doesn't fit the work. This org is that software: the
CAM tree the machines actually run from, the plumbing between CAM, ERP and the
shop floor, and a handful of products built for customers.

Four repos carry almost all of the activity. The status column below tells you
which, and the rest is here so you can tell a parked experiment from a working
tool without opening it.

## How it fits together

Programs start in CAM: Fusion for milling and mill-turn, PartMaker/SwissCAM
for the Swiss lathes. The posted G-code lands in grace-cam, the shop's source
of truth. It keeps one folder per part under
`parts/{turning,milling,swiss}/{machine}/{customer}/`, holding the program, a
`recipe.json`, the setup sheet and notes, beside the per-machine post
processors and the validators that run before anything touches iron. A proven
program enters the tree copied byte for byte from the CNC backup, named for
the O-number it declares and hashed, and only then gets a recipe.

tapeworm takes it from there. It holds the versioned program library in
Supabase, assigns each program its machine-specific O-number, resolves
subprogram dependencies, and streams the result to the control over RS-232, a
network share or a USB gadget. On the other end are Tsugami M08SY mill-turns,
Citizen L20 and K16 and Star SR-32 Swiss lathes, a Mazak Integrex 100-IV ST,
and Brother Speedio mills. Machine state comes back out through the ladder99
fork: MTConnect adapters feeding Grafana.

The ERP has its own lane, and nothing in the program path talks to it. Fusion's
tool library syncs nightly through datum into Supabase, with only the identity
slice pushed on to Plex. Plex itself is reached through plex_mcp, an MCP server
over its REST surface that is read-only unless you say otherwise. toollib
indexes every tool call in every program on the backup share into SQLite, and
catalog-chunking turns vendor catalog PDFs into prompt packs for exact
part-number lookup.

## Repos

Status reads: **live** runs the shop today · **active** has real commits this
quarter · **parked** has had none since the date shown · **import** is a working
copy brought in on 2026-07-17 and not developed here since · **fork** tracks an
upstream.

### Runs the shop

| Repo | What it is | Status |
|---|---|---|
| [grace-cam](https://github.com/grace-xwerks/grace-cam) | The CAM source of truth. `parts/{turning,milling,swiss}/{machine}/{customer}/{part}/`, each folder carrying the program, `recipe.json`, setup sheet and notes. Fusion posts forked per machine with a `CHANGELOG.md` each, Grace-standard subprograms, and the validators that run before a program touches iron. Start with its "New here?" section. | live · 2026-10 |
| [tapeworm](https://github.com/grace-xwerks/tapeworm) | Supabase-backed G-code library and transfer platform. Versioned programs with approval states and SHA-256, per-machine O-number namespaces, a subprogram dependency graph, and a transfer agent that streams over RS-232, network share or USB gadget. pnpm monorepo: React cockpit, agent, Rust serial core, MCP server. MIT. | active · 2026-10 |
| [post-development](https://github.com/grace-xwerks/post-development) | Staging for Fusion posts under development (Mazak Integrex i-100 and i-100S, Tsugami M08SY) before they graduate into grace-cam. | staging |
| [grace-integrex-post](https://github.com/grace-xwerks/grace-integrex-post) | Standalone Fusion post for the Mazak Integrex 100-IV ST on Matrix control. Public as a reference for other owners of the same machine; the working copy now lives under grace-cam's `posts/`. Proprietary license, v0.5.0. | frozen · 2026-01 |

### Tooling data and ERP

| Repo | What it is | Status |
|---|---|---|
| [plex_mcp](https://github.com/grace-xwerks/plex_mcp) | MCP server for Plex, Rockwell's ERP/MES. A catalog of the whole known REST surface as a Bruno collection, search and describe tools over it, and a generic invoke that is GET-only unless `PLEX_ALLOW_WRITES` is set and the call passes `confirm=True`. Also home of the raw-material part-code spec and its encoder/decoder. | active · 2026-10 |
| [datum](https://github.com/grace-xwerks/datum) | Nightly Fusion 360 tool library sync. Fusion JSON is the source of truth, Supabase holds the enriched record, Plex gets only the identity slice its schema accepts. Pre-sync validation gate, 215 tests, CI on PRs. Public. | parked · 2026-05 |
| [partmaker-toolkit](https://github.com/grace-xwerks/partmaker-toolkit) | Companion tooling for PartMaker/SwissCAM, the shop's Swiss CAM standard. First target is tool-library sync through PartMaker's own Master Tools Database CSV path; STEP alignment and profile extraction after. Local MCP for tool-crib queries, sign-out data parser. | planning · 2026-08 |
| [toollib](https://github.com/grace-xwerks/toollib) | Crawls the CNC backup share, extracts every tool call and its drawer comment into SQLite, serves it through a read-only FastAPI and a React SPA. | import |
| [catalog-chunking](https://github.com/grace-xwerks/catalog-chunking) | Vendor tooling catalogs, PDF in, token-lean prompt packs out, for exact part-number lookup. One engine; each catalog is a config file, not new code. | 2026-07 |
| [plexchat](https://github.com/grace-xwerks/plexchat) | Q&A chatbot for Plex administration, answering from the vendor's online manuals. | import |
| [swiss-fusion](https://github.com/grace-xwerks/swiss-fusion) | Citizen L20 Fusion post and the Swiss turning research notes behind it. | import |
| [plex-code-mcp](https://github.com/grace-xwerks/plex-code-mcp) | Earlier standalone MCP for building and decoding raw-material part codes. That module now lives in plex_mcp. | superseded |
| [faasst](https://github.com/grace-xwerks/faasst) | Design for a speed-and-feed API: manufacturer catalog data to cutting parameters on ISO 13399 geometry and ISO 513 material groups. README only. | parked · 2026-04 |
| [whizcut](https://github.com/grace-xwerks/whizcut) | WhizCut vendor catalog and WhizHold PDFs. Reference, no code. | reference |

### Machine monitoring and shop infrastructure

| Repo | What it is | Status |
|---|---|---|
| [ladder99](https://github.com/grace-xwerks/ladder99) | Fork of Ladder99: MTConnect adapters, agent, Grafana. Grace's changes: a Brother Speedio adapter and schema, Grafana moved from 8 to 12 with AngularJS removed, a Grace setup running on the Mazak demo agents, and the docker pipeline building again. Apache-2.0. | fork · 2026-09 |
| [hid-barcode-scanner](https://github.com/grace-xwerks/hid-barcode-scanner) | Fork of an Android app that scans barcodes and types them into a PC as a Bluetooth keyboard. Shop-floor scanning without a tethered gun. GPL-3.0. | fork · 2026-10 |
| [dev-station](https://github.com/grace-xwerks/dev-station) | Inventory of the remote dev machines: SSH targets, bootstrap scripts, IAP tunnel helper, rebuild history. | 2026-09 |
| [net_explore](https://github.com/grace-xwerks/net_explore) | Shop network notes: the SG350 switch design and router troubleshooting. | notes |

### Internal apps

| Repo | What it is | Status |
|---|---|---|
| [grace](https://github.com/grace-xwerks/grace) | GRACE, the internal technical agent at ai.graceops.dev. RAG chat over a Supabase knowledge base for staff, RFQ qualification, admin panel. React on Cloudflare Workers, Claude behind it. | deployed |
| [levis-superapp](https://github.com/grace-xwerks/levis-superapp) | Levi, customer job-status updates at levi.graceops.dev. One link per job, no account, optional push notifications; the shop posts updates from a magic-link admin. Built for Prevail Precision. | deployed · 2026-04 |
| [use-chat](https://github.com/grace-xwerks/use-chat) | Chrome extension that spots internal recipients in a Gmail compose window and nudges the sender toward Google Chat. Public. | 2026-05 |

### Field products

| Repo | What it is | Status |
|---|---|---|
| [stratum](https://github.com/grace-xwerks/stratum) | Target-side aim-tracking instrument for archery, built for Prime Archery. Laser on the bow, a Nicla Vision sensor at the target face reporting centroid plus IMU over BLE, a React Native app drawing the point cloud with grouping stats and CSV export. Gen 1 is three iPad field-demo units running two laser-aim games; Android is not a Gen 1 target. | Gen 1 build · 2026-08 |
| [pointCloudThing](https://github.com/grace-xwerks/pointCloudThing) | Stratum's predecessor, from when the scope still included rifle work. Its design space moved into stratum's `docs/vision/`. | archived |

### Forward R&D

The bet on what CNC programming looks like next: a node-graph dataflow with
structured JSON as the AI bridge, and everything downstream of it, G-code,
quoting, scheduling, a different consumer of the same validated job JSON. Both
repos hit their milestones in spring 2026 and have been quiet since while the
shop-floor work above took priority.

| Repo | What it is | Status |
|---|---|---|
| [toolpath-logic-kernel](https://github.com/grace-xwerks/toolpath-logic-kernel) | Node-graph CAM. A Python node canvas exports JSON, C# ingests it into a timeline, an F# state machine validates two-channel sync, and a post emits Citizen L20 two-channel G-code. An LLM can author the JSON against a schema and never touches spatial math. All four milestones marked complete. | parked · 2026-06 |
| [tlk-quoting-engine](https://github.com/grace-xwerks/tlk-quoting-engine) | Quoting as another consumer of TLK's validated job JSON: part description, AI translation, cycle time, material cost, price breaks. C# / .NET 10 minimal API with a Gemini bridge. Phases 0 to 4 done; the HTML frontend is a placeholder. | parked · 2026-05 |

### Experiments and scratch

| Repo | What it is | Status |
|---|---|---|
| [fusion_mcp_testing](https://github.com/grace-xwerks/fusion_mcp_testing) | A 104-test suite for an Autodesk Fusion 360 MCP server. Public. | 2026-05 |
| [fusion_manage_lab](https://github.com/grace-xwerks/fusion_manage_lab) | Autodesk Fusion Manage auth and API probes. | import |
| [tapewormPlus](https://github.com/grace-xwerks/tapewormPlus) | Scaffold for a tapeworm successor layout. The work is in tapeworm. | import |
| [demo-repository](https://github.com/grace-xwerks/demo-repository) | Sandbox for trying Actions and org integrations without touching a real project. | sandbox |
| [workspace](https://github.com/grace-xwerks/workspace) | This org as a repo: a declared manifest of every repo, a sync tool, and the shared Claude Code skills. See Start here. | active · 2026-10 |
| [.github](https://github.com/grace-xwerks/.github) | This page, plus the org-wide `.gitattributes` every repo copies. | |

## Stack at a glance

- **Python** for validators, parsers, sync jobs and every MCP server.
- **TypeScript** pnpm monorepos for the apps, Vite + React on the front, React Native for stratum.
- **Rust** in exactly one place: tapeworm-core, the serial transfer, where bytes and timing matter.
- **Fusion posts** are JavaScript in `.cps` files, forked from Autodesk's and diffed per machine.
- **C# and F#** only in the TLK pair.
- **Supabase** (Postgres) under tapeworm, datum and grace. Cloudflare Workers for the hosted apps. Shop-floor services run on-prem.
- **MCP** is the house pattern for putting an AI client in front of a system: plex_mcp, tapeworm's server, partmaker-toolkit.

## Conventions

- **Line endings.** LF everywhere, enforced by the `.gitattributes` in this repo and copied into every repo. Machine-facing files (`.nc`, `.eia`, `.cps`, `.csv` and friends) are marked `-text`: git never rewrites a byte of a proven program.
- **Proven programs are evidence.** Copied byte for byte from the CNC backup, named for the O-number they declare, with the backup path, date and SHA-256 recorded next to them.
- **No silent post changes.** Every edit to a post processor gets a line in that post's `CHANGELOG.md` in the same commit.
- **Writes to the ERP are opt-in twice.** An environment flag and an explicit confirm on the call. Reads are free.
- **Branch, PR, squash-merge.** Most work happens in Claude Code sessions and git worktrees; the workspace repo's `/overlord` skill keeps session, branch, worktree and PR lined up.

## Start here

```sh
gh repo clone grace-xwerks/workspace grace-xwerks
cd grace-xwerks
node sync.mjs
```

That clones every repo in the manifest beside it and reports branch, tree
state and position against origin for each. Needs `git`, `node` 20+, and an
authenticated `gh` with org membership for the private repos. Then read
grace-cam's README from the top, and tapeworm's after that.

## The thesis, briefly

Most shops buy software. Grace writes it where the market option doesn't fit
the work or where the workflow itself is the edge, and every repo above is paid
for in shop-floor leverage: faster setups, fewer crashes, less duplication
between CAM and ERP. The longer bet, carried by the TLK pair, is that the right
primitive for AI in manufacturing is validated structured JSON rather than
natural language straight to G-code.

[graceengineering.com](https://graceengineering.com)
