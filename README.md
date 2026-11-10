# Gatekeeper

**Status: concept / specification draft v0.1. There is no implementation yet.**
Nothing in this repository is ready to use. The specification is a draft and
will change.

Gatekeeper is a planned open, reusable component that runs on a person's own
device, between AI agents and that person's private data. It answers an
agent's questions with the minimum needed (yes / no, a coarse range, or a
refusal), only under a policy the person has consented to, and it logs every
disclosure.

## The problem

AI agents are being connected to personal data, often through the
[Model Context Protocol (MCP)](https://modelcontextprotocol.io/). Today
"access" almost always means the agent receives the whole field or the whole
document, even when it only needs one fact. And even a strict yes/no
interface leaks if the agent may ask freely: many harmless questions ("valid
in March? in April? in May?") reconstruct the underlying value step by step.

## One example

An agent helping with a job application asks:

> Is the residence permit valid for more than 6 months from today?

Gatekeeper checks that this agent, for this stated purpose, may ask this kind
of question about this attribute, and that its question budget is not used
up. It answers **"yes"**. The agent never sees the expiry date, the document
number or the document itself. The person can read the entry in the audit
log afterwards.

## How it works

```
  AI agent  (e.g. via MCP)
     |  typed question: attribute + predicate + purpose
     v
 +------------------------------ Gatekeeper (on device) ----+
 |  Policy       who may ask what, for which purpose,       |
 |               how often, until when                      |
 |  Arbitration  per question: allow / generalise / refuse  |
 |               (deterministic, no language model)         |
 |  Disclosure   predicate result, coarse range or refusal  |
 |               (optional proof via a plugin, e.g. SD-JWT) |
 |  Audit log    hash-chained record of every disclosure    |
 +----------------------------------------------------------+
     |  reads typed attributes
     v
  Local data  (the host app's own store; never leaves as raw values)
```

- **Policy**: access rules the person consents to (requester, purpose,
  allowed predicates per attribute, budgets, expiry, revocation).
- **Arbitration**: the decision for one question, made by fixed rules that
  can be audited and tested.
- **Disclosure**: the controlled result. A refusal looks the same whatever
  the underlying data is.
- **Audit log**: a tamper-evident, hash-chained log the person can read.
- **Reconstruction protection**: question budgets per requester and
  attribute, generalisation, detection of overlapping queries, and refusals
  decided without looking at the true answer ("simulatable auditing").
- **Proof plugins**: when a recipient needs verifiable evidence, a plugin
  built on an existing, audited library attaches it. SD-JWT
  ([RFC 9901](https://www.rfc-editor.org/rfc/rfc9901.html)) is the first
  planned plugin. Gatekeeper contains no home-made cryptography.

Details: [SPEC.md](SPEC.md). Worked example: [examples/](examples/).

## Non-goals

- Making agents trustworthy. Gatekeeper limits what an agent learns; it
  does not judge what the agent does with it.
- Prompt-injection detection or text filtering.
- Statistics over many people (that is differential privacy's domain).
- New cryptography. Proofs come from existing libraries. Functional and
  predicate encryption are a research direction only, not a deliverable.
- Storage, sync or encryption of the data itself. The host app owns that.
- A cloud service. Gatekeeper runs where the data is.

## Status and roadmap

Today: concept and this specification draft. Funding is being sought to
build the reference implementation.

Planned milestones:

1. **Specification v0.1**: question language, disclosure types, purpose and
   consent objects, audit-log format as JSON Schema, with test vectors.
2. **Core in Dart**: attribute interface, predicate evaluation,
   generalisation, purpose binding, refusal as a normal answer, hash-chained
   audit log on libsodium.
3. **Reconstruction protection**: question budgets, simulatable refusal,
   overlap detection, and an open attack test suite (e.g. binary search on a
   date) as the benchmark.
4. **MCP adapter and consent UI**: a local MCP server, a reusable Flutter
   dialog for consent and the audit log, an example agent.
5. **Proof interface**: plugin API, first plugin SD-JWT (RFC 9901) via an
   existing Dart package; feasibility study for range proofs.
6. **Two reference cases and release 1.0**: migrants' documents (Relocat)
   and a health example ("vaccination less than 10 years old?"),
   documentation, external review.

The reference implementation is planned in Dart (Flutter: Android, iOS,
desktop). The specification is language-neutral so others can implement it
in Kotlin, Swift or Rust and test against the same vectors.

### Origin

The idea comes from [Relocat](https://relocat.org), an end-to-end encrypted
document vault. Its privacy firewall decides per data category and
counterparty, but it hands an agent raw fields such as an expiry date.
Gatekeeper is meant to replace that with predicates. Relocat is the planned
first reference case; Gatekeeper itself is generic (health, legal, finance,
public administration).

## Prior art

One line each; the full comparison is in [docs/prior-art.md](docs/prior-art.md).

- **EUDI Wallet / SD-JWT**: one-time selective disclosure of credential
  claims to a known verifier, not repeated questions from an agent.
- **MCP authorization and elicitation**: grants access per tool or scope,
  not per predicate; no question budget.
- **Commercial AI firewalls / MCP gateways**: filter prompts and text,
  usually as a cloud proxy; they do not decide what an answer reveals.
- **AirGapAgent and agent-firewall research**: a language model decides
  what to share; Gatekeeper decides by fixed, testable rules.
- **Query auditing / inference control**: the theory we build on, made for
  central statistical databases, not one person's data on one device.
- **Solid**: document-level access control; all or nothing.
- **Differential privacy**: protects individuals in aggregates; noise is not
  an answer to "is my permit valid?".

## Licence

[Apache License 2.0](LICENSE). See [NOTICE](NOTICE).

## Contact

Please use [GitHub issues](https://github.com/ruslanzykun/gatekeeper/issues)
for questions and feedback on the specification. Security reports: see
[SECURITY.md](SECURITY.md). Contributions: see
[CONTRIBUTING.md](CONTRIBUTING.md).
