# When it does not work

Run the checks in order. The first two are cheap and settle most cases.

| # | Check | Proves |
| --- | --- | --- |
| 1 | Call `server_info` | Separates "unreachable or not signed in" from "signed in, call refused" |
| 2 | Open `https://mcp.mentat.business/health` in a browser | The deployment is up. It answers `{"status":"ok"}` anonymously |
| 3 | Call `list_portfolios` | This server can reach BusinessIntelligence as you, and you may read there |
| 4 | Call `record_project_insight` on a project you own | Writes are allowed for your organisation and your role |

## Symptom index

| Symptom | Cause | Fix |
| --- | --- | --- |
| Browser shows a redirect-uri mismatch during sign-in | The callback port in this harness's config is not one the realm registers | Use port `8125`, or register the port you used on `mentat-public-client` |
| Sign-in never opens a browser, or fails naming client registration | The harness is trying dynamic client registration, which this realm refuses | The harness must be told to use client id `mentat-public-client`. A harness with no field for it cannot sign in to this server |
| Every tool answers 401 | No token, or an expired one | Sign in again. If it recurs immediately, the token's audience does not name this server |
| Every tool answers 403 | The account belongs to no tenant, or has neither permission | Grant a role carrying `mcp:tools.read` on the roles screen, then sign in again |
| Reads work, every write answers 403 | The account has `mcp:tools.read` but not `mcp:tools.write`, or the organisation is read-only | Grant the write permission, then sign in again — the old token does not gain it. A read-only organisation is a billing state, fixed on the billing screen |
| A permission was granted but the call still answers 403 | The token was minted before the grant | Sign out fully and back in. Refreshing is not enough |
| `SERVER_MISCONFIGURED` saying BusinessIntelligence would not accept this server's credentials | The token has `mcp:tools.*` but not the BusinessIntelligence permission the call needs | Grant `business-intelligence:canvas.write` (or the read one) on the same role, then sign in again |
| `CONFLICT` naming a canvas version on every write | The write is sent with a version older than the canvas | Read the block again and send the version it answers; carry each answered version into the next write |
| `NOT_FOUND` on `get_block` listing codes | The code is not on this canvas | Use one of the codes listed; custom blocks have codes of their own |
| `server_info` answers but names a different deployment | The harness is pointed at another host | Check the URL in this harness's MCP config |
| Tools are missing entirely from the session | The MCP server is configured but not loaded | Reload the harness. Most do not pick up a new server mid-session |

## The one that is not a fault

A caller who belongs to no tenant gets 403 on every tool, including `server_info`. That is the
tenant isolation working, not a broken deployment. The fix is joining a tenant, not changing
anything about the server or the client.

## What this cannot tell you

If `/health` answers and `server_info` still fails for every account in every harness, the problem
is in the deployment rather than in any client — the realm, the ingress route, or the token
audience. That is a change in `src/services/Mcp/`, not here.
