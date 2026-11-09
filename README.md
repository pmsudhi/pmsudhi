# Sudheesh PM

**I make data infrastructure smaller, faster and measurable.**

CTO at [DataHQ](https://datahq.ai). 26 years in product development, R&D and software architecture,
and I still write the engines myself. Kochi, India.

[LinkedIn](https://www.linkedin.com/in/sudhi/) · [Cameleer](https://cameleer.ing/) · [X](https://x.com/pmsudhi) · pmsudhi@gmail.com

## The pattern

Most data infrastructure was designed for servers with gigabytes to spare and seconds to warm up, or
it assumes your data moves into someone else's platform. Serverless functions, edge devices, AI agents
and enterprises with their own data centres fit neither. My work is replacing the heavy part and
keeping the interface, in three rules:

- **Compatible.** I implement the standard your ecosystem already speaks (Apache Camel routes,
  JDBC, ODBC and ADBC, OCEL 2.x) and read the systems you already run, so adopting it means no
  rewrite and no retraining.
- **Smaller and faster.** Standard-library Rust, one binary, commodity hardware, no runtime to install.
- **Proven.** Every number names its method, its source and its cost. The harness is in the repo.

| Layer | Project | Stays compatible with | Where it stands |
| --- | --- | --- | --- |
| Enterprise data | **DataFabric** (DataHQ) | Your POS, ERP, WMS, TMS, HRMS and banks | In production at DataHQ |
| Integration and agents | **Cameleer** | Apache Camel routes, YAML and XML DSLs, MCP | Public, v0.5, measured; attestation in the next release |
| Storage | **HelixDB** (upstream contributions) | Existing databases open unchanged | 2 PRs merged |
| Process mining | **pm-engine** | OCEL 2.x, pm4py parity target | Private, phase gates measured |
| Connectivity | **karez** | JDBC, ODBC, ADBC | Private, design stage |
| Web | **simoom** | HTTP/1.1, HTTP/2, HTTP/3 | Private, under construction |
| Personal data | **KnowDB** | Email, calendar, contacts, local-first | Private, pre-build |

## At DataHQ: enterprise data on hardware you already own

[DataHQ](https://datahq.ai) is built on the same thinking. Enterprise data usually has to be loaded
into data-warehouse software before anyone can use it. **DataFabric** keeps it inside the enterprise,
on-premise on commodity hardware, and builds the working model on top of the systems already there.

- **One model, built from events.** An event-sourced ledger turns every sale, purchase order, stock
  move and payroll run into money in the same model. Process discovery (procure-to-pay, order-to-cash)
  is read from the event stream, not drawn by hand.
- **Fabric, Spyke, Pilot.** Fabric builds the graph, Spyke turns it into reports, dashboards, alerts
  and the board MIS, and Pilot simulates plans, answers questions in plain English and writes approved
  plans back, with an audit trail on every number.
- **For physical businesses.** F&B, retail, manufacturing and logistics: outlets, stores, plants and
  depots on one ledger that people and AI agents both read.
- **Grounded AI.** Agents read one defined, deterministic model, so the same question gets the same
  answer, with the source row that proves it.

## Proof

| | |
| --- | --- |
| **83.7% smaller** | graph databases in HelixDB after my zstd compression PR (merged) |
| **2.5x faster** | HelixDB edge writes after my flush fix (merged) |
| **About 440x** | faster startup than Apache Camel on the JVM (2.5 ms against 1,106 ms) |
| **About 50x** | less memory for the same route (8 MB against 399 MB) |
| **1.7x to 8.7x** | Camel's throughput in all 11 scenarios measured |
| **41.6k tool calls/s** | MCP server throughput, 1.85x Apache Camel's, with every call checked, run once and audited |
| **196 of 200** | field-test runs correct with Claude Opus 5.5 on 40 real requests, none acting twice |
| **0 dependencies** | in Cameleer's 230 components, with 2,800+ tests |

## Merged upstream: HelixDB

[HelixDB](https://github.com/HelixDB/helix-db) is a Rust graph and vector database with 6.3k stars.
Each PR carries a reproducible harness and states its costs next to its gains.

| PR | What changed | Result | How it was measured |
| --- | --- | --- | --- |
| [#1255](https://github.com/HelixDB/helix-db/pull/1255) | zstd compression of SST and WAL blocks | Graph databases **83.7% smaller**, vector databases **46.5% smaller** on disk (S3: 82.7% and 37.6%). Writes cost at most 3.7% more. A cold vector read on local disk is 19.5% slower. | Median of 6 runs on two 4-vCPU droplets |
| [#1220](https://github.com/HelixDB/helix-db/pull/1220) | Stop flushing deferred index state before element reads | Edge writes **2.5x faster** (2,811 to 7,098 per second). A real 21k-event log imports in 52 s instead of 124 s. | One run each, Windows, release build |

[#1216](https://github.com/HelixDB/helix-db/pull/1216) found why writes slowed as a label grew: every
write re-read the whole label bitmap. The maintainers shipped a generalised fix as
[#1239](https://github.com/HelixDB/helix-db/pull/1239).

## Cameleer: Apache Camel without the JVM, and the trusted action layer for AI agents

[Cameleer](https://github.com/pmsudhi/cameleer) ([site](https://cameleer.ing/), Apache-2.0) runs your
Apache Camel routes, patterns and endpoint URIs unchanged as one native Rust binary, with no
dependencies. Serve the same routes as MCP tools and it becomes the layer between an AI agent and your
systems: **the model decides what, a tested route decides how, every call is checked, run once,
approved and audited, and inside a confidential VM a client can get hardware evidence of exactly which
routes and tools it is talking to.**

### Faster and far smaller

Cameleer 0.4.0 against Apache Camel 4.23 on Java 21 and Camel compiled natively with Quarkus: the same
routes, identical cloud servers, mean of two runs.

| | Cameleer | Camel (JVM) | Camel (Quarkus native) |
| --- | ---: | ---: | ---: |
| Startup to running routes | **2.5 ms** | 1,106 ms | 30 ms |
| Peak memory, direct call | **7 MB** | 399 MB | 113 MB |
| REST API with JSON | **80.3k req/s** | 37.9k | 17.9k |
| HTTP into PostgreSQL | **11.0k req/s** | 4.8k | 3.5k |
| Kafka stream processing | **6.1k records/s** | 698 | 694 |
| Content-based router | **1.14M msg/s** | 375k | 119k |
| In-memory request-reply | **719k msg/s** | 117k | 72k |

Cameleer leads on throughput in all 11 scenarios and on p99 latency in 8 of 9. The ninth, HTTP into
Kafka, is slightly worse (8.6 ms against 7.8 ms) while serving 1.7x the requests. Method, raw data and
every scenario: [docs/BENCHMARKS.md](https://github.com/pmsudhi/cameleer/blob/main/docs/BENCHMARKS.md).
Rerun it with `scripts/cloud-bench` for about a dollar of cloud time.

### The action layer: routes as MCP tools

A tool is a route, and `cameleer run tools.camel.yaml --mcp` serves it to Claude, IDE assistants and
agent frameworks over stdio or Streamable HTTP. The model decides what to call; the server decides
whether and how it runs. Every call is:

- **Checked** against a JSON Schema, with unknown arguments dropped.
- **Run once.** A retried call in the same conversation returns the first result, so a card is not
  refunded twice.
- **Approved** when risky, by the user in the client or by an operator outside the chat.
- **Bounded** by rate limits, circuit breakers and a timeout that cancels the route.
- **Recorded** with personal data redacted, in a hash-chained audit log.
- **Pinned.** A tool definition cannot change under a client, and a kill switch withdraws a tool.

- **Both MCP protocol versions on one endpoint:** 2026-07-28 stateless and 2025-11-25, 2025-06-18 and
  2025-03-26 with sessions, over Streamable HTTP and stdio, with API-key, JWT and OAuth token checks.
- **A gateway too:** it imports other MCP servers' tools, with protection against a tool changing under
  you, and serves them again under the same rules, so one endpoint fronts many.
- **An existing route becomes a tool by changing one line.** Next release: `--mcp-expose` serves REST
  operations as typed tools with no edits, and writes need approval by default.

Measured 2026-11-06 on three DigitalOcean 4-vCPU droplets, 64 connections, the two servers swapping
roles, raw records in the repo:

| MCP `tools/call` in a session | Cameleer | Apache Camel MCP server 4.23.0 | MCP TypeScript SDK 1.32.2 |
| --- | ---: | ---: | ---: |
| Calls per second | **41,620** | 22,458 | 1,536 |
| p99 latency | 5.9 ms | **5.2 ms** | 79 ms |
| Peak memory | **53 MB** | 317 MB | 214 MB |

Cameleer also serves 51,700 stateless calls per second (2026-07-28) in 26 MB; the other two do not
offer that mode. Camel's server has the lower p99, and every Cameleer call in the run went through the
whole checking envelope, which costs about 6 microseconds. It passes the official MCP conformance
suite: 39 of 39 for 2025-11-25 and 26 of 26 for 2025-06-18. That suite predates 2026-07-28, so that
version is covered by Cameleer's own tests.

### The trust layer: evidence of what is running

An agent that calls a tool cannot normally tell which server, routes and tools are behind it. Inside a
confidential VM (Intel TDX, AMD SEV-SNP, or a dstack CVM such as Phala Cloud), Cameleer returns a
hardware-signed report that covers:

- the **client's own nonce**, so the evidence is fresh;
- the **server's TLS key**, so it belongs to this connection (a TLS-terminating proxy breaks the check);
- a **manifest** of the exact release, routes (SHA-256), configuration with secrets redacted, and tool
  fingerprints;
- a **hash-chained log** of every change since start-up, such as a route added or a tool withdrawn.

A client gets it over HTTP at `/q/attestation`, in the MCP handshake, or through an opt-in tool. If
attestation is on and no evidence can be produced, start-up fails, so a process cannot look attested
when it is not.

Status: on `main` and in the next release (the current release is 0.5.1). The tests use a simulated
TEE agent and it has not yet run on real TEE hardware. Cameleer does not verify Intel or AMD signature
chains itself; a client uses the platform's verifier. There is no gateway policy for remote servers,
no sealed secrets yet, and no protection against physical attacks on the host. The manifest means
"this exact release" when the image is pinned by digest. Details in
[docs/guides/attestation.md](https://github.com/pmsudhi/cameleer/blob/main/docs/guides/attestation.md).

## Also building

- **pm-engine** (private). Process mining for object-centric event logs too large for memory. It began
  on HelixDB; the measurements led to the upstream PRs above and then to a ground-up rewrite when the
  numbers said so. Native OCEL 2.x, pm4py parity as the target, and Rust, Python, TypeScript and Go
  SDKs and an MCP surface planned. I also wrote the OCEL-Graph RFC, an extension proposal to the OCEL
  standard's maintainers. Every phase ends in a measured gate.
- **karez** (private, design stage). Database connectivity for Rust: one API, native PostgreSQL,
  MySQL, SQL Server and SQLite drivers, Arrow-native results, exported as JDBC, ODBC and ADBC drivers.
  Nothing is measured yet, and the targets are labelled as targets.
- **simoom** (private). A standard-library-only HTTP/1.1, HTTP/2 and HTTP/3 engine for edge devices,
  functions and servers. It is Cameleer's HTTP engine.
- **KnowDB** (private, [releases](https://github.com/pmsudhi/PKC)). A local-first assistant over email,
  calendar and contacts that never sends your data anywhere you did not ask.

## 26 years, one thread

The same three rules run through every role. My belief since the first job: what matters has to be
measured, analysed and monitored.

| Years | Role | What I did | The thread |
| --- | --- | --- | --- |
| 2000 to 2006 | Programmer, then lead programmer of 12, Beehive Systems (WASP3D) | Built the workflow module and services of a live broadcast product. Code review, profiling and optimisation, and time-critical live projects with decisions made on the spot. | **Measured** under live deadlines |
| 2006 to 2009 | .NET product architect | Integrated the product with third-party broadcast software and hardware. Designed the protocol and API that let OEMs and service providers build on it. Owned performance and scalability. | **Compatible**: a standard others build on |
| 2009 to 2020 | Head of product delivery and third-party integration | Planned and launched a SaaS business unit that raised ARPU and grew the client base. Set up a development centre and DevOps. Led 35 developers and a 10-person test team. Delivered time-limited broadcast projects such as World Cup and election coverage. | **Proven** at scale, to a date that cannot move |
| 2021 to now | CTO, DataHQ | DataFabric, Fabric, Spyke and Pilot on one event-sourced model, plus the open-source engines on this page. | All three, with the code written myself |

## How I work

Every number carries its source (measured here, vendor, third party) and the date it was checked.
Options are weighed in a table with a verdict. A build starts after a go/no-go measurement, and
benchmarks are written so a maintainer can rerun them.

## Talk to me about

Moving Camel routes off the JVM, making AI agent actions safe to run and provable, keeping enterprise
data on your own hardware, and Rust for data infrastructure. pmsudhi@gmail.com or
[LinkedIn](https://www.linkedin.com/in/sudhi/).
