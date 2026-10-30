# The front door, field by field

Load this when checking the research tools, writing the status, choosing the next action, or printing
the menu. The skill carries the seven steps; this carries what each line is read from.

## The five status lines

| Line | Read from | Say |
| --- | --- | --- |
| Where the venture stands | `get_project`: `tier`, `status`, `overallConfidence`; the brief's `mission`: `referenceCode`, `metricName`, `latestValue`, `metricUnit`, `metricTarget`, `metricDirection`, `deadline`, `latestAsOf` | "alcooldiscount is at tier Build, Active, confidence 0.31. G-01 online revenue: 412,000 EUR as of 1 September against 24,000,000 EUR by 31 December 2027." When `latestValue` is null: "G-01 has no measurement yet." When `mission` is null: "This project has no mission yet — the planner writes one with you." |
| The plan | the brief's `project.roadmap`: `lastCompleted`, `inFlight`, `next` (each `slug`, `title`, `position`, `tier`, `status`), `gatesAwaitingPerson` (the gate is due for evaluation), `hasThesis`; the brief's `project.status` (`AwaitingUnlock` = a passed gate waits for the person); `get_gate_status`: `roadmap.completedOperationsInTier`, `roadmap.pendingOperationsInTier`, `gateName`, `wouldPass`, `currentScores` (each `blockCode`, `confidence`, `threshold`, `passed`), `latestEvaluation`, `consecutiveFailures` | "Last finished: 7 customer-profile. Next: 8 persona. 3 of 12 operations of this tier done; the gate to Design would not pass yet (CS 0.42 of 0.60, VP 0.10 of 0.60)." Add "A run of 9 is paused on your answer." when `inFlight.status` is `Paused`; "The gate to Design is due: every row of this tier is done." when `gatesAwaitingPerson` is not empty; "The gate to Design passed and waits for your unlock." when `status` is `AwaitingUnlock`; "The last gate check failed; the plan goes back to \<nextOperationSlug\>." when `latestEvaluation` failed; "No plan yet: the roadmap has no thesis." when `hasThesis` is false |
| What happened last | the brief's `project.changesSinceLastRun` (`entityCode`, `subject`, `kind`, `actorKind`, `runId`, `occurredAtUtc`) and `changesTruncated`; `list_runs(portfolioId, projectId, pageSize: 3)`: `kind`, `status`, `triggeredBy`, `agentName`, `routineName`, `endedAtUtc`, `summary` | "Since the CEO last ran: 6 entries and 3 hypotheses written by the agent, 2 entries edited by you. Last runs: Heartbeat, Completed this morning — 'Ran customer-profile, one question for you'; …" When `changesTruncated`: "more than 50 changes — say 'show me the feed' for the rest" |
| What waits for you | the brief's `project.inboxPendingCount`, `myOpenWork` (codes and titles), `resolvedSinceLastRun` | "3 things wait in your inbox. Open work: W-04 build the watchlist (In progress). You answered 1 escalation since the last run." When the count is 0: "Nothing waits in your inbox." |
| Open on the canvas | the brief's `project.overview`: `blocks` (each `code`, `name`, `entryCount`, `hypothesisCount`, `confidence`), `openQuestionCount`, `topRisks` (count), `hypothesesByStatus`, `runningExperiments` (count), `openContradictions` (count) | "Nothing written yet on VP, CH, CR, RS, KA or KP. 12 open questions, 4 risks above the line, hypotheses: 9 untested, 2 under test, 1 validated; 1 experiment running; no open contradictions." |

The empty blocks are the ones whose `entryCount` is 0; name them by code rather than listing what every
block holds, and say "every block holds something" when none is empty. Never call `get_canvas` at the
front door: it is large, and these counts are what the line needs.

## The next-action rule

The rule lives in `mentat-advance/reference/next-action.md`; read it there when it is installed. In short, from the plan line, the first that applies: (1) the project is not active → "Unlock the next tier, or not: the gate to <tier> passed and waits for you on the project screen" — no skill; (2) a run is paused and its ask was answered → "Resume the paused run of <slug>: the answer is in." — `mentat-advance`; paused and unanswered → "The run of <slug> waits for your answer in the inbox."; (3) `hasThesis` is false → "Make the plan: the roadmap has no thesis." — `mentat-planner`; (4) `gatesAwaitingPerson` is not empty, or `get_gate_status` says the gate would pass with rows pending → "Check the gate to <tier>." — `mentat-gate-check`; (5) a next row → "Run <position> <slug>." — `mentat-operation`, or "Write the job for the move <idea code>." — `mentat-advance`; (6) otherwise, no row left → "The plan is finished: make the next one with the planner." — `mentat-planner`.

Say the sentence, name the skill, ask "Do it now?" and wait.
STOP and call the AskUserQuestion tool to clarify.

## The menu

Ten lines, in the order a person meets the work. Each line is that skill's own first sentence, so the
two cannot disagree; the build refuses a line that names a skill directory that does not exist, and
refuses a line whose words the named skill's own description does not use. A skill not yet in this
version carries the suffix; picking it answers "That part of Mentat is not installed yet; the plugin
update will bring it." and shows the menu again.

```
What else I can do:
 1. Make or redo the plan            — /mentat:mentat-planner
 2. Do the next row of the plan      — /mentat:mentat-advance
 3. Run one operation                — /mentat:mentat-operation
 4. Propose hypotheses on a block    — /mentat:mentat-hypothesis
 5. Design or run an experiment      — /mentat:mentat-experiment
 6. Check the gate                   — /mentat:mentat-gate-check
 7. Distil the insights              — /mentat:mentat-distiller
 8. Type the links                   — /mentat:mentat-linker
 9. Routines: switch on, run now     — /mentat:mentat-routines
10. Update the plugin                — /mentat:mentat-update
Or just say what you want written or read on the canvas.
```

On Codex the prefix renders `$`; on Claude Code it carries the plugin's namespace, `/mentat:`; on
harnesses with no command the line names the skill in words. A line carrying the suffix still names
its skill, so a person knows what is coming and the build can tell when it has arrived: when a skill's
story lands, its author deletes the suffix from that line, and the build refuses a stale one the same
day.

## The research tools

**The check.** One line, the same on Windows, macOS and Linux. It prints what this machine has and
the "not now" dates; it never prints a key:

```
node -p "const fs=require('fs'),p=require('path'),os=require('os');const win=process.platform==='win32';const onPath=(n,extra)=>[...process.env.PATH.split(p.delimiter),...extra].some(d=>d&&['',...(win?process.env.PATHEXT.split(';'):[])].some(e=>fs.existsSync(p.join(d,n+e))));const env=p.join(os.homedir(),'.config','last30days','.env');const f=p.join(win?p.join(process.env.LOCALAPPDATA,'Mentat'):p.join(os.homedir(),'.local','state','mentat'),'research-tools.json');JSON.stringify({agentReach:onPath('agent-reach',[p.join(os.homedir(),'.local','bin')]),firecrawl:onPath('firecrawl',[]),ytDlp:onPath('yt-dlp',[]),scrapeCreatorsKey:Boolean(process.env.SCRAPECREATORS_API_KEY)||(fs.existsSync(env)&&/^SCRAPECREATORS_API_KEY=\S+/m.test(fs.readFileSync(env,'utf8'))),notNow:fs.existsSync(f)?JSON.parse(fs.readFileSync(f,'utf8')).notNow:{}})"
```

The other two are read from this session, not the machine: Treg is here when the session has the
`treg` tools; last30days when a `last30days` skill is listed. Firecrawl also counts as here when a
`firecrawl` skill is listed.

**What each adds**, one line each, only for the missing ones:

| Tool | Say |
| --- | --- |
| agent-reach | "agent-reach reads Reddit, YouTube with subtitles, GitHub, X and the web for free, so I can quote what people say, with the link." |
| ScrapeCreators | "ScrapeCreators reaches the best Reddit threads of all time and their comments, YouTube transcripts and comments, TikTok, Instagram, X, LinkedIn, competitors' ads and creators' audiences by age and gender. It is paid per request, on your own key." |
| Firecrawl | "Firecrawl searches the web and reads whole pages and sites, such as competitors' pricing. It spends your own credits after a free allowance." |
| Treg | "Treg answers what no page shows: search volumes, search results, company and people data, ad libraries. Each call spends your own Treg credit." |
| last30days | "last30days shows what people are saying now on Reddit, X, YouTube, TikTok, Instagram and Hacker News, freshest first." |

Close with three lines: with them, canvas entries carry quoted, linked citations and can be `Reported`
instead of `Inferred`; "Research still works without any of them, through my own web search and
fetch: they only add depth, such as quoted posts, transcripts, comments and counts."; "Mentat has no
affiliation with any of these tools or their providers: they are third-party options you may choose
to install and pay for." Then ask whether to set up the missing ones, all, some, or not now.

**Setting up**, on yes, one tool at a time, saying what each command does before running it:

| Tool | Set up |
| --- | --- |
| agent-reach | `uv tool install https://github.com/Panniantong/Agent-Reach/releases/latest/download/agent-reach.zip`, then `agent-reach setup`, and show the installation report it prints. Never the package of the same name on the Python package index: it is not this tool. Needs uv |
| ScrapeCreators | The person makes an account at scrapecreators.com and puts the key in their environment as `SCRAPECREATORS_API_KEY`, or as that line in `~/.config/last30days/.env`. They type it there themselves |
| Firecrawl | `pnpm add -g firecrawl-cli`, then `firecrawl config` to sign in |
| Treg | The person signs up at treg.to and adds its server, `https://treg.to/mcp/`, to this harness with their own key, as Treg's setup page says; the tools appear in the next session |
| last30days | On Claude Code: `claude plugin marketplace add mvanhorn/last30days-skill`, then `claude plugin install last30days@last30days-skill`. Elsewhere: as https://github.com/mvanhorn/last30days-skill says. Needs Python 3.12 or later |

yt-dlp, the program that fetches YouTube subtitles: agent-reach's setup brings its own. Offer
`pip install yt-dlp` only when last30days is set up and agent-reach is not.

**Not now.** Record the date for each tool offered, in `research-tools.json`: under
`%LOCALAPPDATA%\Mentat\` on Windows, the folder `mentat schedule` keeps its scripts in, and under
`~/.local/state/mentat/` on macOS and Linux. Name the tools in the list:

```
node -p "const fs=require('fs'),p=require('path'),os=require('os');const win=process.platform==='win32';const f=p.join(win?p.join(process.env.LOCALAPPDATA,'Mentat'):p.join(os.homedir(),'.local','state','mentat'),'research-tools.json');const s=fs.existsSync(f)?JSON.parse(fs.readFileSync(f,'utf8')):{notNow:{}};for(const t of ['agent-reach','scrapecreators','firecrawl','treg','last30days'])s.notNow[t]=new Date().toISOString().slice(0,10);fs.mkdirSync(p.dirname(f),{recursive:true});fs.writeFileSync(f,JSON.stringify(s,null,2));f"
```

The file holds nothing else. Which tools are here is never written down.
