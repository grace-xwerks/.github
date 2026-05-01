# Grace Engineering

Precision CNC manufacturing — and the software that runs it, plus the experiments betting on what comes next.

This org houses three threads of work: the production tooling that operates the shop fleet in the near future, the R&D track reimagining CAM around AI-generated node graphs, and customer-facing products built on Grace's hardware-software fluency.

---

## 🏭 Production manufacturing

Software that runs the shop.

**[grace-cam](https://github.com/grace-xwerks/grace-cam)** — The shop floor's source of truth. Parts library organized by `{turning,milling,swiss}/{machine}/{customer}/{part}/`, Fusion post processors per machine, Grace-standard subprograms, and validators that catch crashes before they hit the spindle.

**[grace-integrex-post](https://github.com/grace-xwerks/grace-integrex-post)** — Custom Autodesk Fusion post processor for Grace's Mazak Integrex 100-IV ST dual-turret mill-turn (serial 195369). G109 turret selection, Matrix M-codes, drilling/tapping/threading cycles. *Public so other Integrex 100-IV ST owners can borrow from it.*

**[tapeworm](https://github.com/grace-xwerks/tapeworm)** — Move G-code between VS Code and a CNC over RS-232. Drip-feed, send, receive — Haas, Fanuc, Mazak, Mitsubishi. Rust serial core and TypeScript MCP server are built; the VS Code extension wrapping them is the next layer.

**[datum](https://github.com/grace-xwerks/datum)** — Nightly Fusion 360 → Supabase → Plex tooling sync. Fusion JSON is the source of truth; Supabase holds the enriched record (geometry, holders, pocket assignments); Plex gets the identity slice it can accept. *Public; lifts from a public reference base.*

## 🧪 Forward-looking R&D

The bet on what CNC programming looks like next: a node-graph dataflow with structured JSON as the AI bridge — and everything downstream (G-code, quoting, scheduling) becoming a different consumer of the same validated job JSON.

**[toolpath-logic-kernel](https://github.com/grace-xwerks/toolpath-logic-kernel)** — Open-source node-based CAM interface and kinematic math engine. LabVIEW-style visual dataflow replaces 3D CAD; LLMs author against a strict JSON schema and never touch spatial math. Python frontend, C# ingestion, F# state machine, generates crash-free G-code for Citizen L20 today. *Public.*

**[tlk-quoting-engine](https://github.com/grace-xwerks/tlk-quoting-engine)** — Automated CNC quoting on TLK output. Customer description → AI translation → cycle-time + material-cost + price-break math. Quoting is just a different consumer of the same validated job JSON. .NET 10 minimal API + Gemini AI bridge. Phases 0–4 shipped; HTML frontend pending.

## 🎯 Customer products

**[stratum](https://github.com/grace-xwerks/stratum)** — *Every shot, mapped.* Real-time laser dot tracking and grouping analysis for archery and rifle precision shooting. Live point-cloud overlay, session recording, CSV export, ring-tap calibration for Vegas / NFAA / outdoor / 3D / benchrest faces. Built for **Prime Archery** and **Montana Rifle Company**. React Native; Android-first.

---

## The thesis

Most CNC shops buy software. Grace builds it — selectively, where the market option doesn't fit the work or where the workflow itself is the differentiator. Every repo here is paid back in either shop-floor leverage (faster setup, fewer crashes, less duplication between CAM and ERP) or a customer relationship. The R&D track is a longer bet: that the right primitive for AI in manufacturing is structured JSON, not natural language straight to G-code, and that whoever ships the CAM layer for that future has access to a much broader workflow than just generating toolpaths.

[graceengineering.com](https://graceengineering.com)
