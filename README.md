# Notice

CHANGES FOR TESTING PURPOSES

Notice is a general-purpose contract lifecycle management workspace with a human-accountable workflow for teams moving from Word documents, email threads, shared drives, and approval chasing into one governed process.

It covers intake, drafting, review, counterparty exchange, approval, signature, and post-signature work without requiring every business user to become a CLM administrator.

## Start here

| I want to… | Start with… |
| --- | --- |
| Browse all product and implementation documentation | [Documentation index](./docs/README.md) |
| Learn the product from beginning to end | [HTML how-to guide](./docs/how-to/index.html) or the [Markdown user guide](./docs/USER_GUIDE.md) |
| Try Notice safely on my computer | [Local evaluation](#try-notice-locally) |
| Report a bug or contribute | [Contributing guide](./CONTRIBUTING.md) and [governance](./GOVERNANCE.md) |
| Get help or report a vulnerability | [Support](./SUPPORT.md) and [security policy](./SECURITY.md) |
| Configure or verify DocuSeal | [DocuSeal integration profile](./docs/DOCUSEAL.md) |
| Host an approved OCR model privately | [Customer-hosted OCR and bring-your-own model](./docs/CUSTOMER_HOSTED_OCR.md) |
| Integrate a service principal or approved agent | [Governed agent API](./docs/AGENT_API.md) |
| Pilot it at a private-wealth firm | [Private-wealth starter profile](./docs/WEALTH_MANAGEMENT_PROFILE.md) |
| Prepare a staging implementation | [15-minute staging deployment](./docs/DEPLOYMENT_QUICKSTART.md) |
| Understand what the customer must supply | [Implementation handoff](./docs/IMPLEMENTATION_HANDOFF.md) |
| Review production and security requirements | [Production setup](./docs/PRODUCTION_SETUP.md), [Security](./SECURITY.md), and [Threat model](./docs/THREAT_MODEL.md) |

The HTML guide is self-contained. After cloning the repository, open `docs/how-to/index.html` in a browser; it does not need a documentation server or external web assets.

> [!IMPORTANT]
> Notice is an off-the-shelf product foundation, not an automatically approved production service. The reusable CLM workflow is implemented. A deployer still installs it in the customer's environment, and the customer still approves its people, paper, policy, providers, records rules, and go-live evidence.

## What Notice does

- Starts an agreement from a published intake form, an exact published governed-template version, a clearly labeled starter template, or an imported PDF, DOCX, RTF, or text file.
- Governs legal entities, counterparties, groups, custom fields, templates, deterministic workflows, approval authority, intake forms, agreement relationships, and reusable saved views inside each workspace.
- Keeps immutable working-paper versions, source hashes, approved clauses, comments, and attributable history together.
- Runs one controlled counterparty round at a time using an expiring portal link, an approved manual process, or configured signed mail gateways. Portal transfers have durable per-round upload/download budgets that survive link rotation, and only one upload/scan workload can hold the round lease at a time.
- Structurally inspects and scans every returned file. Clean, policy-eligible files can proceed to human review; an unavailable or inconclusive scan stays quarantined, invalid or malicious content is rejected, and a fresh correlated malicious rescan cannot be displaced by a concurrent clean result.
- Optionally turns a clean PDF or DOCX into the first immutable text working version through one deployment-selected customer-hosted OCR model/profile. Bytes still cross only the signed scanner gateway. An uncorrelated or malformed clean result stays quarantined and unreadable; an exact-request malicious verdict cannot be downgraded by malformed ancillary fields; and there is no provider fallback.
- Routes the exact current paper for approval and prevents stale approvals from authorizing a later version.
- Starts every new or materially changed agreement as `Unassessed` and requires an attributable, exact-paper risk decision before approval or signature.
- Creates governed DocuSeal signature packets without exposing signer bearer links in Notice. The production profile is pinned to DocuSeal 3.2.1 and API contract 1.0.24; an optional self-hosted MCP live test validates the pinned tool inventory and invokes only read-only `load_template` against the same configured template. Mutation tools remain outside the agent and signing boundary.
- Provides tenant-scoped portfolio reporting and insertion-fenced live work queues for approvals, signatures, obligations, schedules, and events; refresh after queue mutations because existing rows can move between pages.
- Supports governed clause playbooks, bounded ingestion tracking, and extractor-registered repository fragments pinned to immutable document version/hash plus a caller-asserted source-text digest. Reviewers must compare citations with the protected paper, and accepted proposals never silently apply AI-produced changes.
- Supports scoped service principals, expiring human-sponsored grants, machine-readable action discovery, and durable agent plans. A sponsor cannot approve or reject their own plan, the reviewer must still be an active Admin when the decision commits, and executable reads recheck exact principal/grant/sponsor/time/resource/paper authority both when claimed and when completed. Version 1.1 execution is limited to agreement reads and human-approved, server-derived required-approval routing; approval decisions, risk waivers, signed-evidence activation, membership, and integration decisions remain human-controlled.
- Tracks one-time and recurring renewal, notice, reporting, and other post-signature obligations as governed schedules and materialized occurrences, with eligible owners, local-calendar reminders, and immutable completion evidence.
- Enforces verified identity, workspace membership, role permissions, tenant-scoped data access, retention controls, and signed exports.
- Sends retry-safe events through independent mail and CRM delivery paths. A deployable CRM gateway with HubSpot, Salesforce, and signed custom-webhook adapters is included.

Every agreement follows the same lifecycle:

```text
Intake -> Draft -> Review -> Negotiation -> [risk + approval gate] -> Signature -> Active
```

Risk assessment and approvals are exact-paper gates into Signature, not separate persisted stages. Notice records evidence and enforces workflow boundaries. It does **not** decide whether language is legally or commercially acceptable.

## A contract from start to finish

### 1. Create or import

Start with a published business intake form, governed template, starter template, or supplier/customer/company paper. Intake conversion binds the exact form, template version, workflow, and reviewed business fields before it atomically creates a governed agreement. The browser records the signed-in Legal/Admin reviewer as initial owner and leaves structured entity/counterparty references for deliberate maintenance in Contract details. A direct user can also upload a file up to 20 MiB, review the proposed metadata, and correct it before creation.

In production, plain text and RTF can be processed within the Worker. PDF and DOCX receive bounded structural inspection there—PDF header/trailer/xref and referenced-object checks, and DOCX ZIP/container plus required-OOXML-part checks—but are never rendered, text-extracted, or interpreted in the application Worker. A deployment can select one bounded customer-hosted OCR model/profile behind the existing signed scanner gateway; successful output becomes an immutable, source-hash-linked working version that still requires exact-paper review. If OCR is explicitly disabled, an authorized user supplies reviewed working text instead. See [Customer-hosted OCR](./docs/CUSTOMER_HOSTED_OCR.md).

### 2. Review the exact working paper

Notice loads the latest immutable version before enabling edits or exports. Saving creates a new version with a SHA-256 hash and audit event. If another save wins first, the user must reload instead of silently overwriting it.

Legal and admin users can insert approved clauses into the current editor buffer. They become durable only after the user saves a new version; other roles can read the library when permitted. Governed playbooks, extractor-registered fragments, and cited extraction proposals provide review evidence that must be compared with the protected paper; an accepted proposal remains blocked until an authorized governed executor applies it.

### 3. Exchange redlines safely

An authorized user prepares a hash-bound round from the exact current paper and records the expected counterparty. The normal first-pilot path is:

1. issue an expiring portal link;
2. send it through the customer's approved contracting mailbox and record the bounded message reference;
3. let the counterparty download the outbound file and return one revised file through the portal;
4. wait for Notice to inspect, hash, and scan the return;
5. review the exact clean DOCX in Microsoft Word or another approved environment;
6. provide the complete reviewed text for that one returned attachment and record a decision note; and
7. promote it explicitly or reject/close the round.

Optional automated outbound delivery and inbound email capture require separately configured and accepted signed gateways. Dedicated reply aliases are generated only when inbound capture is enabled and attested.

The portal bearer remains reusable for safe retries during its active round, but its transfer budget belongs to the round and does not reset when staff rotate the link. One round admits at most 10 uploads: no more than 4 accepted returns, no more than 6 rejected or failed attempts, and no more than 96 MiB of cumulative multipart request bytes. Upload work is serialized through one five-minute lease. Downloads are capped at 20 attempts and 256 MiB cumulatively. A busy upload returns `429` with `Retry-After: 5`; an exhausted budget requires staff to close or resolve the round and prepare a new one. Notice never exposes the live counters to the portal holder.

While automated delivery is queued or processing, Notice blocks portal-link rotation, round closure, and promotion. Gateway acceptance confirms only the gateway boundary; it does not prove that the counterparty received the message.

Notice does not parse Word Track Changes into production clause decisions, accept changes automatically, or make a playbook verdict. Promotion creates a new immutable working version, revokes the portal link, invalidates stale approval/signature readiness, and requires the new paper to be approved again.

### 4. Approve and sign

The shipped starter policy requires the `Legal` approval lane for every routed agreement and adds `Finance` for High-risk work, a recorded risk-assessment waiver, or a stored USD value above 100,000 whole dollars. A published governed workflow can instead snapshot a deterministic, bounded set of conditional lanes when an agreement is created. These capitalized lanes are workflow labels, not the lowercase workspace roles.

Approval authority can be assigned directly to an active user or through an active workspace group, per lane, currency, and optional minor-unit amount ceiling. A narrow role-based compatibility default remains only until the first active or revoked authority record is configured for that lane. Every current approval request must pass, including any additional current lane; each required lane needs a distinct attributable decider, and nobody—including legal or admin—can decide a request they created.

When the current paper, risk decision, and required approvals match, Notice creates a DocuSeal packet using the configured template's exact ordered signer roles. DocuSeal delivers signer email; production Notice never returns public signer URLs. Provider completion stores one scanned final artifact and moves the packet to `completed_review_required`; an authorized human must confirm the exact approved basis and retained artifact hash before the record can become Active.

### 5. Operate the signed agreement

Create one-time, weekly, monthly, quarterly, or annual owned schedules for renewals, notice windows, reporting, or other promises. Notice materializes uniquely numbered occurrences from the original calendar anchor, sends retry-safe local-calendar reminders, blocks offboarding that would strand open work, and requires immutable evidence for completion. Portfolio totals, renewal windows, paged work queues, agreement relationships, custom fields, history, document hashes, approvals, signature evidence, schedule/occurrence history, retention state, and signed exports remain connected to the same workspace.

For the visual, searchable walkthrough and role-by-role instructions, use the [HTML how-to guide](./docs/how-to/index.html).

## Try Notice locally

Requirements:

- Node.js `>=22.13.0 <23`
- npm `>=11.3.0 <12`

Use the current security-patched Node 22 LTS release for staging and production builds. Release CI runs the full validation suite on Node `22.23.2` and separately verifies the declared `22.13.0` engine floor; npm remains pinned to `11.3.0` for reproducible dependency and SBOM evidence.

Install and start the explicit evaluation mode:

```bash
node --version
npm --version
# If npm is older than 11.3.0:
npm install --global npm@11.3.0
npm --version
npm ci
npm run dev:evaluation
```

Some Node 22 installers still bundle npm 10. Upgrade npm before installation rather than using an older client against the reviewed lockfile. If the global npm location is not writable, use a Node version manager or another user-owned installation; do not use `sudo` in the project checkout.

Open the local address printed by the command. Evaluation mode is loopback-only and includes a synthetic administrator and synthetic agreement records. Use synthetic documents only.

Plain `npm run dev` is **not** an evaluation shortcut. Notice defaults to production behavior and fails closed when required production identity, workspace, storage, or provider configuration is missing. Local D1/R2 evaluation state is stored under the ignored `.wrangler/` directory.

Evaluation demonstrates the interface and lifecycle. It is not evidence that production SSO, scanning, delivery, signature, retention, backup, or customer policy has been accepted.

## Validate a checkout

Run the core repository gate:

```bash
npm test
```

It covers migrations, unit and security behavior, upload and creation retries, tenant isolation, TypeScript, lint, platform compatibility, production build, deployment packaging, rendered output, and evaluation/production API flows. Negotiation, portal, signed email, quarantine, idempotency, concurrency, approval, signature, recurring obligations, outbox, retention, and CRM-gateway boundaries are included.

Install Chromium once, then run both real-browser acceptance suites:

```bash
npx playwright install chromium
npm run test:e2e
npm run test:e2e:production
```

The first suite exercises evaluation-mode lifecycle, responsive, keyboard, and accessibility behavior. The second uses an isolated production-mode harness with separately signed admin/viewer Cloudflare Access JWTs and temporary D1, R2, DocuSeal, scanner, mail, and CRM fixtures. CI runs the core gate and both browser suites.

Dependency and release evidence is available separately:

```bash
npm run audit:production
npm run audit:all
npm run audit:signatures
npm run sbom -- --revision "$(git rev-parse HEAD)" --require-clean --output notice-sbom.cdx.json
```

CI repeats the full gate and retains a revision-bound CycloneDX inventory. These checks do not replace live provider, restore, security, or customer acceptance testing.

## What is ready, and what remains customer work

| Owner | Responsibility |
| --- | --- |
| Repository | CLM interface and APIs, enterprise configuration and governed intake, authorization, Cloudflare Access JWT validation, D1/R2 model and migrations, secure negotiation rounds, governed templates/workflows/authority, repository intelligence, portfolio/work queues, DocuSeal boundary, scheduled jobs, CRM gateway, provider protocols, diagnostics, retention controls, evidence, and tests |
| Deployer or implementation partner | Provision isolated Cloudflare resources, render customer configuration, apply migrations, connect identity and provider accounts, configure approved policies and authority records, run diagnostics, and collect acceptance evidence |
| Adopting customer | Approve administrators, roles and delegated authority, templates/playbook, approval rules, portal lifetime, Word-review procedure, provider accounts, data region, retention/hold policy, CRM mappings, operating controls, and production go-live |

This remaining work is bounded installation and customer decision-making. It is not a requirement to redesign the core CLM.

## Production implementation path

1. Run `npm ci`, `npx playwright install chromium`, `npm test`, `npm run test:e2e`, and `npm run test:e2e:production` on the exact reviewed revision.
2. Follow the [staging quickstart](./docs/DEPLOYMENT_QUICKSTART.md) to render ignored, customer-specific deployment files.
3. Provision isolated Cloudflare Workers, D1, and R2 resources; apply every committed migration in order through `0021`, then verify exact `notice_schema_state` marker 21.
4. Connect Cloudflare Access, DocuSeal, the required production scanner and outbound mail gateway, and any selected optional services. If customer-hosted OCR is selected, deploy its pinned model/profile privately behind that same scanner gateway.
5. Run readiness, provider diagnostics, scheduled-job checks, retention dry-runs, backup/restore, and synthetic end-to-end acceptance before admitting real data.

The tracked `wrangler.jsonc` is a customer-independent, binding-only source profile. **Do not deploy production directly from it.** Render customer-specific configuration and use the guarded deployment wrapper described in the [deployment quickstart](./docs/DEPLOYMENT_QUICKSTART.md) and [production setup](./docs/PRODUCTION_SETUP.md). The wrapper reads and validates each operator input once, canonicalizes its configuration and secrets in memory, and supplies them only to the exact repository-pinned Wrangler child through bounded process pipes; it never gives Wrangler the original input paths. It also rejects inherited Wrangler target overrides, withholds the Cloudflare credential from Git and the build, neutralizes Node/npm loader settings, and runs the exact pinned local Vinext build directly outside npm lifecycle configuration.

The repository provides signed protocols and executable conformance tests for the required scanner and outbound delivery gateway, plus the optional inbound-email gateway; those gateway services and any OCR model/runtime are external customer implementations. The scanner contract includes the optional customer-hosted OCR operation. The CRM gateway and its supported adapters are included as separately deployable code in [`integrations/crm-gateway`](./integrations/crm-gateway/README.md).

Production readiness can be checked without printing secret values:

```bash
npm run readiness
npm run readiness -- --json
```

The protected runtime equivalent is `GET /api/readiness`. A green technical report is necessary but is not business, legal, security, or regulatory approval.

## Important product boundaries

- One production workspace is selected per deployment, although every stored row and object remains workspace-scoped.
- Approval authority is configurable by lane, direct user or group, currency, and optional amount ceiling. Until a lane has any authority record, the documented role-based compatibility default applies; configuring or revoking the first record disables that fallback for the lane.
- The repository still includes source-controlled starter paper and starter approval policy, while administrators can govern versioned templates, deterministic workflow definitions, intake forms, clause playbooks, and exact publishing lifecycles in the product. Initial governed-template and playbook creation atomically commits the parent, immutable first version, durable retry receipt, and audit evidence so an interrupted request cannot strand an empty parent. Parent-only creation is not accepted; the separate version action remains available to repair an existing draft parent from an earlier deployment and to add later governed revisions.
- Agreement values are stored canonically in minor units with three-letter currency codes and zero-, two-, or three-decimal currency handling. Thresholds and authority ceilings compare only within the exact stored currency; Notice performs no foreign-exchange conversion.
- Production does not render, text-extract, or interpret PDF/DOCX in the application Worker or convert Word Track Changes into clause decisions. Before scanning, supported input receives complete bounded format inspection: valid UTF-8/control policy for text, balanced bounded RTF, PDF header/trailer/xref and referenced-object checks, or canonical DOCX ZIP-entry/layout and required-OOXML-part structure checks. Those checks are not semantic document proof and do not replace malware scanning or human review.
- Customer-hosted OCR is deployment-selected rather than an end-user model picker. Notice calls only the signed scanner gateway, accepts only the exact configured model/profile and source hash, caps returned text, records provenance, and never falls back. The customer supplies and operates the isolated engine, weights, parser, private network, capacity, and policy; no OCR/model provider is bundled.
- Repository intelligence is evidence-first but not a canonical-text oracle: search returns extractor-registered text and caller-supplied offsets/digests pinned to an immutable document version/hash. A human must compare citations with the protected paper; proposal acceptance records that review but does not mutate an agreement. OCR does not add an automated legal verdict or prove registered fragment provenance.
- The scanner and mail boundaries are protocols, not bundled vendor services. Customer accounts, credentials, domains, routes, and acceptance remain deployment inputs.
- The agreement register, agent-plan inbox, and approval/signature/obligation/schedule/event work queues use bounded cursor paging; the portfolio route returns bounded grouped totals, and CRM resync is paged. Full-document analytics and target-environment large-estate load evidence remain deployment work.
- Governed agents can discover the broader action catalog, but v1.1 execution is deliberately limited to `agreement.read` and human-approved server-derived `approval.request` routing. A grant sponsor cannot review their own plan; a different current active Admin must decide it. Read claim and completion both repeat the exact authority, expiry, agreement-version, and latest-paper fences. Risk, signature, configuration, intake, intelligence, and obligation mutations remain human-only.
- The optional DocuSeal MCP check is a compatibility diagnostic, not an agent execution channel: it initializes, acknowledges initialization, lists annotated tools, and invokes only read-only `load_template` to cross-check the configured template ID and ordered signer roles. It never invokes `create_template`, `send_documents`, or another mutation/open-world tool.
- Failed delivery can dead-letter for investigation, but there is no general operator replay console.
- Durable per-round portal budgets bound one bearer round even across token rotation. The in-process rate limiter remains defense in depth; production still needs distributed edge/WAF controls for cross-round, IP, tenant, and aggregate abuse.
- Hash chains and signed exports do not replace independently retained evidence where stronger evidentiary guarantees are required.
- Notice is not legal advice, a compliance certification, or automatically suitable for regulated production data.

See [What the adopting business still supplies](./docs/USER_GUIDE.md#what-the-adopting-business-still-supplies), [Operations](./docs/OPERATIONS.md), and the [Threat model](./docs/THREAT_MODEL.md) before approving production use.

## Repository map

| Path | Purpose |
| --- | --- |
| `app/` | User interface, API routes, and product services |
| `db/` and `drizzle/` | D1 schema, guards, and reviewed migrations |
| `integrations/crm-gateway/` | Separately deployed durable CRM gateway and adapters |
| `deploy/` and `scripts/` | Deployment plans, guarded release workflows, readiness, and SBOM tooling |
| `docs/` | User, implementation, architecture, operations, security, and release documentation |
| `tests/` | Unit, security, migration, rendering, deployment, API, and real-browser evidence |

## Documentation by audience

| Audience | Documentation |
| --- | --- |
| Everyday users | [HTML how-to](./docs/how-to/index.html), [User guide](./docs/USER_GUIDE.md) |
| Private-wealth pilot owners | [Starter implementation profile](./docs/WEALTH_MANAGEMENT_PROFILE.md) |
| Evaluators and contributors | [Product brief](./PRODUCT_BRIEF.md), [Contributing](./CONTRIBUTING.md) |
| Implementation partners | [Deployment quickstart](./docs/DEPLOYMENT_QUICKSTART.md), [Implementation handoff](./docs/IMPLEMENTATION_HANDOFF.md), [Production setup](./docs/PRODUCTION_SETUP.md), [Deployment](./docs/DEPLOYMENT.md) |
| Operators and security reviewers | [Operations](./docs/OPERATIONS.md), [Security](./SECURITY.md), [Threat model](./docs/THREAT_MODEL.md) |
| Integration and application developers | [Architecture](./docs/ARCHITECTURE.md), [API](./docs/API.md), [Governed agent API](./docs/AGENT_API.md), [DocuSeal integration](./docs/DOCUSEAL.md), [Customer-hosted OCR](./docs/CUSTOMER_HOSTED_OCR.md), [Gateway conformance](./docs/GATEWAY_CONFORMANCE.md), [CRM gateway](./integrations/crm-gateway/README.md) |
| Release owners | [Public release and adopter handoff](./docs/PUBLIC_RELEASE.md), [Third-party notices](./THIRD_PARTY_NOTICES.md) |

Source visibility is a release-governance decision. Follow [Public release and adopter handoff](./docs/PUBLIC_RELEASE.md) before changing repository visibility; `"private": true` in `package.json` prevents npm publication, not source-host disclosure.

## Contributing and project stewardship

Contributions are welcome when they preserve Notice's production boundaries and use synthetic data. Start with [CONTRIBUTING.md](./CONTRIBUTING.md), follow the [Code of Conduct](./CODE_OF_CONDUCT.md), and use the repository's structured bug or feature issue forms. The [governance model](./GOVERNANCE.md) identifies decision and release ownership; [SUPPORT.md](./SUPPORT.md) explains the best-effort community support boundary; [SECURITY.md](./SECURITY.md) is the only path for vulnerability reports. Notable public changes are recorded in the [changelog](./CHANGELOG.md).

## License

Licensed under the [MIT License](./LICENSE). Third-party services and dependencies retain their own licenses and terms.
