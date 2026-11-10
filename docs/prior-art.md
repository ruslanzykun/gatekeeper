# Prior art

How Gatekeeper relates to existing work. For each: what it does, why it is
not enough for the problem in [README.md](../README.md), and why Gatekeeper
does not simply build on it (or where it does). Corrections are welcome as
GitHub issues.

## EUDI Wallet and SD-JWT

**What it does.** The European Digital Identity Wallet presents identity
attestations to registered verifiers with selective disclosure, using
SD-JWT VC and ISO mdoc. SD-JWT
([RFC 9901](https://www.rfc-editor.org/rfc/rfc9901.html)) lets a holder
reveal only selected issuer-signed claims, with key binding to the verifier.
Predicates such as "over 18" exist where the issuer signed them as claims.
Zero-knowledge proofs are under discussion in the Architecture and Reference
Framework.

**Why it is not enough.** It models a one-time presentation to a known
verifier, not an agent that asks repeatedly and adaptively. There is no
defence against reconstruction across many questions, and its scope is
identity attestations, not arbitrary private data (health records,
contracts). Privacy researchers have also raised linkability concerns about
the framework.

**Relation.** Complementary. SD-JWT is Gatekeeper's first proof plugin
(SPEC § 9), and wallet attestations can become attribute sources.

- SD-JWT VC draft: <https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/>
- ARF discussion topic on zero-knowledge proofs:
  <https://eu-digital-identity-wallet.github.io/eudi-doc-architecture-and-reference-framework/3.0.0/discussion-topics/g-zero-knowledge-proof/>
- University of Luxembourg, privacy evaluation of the ARF:
  <https://orbilu.uni.lu/bitstream/10993/66335/1/Privacy_evaluation_of_the_European_Digital_Identity_Wallet_s_Architecture_and_Reference_Framework.pdf>

## MCP authorization and elicitation

**What it does.** The Model Context Protocol defines OAuth 2.1-based
authorization with scopes, and elicitation, through which a server can ask
the user for input during a session. The specification states that users
must explicitly consent to data access, and also that MCP itself cannot
enforce these principles at the protocol level.

**Why it is not enough.** Scopes are coarse (a tool or a resource), not a
predicate over one attribute. There is no notion of a question budget or of
what several answers reveal together.

**Relation.** Gatekeeper builds on MCP as a transport: it is planned as a
local MCP server, not a replacement for MCP.

- Authorization: <https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization>
- Elicitation: <https://modelcontextprotocol.io/specification/2026-07-28/client/elicitation>
- Coalition for Secure AI, MCP security paper (incl. consent fatigue):
  <https://www.coalitionforsecureai.org/wp-content/uploads/2026/03/model-context-protocol-security-1.pdf>

## Commercial AI firewalls and MCP gateways

**What they do.** Products such as [Lakera](https://www.lakera.ai/),
[Prompt Security](https://www.prompt.security/) and
[Nightfall](https://www.nightfall.ai/), and the open-source
[LLM Guard](https://github.com/protectai/llm-guard), detect prompt
injection and filter sensitive patterns in text, usually as a proxy in an
organisation's network or cloud.

**Why they are not enough.** They inspect the text flowing to and from a
model; they do not decide what an *answer* reveals about one person's data,
nor track what many answers reveal together. They mostly run outside the
person's device.

**Relation.** Different layer. A text filter can sit in front of an agent;
Gatekeeper sits in front of the data. The commercial products are
proprietary and cannot be embedded in an open app.

## AirGapAgent and agent-firewall research

**What it does.** AirGapAgent (Bagdasarian et al., 2024) restricts a
conversational agent to the data needed for its task, using a separate
model guided by contextual integrity. "Firewalls to Secure Dynamic LLM
Agentic Networks" (Abdelnabi et al., 2025) shows that coarser,
task-appropriate data sharing sharply reduces attacks. CaMeL (Debenedetti
et al., 2025) separates control flow from untrusted data to defeat prompt
injection by design.

**Why it is not enough.** Disclosure decisions are made by a language
model, which is not deterministic and hard to audit. These are research
prototypes, and none keeps a question budget across sessions.

**Relation.** This work supports the approach (minimal, task-appropriate
answers). Gatekeeper makes the decision with fixed, testable rules instead
of a model.

- AirGapAgent: <https://arxiv.org/abs/2405.05175>
- Firewalls to Secure Dynamic LLM Agentic Networks: <https://arxiv.org/abs/2502.01822>
- Defeating Prompt Injections by Design (CaMeL): <https://arxiv.org/abs/2503.18813>

## Query auditing and inference control

**What it does.** Research on statistical databases since the late 1970s
(Dobkin, Jones and Lipton 1979; Adam and Wortmann 1989; Kenthapadi, Mishra
and Nissim 2005/2013; Nabar et al. 2006) studies how answers to aggregate
queries can be combined to reveal individual values, how refusals
themselves leak, and when auditing is computationally hard. Simulatable
auditing decides refusals using only what the attacker already knows.

**Why it is not enough.** It was developed for central databases about many
people, not for one person's data on one device, and it exists as theory,
not as an embeddable component.

**Relation.** Gatekeeper uses it directly as the basis for its
reconstruction protection (SPEC § 8).

- Kenthapadi, Mishra, Nissim, *Denials leak information: Simulatable
  auditing*: <http://theory.stanford.edu/~nmishra/Papers/denialsLeakInformation.pdf>
- Nabar et al., *Towards Robustness in Query Auditing*, VLDB 2006:
  <https://www.vldb.org/conf/2006/p151-nabar.pdf>
- Adam, Wortmann, *Security-control methods for statistical databases: a
  comparative study*, ACM Computing Surveys 1989:
  <https://dl.acm.org/doi/10.1145/76894.76895>

## Solid

**What it does.** Solid stores personal data in pods and controls access
with Web Access Control (WAC) or Access Control Policies (ACP).

**Why it is not enough.** Access is granted per resource or container: a
reader gets the document or nothing. There are no predicates and no budget.

**Relation.** A Gatekeeper could sit in front of a pod as an attribute
source; Solid's access control does not replace it.

- WAC: <https://solidproject.org/TR/wac>
- ACP: <https://solidproject.org/TR/acp>

## Differential privacy

**What it does.** Differential privacy adds calibrated noise so that
statistics over many people reveal little about any one of them. OpenDP is
a maintained open-source library.

**Why it is not enough.** For a single yes/no question about one person,
noise means a wrong answer some of the time. "Is my permit valid?" needs the
correct answer.

**Relation.** Not used in v0.1. The budget idea is related in spirit, but
Gatekeeper's budgets count questions rather than adding noise.

- OpenDP: <https://github.com/opendp/opendp>

## Platform AI privacy (Apple, Google)

**What it does.** Apple Private Cloud Compute and Google Private AI Compute
protect *where* model inference runs; App Intents and similar APIs control
which app actions an assistant may invoke.

**Why it is not enough.** They do not decide how much an app's answer
reveals, and they are tied to one platform and not open.

**Relation.** Orthogonal. Gatekeeper can run inside an app on these
platforms.

- Apple Private Cloud Compute: <https://security.apple.com/blog/private-cloud-compute/>
- Google Private AI Compute (press coverage):
  <https://www.theregister.com/2025/11/12/google_touts_private_ai_compute/>
