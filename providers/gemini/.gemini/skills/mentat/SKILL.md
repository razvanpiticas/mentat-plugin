---
name: mentat
description: Connects to and drives the Mentat MCP server at mcp.mentat.business — the product's own data, tenant-scoped, under the caller's own permissions. Use when working with Mentat data, when a Mentat tool refuses a call, or when connecting a harness to the Mentat server.
---

# Mentat

Mentat's MCP server fronts the product's BusinessIntelligence service. It holds no database of its
own: every tool call is an HTTP call to a service that already enforces the product's own rules,
made **as the person using Gemini** rather than as a service account.

That is the fact everything else follows from. The server never sees more than the caller does, so
a tool that refuses is usually reporting the caller's own access rather than a fault.

## Connecting

The server is declared under `mcpServers` in `.gemini/settings.json` in this project.

Sign-in is OAuth against the Keycloak realm at `keycloak.quantarcane.io`, using the public client
`mentat-public-client` and the loopback callback on port `8125`. Both halves are fixed: a different
port is a redirect-uri mismatch, not a preference.

**Call `server_info` first when anything is wrong.** It answers without touching any downstream
service, so it separates "the server is unreachable or I am not signed in" from "the server is fine
and the call was refused".

## What access means here

Two permissions gate everything:

| Permission | Covers |
| --- | --- |
| `mcp:tools.read` | Every tool that only reads |
| `mcp:tools.write` | Every tool that changes something |

They are granted through Mentat's own roles screen, per tenant, like every other permission in the
product. A session where reads succeed and writes answer 403 is a role problem, not a connection
problem — and re-authenticating does not fix it, because the token has to be minted again *after*
the role is granted.

Every call is also tenant-scoped from the caller's token. A caller who belongs to no tenant gets
403 on everything, which is correct rather than broken.

## The tools

Seventy-seven tools, all over BusinessIntelligence: nineteen reads (portfolios, projects, the canvas,
one block by code, one claim, one experiment, the method cards, a project's plan and the runs of its
operations) and fifty-eight writes (entries, claims, experiments, evidence, questions, risks, ideas,
contradictions, project insights, the plan's rows, operation runs and method gates). The full table
with permissions and which tools take which version is in [reference/tools.md](reference/tools.md).
**How to use them — which kind, which fields, what order BusinessIntelligence enforces — is the
`mentat-canvas` skill; load it for any read or write on a canvas or a plan.**

`get_roadmap` is the one orientation call beyond the four above: it says where a project stands in the
method, what it has already run and what is next.

`server_info` reports the tool list; when it disagrees with this file or any document, it is right.

## Reading a refusal

A refused call names the arguments at fault and says whether retrying is worth anything. Read it
rather than resending the same call.

- **The refusal names an argument** — fix the argument and retry once.
- **The refusal is a 403** — a permission or tenant problem. Retrying cannot fix it. Tell the user
  which permission is missing.
- **The refusal is a 401** — the token expired or was never minted. Sign in again.
- **The refusal is a CONFLICT naming a canvas version** — the canvas moved under you. Re-read
  and resend with the version it names; the `mentat-canvas` skill explains the version rule.
- **The refusal is a CONFLICT naming a roadmap version** — somebody else changed the project's plan.
  That is a different number from the canvas version: re-read with `get_roadmap` and resend with the
  one it names.
- **The refusal is a CONFLICT quoting a rule of the business model** — the call is not allowed in the
  canvas's current state. Change what you asked for; do not resend.
- **The refusal says a downstream holds nothing at that address** — an id or a code named nothing.
  Read what exists and address it by what the read answered.

A refusal that fits none of these is worth a question rather than a second attempt.
Ask the user directly to clarify what you cannot infer.

For symptoms that survive a retry, read
[reference/troubleshooting.md](reference/troubleshooting.md).

## Rules that are easy to get wrong

**Never invent a tenant.** No tool takes a tenant argument, and none should — the tenant comes from
the token. A call that appears to need one is a sign the wrong tool was chosen.

**Do not cache what a read returns across turns.** It is live state that another person in the
same tenant can change between calls, and every write is checked against the canvas version you
last read.

**A write is not confirmed until the tool says so.** Every write returns the row it wrote; if the
call refused, nothing was written unless the message says otherwise, and reporting otherwise to
the user is worse than reporting the failure.
