# The secure and private agentic software factory

**Version 1.0** · 17 November 2026 · [CC BY 4.0](LICENSE)

What it takes for a factory of AI agents to be secure and private: eleven
properties, all mandatory, each one something a buyer can test.

> Take it, use it, measure every vendor with it. Comment in
> [Discussions](../../discussions), propose changes in
> [Issues](../../issues). See [CONTRIBUTING](CONTRIBUTING.md).

## The agent is already compromised

Here's the starting point for everything that follows. Treat every AI agent
that writes your code as if an attacker is steering it right now.

That isn't paranoia. An agent reads issues, code comments, dependency READMEs,
web pages and tool output, and any of them can carry instructions. Nobody has
solved prompt injection. So nobody can promise you an agent is honest, and a
factory built on that promise is built on sand.

The OWASP GenAI Security Project puts it more bluntly. Its guide to securing
agentic applications says coding agents are, "by definition, a Lethal
Trifecta": they read untrusted input, they hold sensitive data, and they can
reach the outside world ([OWASP, Securing Agentic Applications V2, §5.5.1][saa]).
That's not a description of a bad agent. It's a description of the job.

Think about a contractor redoing your kitchen. You don't hand them your house
keys, your bank card and the alarm code. You let them into the kitchen, and
someone who doesn't work for them checks the wiring before the walls close up.
Not because this contractor is a crook. Because if they were, you'd want the
damage to stop at the kitchen.

That's the whole idea of a secure and private agentic software factory.

## What we mean by "factory"

The phrase "software factory" already means two things, and this is neither.

In the US Department of Defense, a software factory is a hardened DevSecOps
platform where human developers work: Kessel Run, Platform One
([DoD CIO, DevSecOps Fundamentals][dod]). In 2022, CNCF TAG Security published
a [Secure Software Factory reference architecture][cncf] about build
provenance, and OpenSSF's [FRSCA][frsca] implements it. Both are good work.
Both assume a human writes the code.

An **agentic software factory** is a system where AI agents produce the
software changes and people set the intent and own the outcome. Human-run
pipelines are the ancestor. They're not the thing.

A **secure and private** agentic software factory is one that meets all eleven
properties below. Not most of them. All of them. Miss one and it's an AI coding
tool with good intentions, which is fine, but it isn't this.

Every property comes with a test, written as the question a buyer asks a
vendor, with a yes or a no for an answer. If a property can't be tested, it doesn't belong here.

The factory's job ends at the merge. Agents that deploy and run software are a
bigger problem with different threats, and they deserve their own document.

## The eleven properties

### 1. The factory never grades its own homework

Something outside the factory checks every change it makes. That check sits
behind its own trust boundary, and the factory can't change it, skip it or
weaken it. It applies the same policy to the factory's changes as it does to any
other author's, with no fast lane. You can add checks of your own. And the model
that reviews a change is never the model that wrote it.

The same vendor can sell you the factory and the check. What matters is the
wall between them, not the logo.

**The test:** Can the factory change, skip or weaken the check on its own work?
Show me why not.

### 2. Somebody's name is on every merge

Agents can't be held accountable. People can. Policy may merge a change on
its own, within limits you set, but a named person owns every policy that
merges code. Every merge traces back to either a person who approved it or a
rule that a person owns.

A name on the merge is only worth something if that person saw the truth. An
agent can talk its way to a yes: a confident description that says "fixes the
SQL injection, nothing else," sitting on top of a diff that also touches
authentication. OWASP lists this as its own risk, human-agent trust
exploitation ([ASI09][asi]). GitLab's security team has the best line on it:
"Tool calls are evidence, and an agent's explanation of its own reasoning is
secondary" ([GitLab][gitlab]).

So the approver sees the evidence first: the full diff, the checks and what
they found, and what the agent read. Anything the agent wrote about its own
work is labelled as the agent's claim. Each approval records which evidence
record the approver saw, and if that record changes, the approval doesn't
carry over.

**The test:** Pick any merged change. Who approved it, or which rule did, and
who owns that rule? Then show me exactly what the approver saw.

### 3. Your code goes only where you can see it

Every party that processes your code is named, and every one of them is barred
by contract from training on it. You choose which model providers and regions
are allowed. Retention is bounded and you can see the schedule. When you ask
for deletion, you get proof. Your content is encrypted separately from every
other customer's, and in a hosted factory nothing about your runs, records or
data can be reached from another customer's.

**The test:** Show me the processor list, the retention schedule and a deletion
record, and explain where my encryption boundary sits.

### 4. You can run it in your own house

The factory must be able to run inside your environment: your cloud account or
your data centre. Its models can run in your own cloud tenancy, or be external
APIs you choose. A hosted version can exist, and plenty of buyers will be happy
with it. But "you can run it yourself" is part of the definition, because it's
the only privacy promise that doesn't depend on trusting the vendor.

**The test:** Can I deploy it in my account, and which of my model accounts can
it use?

### 5. The blast radius is one run

Each run starts in a fresh environment and nothing carries over to the next.
There's no usable credential inside the agent's environment. Anything
privileged, such as pushing a branch, calling a model or anything touching
production, happens outside it on the agent's behalf. Network access is denied
by default, and every way out goes through one choke point that records it. No
other customer's workload shares the agent's kernel.

This is the property that decides whether your incident report says "the agent
was hijacked" or "the attacker has our GitHub token."

It's also how a factory lives with the lethal trifecta instead of pretending
it away. OWASP's answer is Meta's "Rule of Two": give an agent at most two of
untrusted input, sensitive data, and the ability to change things or talk to
the outside world, or "separate the legs across independently authorized
components" ([Securing Agentic Applications V2, §2.7.3][saa]). A coding agent
can't give up untrusted input or your source code. So the third leg moves
outside the agent.

That moves the credentials outside too, and they need rules of their own.
OWASP calls the failure identity and privilege abuse ([ASI03][asi]). Every
credential the factory uses on the agent's behalf is scoped to one repository
and one action, expires within the run, and is issued when it's needed rather
than stored, through workload identity wherever the platform supports it. One
organisation-wide token that never expires doesn't pass, no matter where it's
kept.

One thing is allowed to carry over: memory, on a short leash. A factory that
remembers your conventions does better work. A factory that remembers whatever
a hijacked run told it is poisoned for good, which OWASP calls memory and
context poisoning ([ASI06][asi]). So an agent can propose a memory but never
write one. A person or the independent check approves it first, the same way
code gets approved. Memory is versioned, you can wipe it, and every run's
record names the version it used.

**The test:** Show me the credential inventory inside a run, the egress log for
one, and what else shares its kernel. Show me the credential used for the last
push, what it could reach, and when it expired. Show me who approved each
memory this run read.

### 6. Write down what the agent read

Every input the agent read is recorded along with its source: your issue, a
third-party package, a web page, a tool result. That includes the agent
instruction files that live in the repository itself, like `CLAUDE.md` or
`.cursorrules`, which OWASP rightly treats as untrusted, because they travel
with every fork ([Securing Agentic Applications V2, §5.5.1][saa]).

Notice what's missing. This definition does not require prompt injection
filtering. No filter reliably catches it, and we're not going to reward
vendors for claiming theirs does.

OWASP recommends a "context admission" gate that checks untrusted content
before it reaches the model ([Securing Agentic Applications V2, §3.1.7][saa]).
Use one. It's good defence in depth. It just doesn't count here, and OWASP's
own guide says why: injection payloads "can evade content filters," so
"runtime isolation should be prioritized over content sanitization alone."
Every property in this definition has to hold on the day the gate misses. The
gate lowers the odds. The rest of this list limits the damage.

What counts is being able to answer the first question after an incident:
what did the agent read that made it do that?

**The test:** Show me the input record for this run.

### 7. You keep the receipt

Every change carries a record: which model produced it, which version, what it
read, which checks ran and what they found, and who or what approved it. The
record is signed and tamper-evident, in an open format, and you hold a copy.

A receipt you can check beats a store's word that you paid. Someone with no
account with the vendor can verify the record is genuine and complete, and it
stays valid after you leave. The expected way to do this is [in-toto][intoto]
attestations or [SLSA][slsa] provenance, but the definition asks for the
outcome, not the format.

The receipt also names the factory's own tools: the agent CLI, every MCP
server, skill and prompt template, and the image the run started from, each
pinned to an exact version or digest. Nothing gets fetched live without a pin.
A poisoned MCP server hijacks every run that loads it, for every customer at
once, without the agent reading a single bad input. OWASP lists that as agentic
supply chain vulnerabilities ([ASI04][asi]) and asks for a "signed,
version-pinned catalog manifest" of tools ([Securing Agentic Applications V2,
§6.4.2][saa]). When a tool goes bad, you should be able to find every run that
loaded it with one query.

**The test:** Hand a change's record to a third party. Can they verify it
without calling the vendor? Then tell me which of my runs loaded a given
version of a given MCP server.

### 8. Privacy is engineered into what it builds

The factory keeps an inventory of the personal data your codebase handles:
what it is, where it flows and why it's held. When a change touches personal
data or adds new personal data, the factory runs a privacy threat analysis and
a data-minimisation check, and both go into that change's record.
[LINDDUN][linddun] is a good method for the threat analysis. It isn't a
required one.

Without the inventory, a factory can't know when a change touches personal
data, and any privacy review it claims is a guess.

**The test:** Show me the inventory. Now show me the privacy threat analysis
for the last change that added a personal data field.

### 9. Some things get stopped at the door

Some damage happens before any merge. A secret pushed to a branch is already
leaked: it's in the history, the CI logs and the forks. A typosquatted package
that CI installs from a pushed branch has already run its install script.

So before anything leaves the agent's environment, deterministic checks outside
the agent's control block secrets, and block any new dependency your policy
doesn't allow, including packages that don't exist or were registered last
week. That last one isn't hypothetical. OWASP warns about "slopsquatting,"
attackers registering the names a model is likely to hallucinate
([Securing Agentic Applications V2, §6.4.2][saa]).

"Deterministic" means no model decides. A model-based check can be talked past
by the same text that hijacked the agent.

**The test:** Show me what runs before a push, and show me that no model makes
the call.

### 10. It fails closed, and you hold the off switch

A run that can't finish stops, nothing half-done reaches your repository, and a
person hears about it. Every run has caps on time, cost and attempts against
the independent check, and hitting any cap stops the run and escalates. That
last cap is the one people miss. An agent that gets unlimited tries against a
check is searching for a way past it.

Failing closed applies to the controls too. OWASP's rule for an enforcement
point that errors out or can't be reached is the right one: "denied or
escalated, never allowed through by default. A hook that fails open can be
bypassed by anyone able to make it fail" ([Securing Agentic Applications V2,
§3.3.7][saa]).

Every refused egress is recorded in the run's record. And you have a kill
switch, like the main water valve in your basement. One action stops every run
and revokes the factory's access to your repositories and model accounts,
without calling the vendor first.

**The test:** Show me the caps on a run. Then let me pull the switch.

### 11. The vendor's people don't get a master key

By default, nobody at the vendor can read your code or your data. Access needs
a grant you approve, for a stated purpose, that expires. Every access appears
in a log you can see, and an empty log has to be a real possibility. Support
works from metadata until you grant more.

Think of a valet key: it starts the car and doesn't open the trunk.

**The test:** Show me the access log for my tenant, and the grant behind every
entry.

## Claiming it

There's no certification body and no badge to buy. A vendor that claims this
definition publishes a conformance statement that goes through all eleven
properties, says how it meets each one, and names the evidence you can ask for.

That works because every property was written to be tested. A false claim
falls apart the moment a buyer asks for the evidence.

## Changing it

This is version 1.0. Comments happen in public, in this repository's
Discussions. Proposals to change a property happen in its Issues, and wording
fixes come as pull requests. The editors listed in
[CONTRIBUTING](CONTRIBUTING.md) decide what goes in.

One rule protects everyone who builds against it: minor versions only clarify
the wording and never change what passes. A new property, or a stricter one,
is a new major version. Conformance statements name the version they claim, so
"we meet 1.x" means the same thing next year as it does today.

## Where this goes

The tools that write our code changed faster than the controls around them.
IDC found that only 28% of organisations using AI coding assistants require
peer review of AI-assisted changes, and only 23.3% log or audit their use
([TechTarget][techtarget]). This is our attempt to say what the controls have
to be, plainly enough that a buyer can hold a vendor to it.

We believe agents will write most of the world's software. We believe the
factories that build it have to assume the worst about the agent and prove the
best about the result. We believe security and privacy can be the reason to
buy a factory, not the cost of having one.

## Sources

- OWASP GenAI Security Project, *Securing Agentic Applications V2: A Builder &
  Defender's Guide*, version 2.0, October 2026, CC BY-SA 4.0. Sections 2.7.3,
  3.1.7, 3.3.7, 5.5.1 and 6.4.2. [genai.owasp.org][saa]
- OWASP GenAI Security Project, *Top 10 for Agentic Applications for 2026*,
  9 December 2025. ASI03, ASI04, ASI06 and ASI09. [genai.owasp.org][asi]
- CNCF TAG Security, *Secure Software Factory reference architecture*, 2022.
  [tag-security.cncf.io][cncf]
- OpenSSF, *FRSCA: Factory for Repeatable Secure Creation of Artifacts*.
  [github.com][frsca]
- DoD CIO, *DoD Enterprise DevSecOps Fundamentals*. [dodcio.defense.gov][dod]
- GitLab, *Securing the software factory at machine speed*, 18 September 2026.
  [about.gitlab.com][gitlab]
- TechTarget, *Vendors build safeguards for autonomous software factories*,
  15 October 2026, citing IDC survey data. [techtarget.com][techtarget]
- [in-toto][intoto], [SLSA][slsa], [LINDDUN][linddun].

[saa]: https://genai.owasp.org/resource/securing-agentic-applications-v2/
[asi]: https://genai.owasp.org/2025/12/09/owasp-genai-security-project-releases-top-10-risks-and-mitigations-for-agentic-ai-security/
[cncf]: https://tag-security.cncf.io/community/resources/security-whitepaper/v1/secure-software-factory/
[frsca]: https://github.com/buildsec/frsca
[dod]: https://dodcio.defense.gov/Portals/0/Documents/Library/DoD%20Enterprise%20DevSecOps%20Fundamentals%20v2.5.pdf
[gitlab]: https://about.gitlab.com/blog/securing-the-software-factory-at-machine-speed/
[techtarget]: https://www.techtarget.com/it-infrastructure/news/366651847/Vendors-build-safeguards-for-autonomous-software-factories
[intoto]: https://in-toto.io/
[slsa]: https://slsa.dev/
[linddun]: https://linddun.org/
