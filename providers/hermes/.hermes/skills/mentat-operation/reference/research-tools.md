# Research tools

Load this when a step needs desk research. It lists tools this machine may have: what each is good
for, how to tell it is here, and the shape of a call. They are options, never a required procedure.
Use them, other tools, or better techniques you know: whatever finds the fact the step asks for with
a citation anyone can check. A tool that is missing is no reason to skip the research.

Research works without any of them, through your own web search and fetch; they only add depth:
quoted posts, transcripts, comments, counts. Mentat has no affiliation with any of these tools or
their providers: they are third-party options the founder may choose to install and pay for.

ScrapeCreators, Treg and Firecrawl spend the founder's own credits. Spend in proportion to what the
step needs.

## Check what is here, and hand it on

Before the first research call of a run, run each tool's "Here when" check below, and look at this
session's own tools and skills for the rest. Say in the run what is here (`observe_run`, one line).
When the `mentat` skill already checked this session and passed the list on, use its list.

Every research subagent's brief names the tools that are here, with the shape of a call for each, and
the page ladder below, and the spend the person approved, word for word. Why: a subagent knows only
what its brief says. A brief that names only web search and fetch gets research done with web search
and fetch, and a page that refuses them is written as not found while Firecrawl and a browser sat
unused on the same machine.

## A page that refuses to be read

A page that answers 403 or 429, shows a captcha or a "verify you are human" wall, or comes back
empty because it draws itself with JavaScript, goes down this ladder, cheapest first, and stops at
the first rung that reads it:

1. **The free reads.** The harness's own fetch, then `agent-reach read <link>` when agent-reach is
   here. Both cost nothing.
2. **Firecrawl.** `firecrawl scrape <link> -o <file>`, then read the file. It renders JavaScript and
   gets past most blocks. It spends credits after its free allowance: name it in the paid list the
   person approves ("Firecrawl, for the pages the free reads cannot open, about one credit a page").
3. **A real browser**, the first that is here:
   - **The Claude in Chrome extension**, when this session has its tools and it answers (its tabs
     context call answers a tab list). It reads the page in the person's own Chrome, so open a new
     tab, read the public page, and touch nothing else: never sign in, post, buy or click past a
     consent the person has not given. It needs the person's browser open, so it is a rung of a
     session, not of a routine's run.
   - **playwright-cli**, its own headless browser:
     `playwright-cli open <link>`, then `playwright-cli snapshot --filename=<file>` or
     `playwright-cli eval "() => document.body.innerText"`, then `playwright-cli close`. It works in a
     routine's run too.
4. **An archived copy.** The Wayback Machine's latest snapshot, written as true on the snapshot's
   date, never as today's.

Only when every rung that is here has failed is the fact written as looked for and not found
(`Unknown`): its text names each rung tried and what it answered ("plain fetch 403, Firecrawl 403,
playwright-cli captcha, Wayback none since 2025"). A rung that is not here is no reason to stop:
- **In a session**, say once which rung is missing and what it would have read, and offer to set it
  up with the steps in the `mentat` skill's front-door reference. Ask the user directly to clarify what you cannot infer. On no, go on
  with the rungs that are here.
- **Unattended**, never install anything. Use the rungs that are here and name the missing one, and
  the pages it left unread, in the run's summary.

## Say what it will cost, before calling

Before the first paid call of a step, list the paid calls it will make and their rough cost, then
ask, and call nothing paid until the person says which. A step that says so in its own words, such as
`list-the-competitors`, writes the answer in its checkpoint.

- One line per call: the service, what it fetches, for which competitors or segments, how many
  requests, and the rough cost: Treg's `catalog_get` answers each call's price; ScrapeCreators is
  about one credit a request; Firecrawl spends credits after its free allowance. A total at the end,
  in money where the price says it, else in credits.
- In a session, the person chooses. Ask the user directly to clarify what you cannot infer. Offer the full list, a smaller one, or
  none: research without paid calls is still research.
- Unattended, nobody can approve: call only the paid services the routine's instructions name, and
  leave the rest waiting, named in the summary with what they would cost.
- **The spend approved is the only limit.** Never add a cap of your own, per call, per competitor or
  per anything, and never skip a call the approval covers because of one. Before calling, or before
  calling a fact too dear, read the other providers `catalog_get` ranks for the same job: the same
  lookup is often a fraction of the price elsewhere. A call that would take the total past what was
  approved is asked about in a session and left waiting unattended.

## What the findings become

Write what you find into the entries the step writes: the segment's ratings and comparison, the end
user profile's fields, the jobs, pains and gains. Each quote or figure is a citation on the statement
it backs, the entry or the fact inside a part: its `reference` (the link), `who` said it, their
`words` quoted exactly (transcripts included), `seenOn` (the day it was published or read), and its
`upvotes`, `comments` and `views` where the place shows them. Ten quotes behind a pain are ten
citations. The statement carries its status: `Reported` when an outside source says it, `Confirmed`
when the founder's own data shows it or you checked it first-hand, `Inferred` when reasoned from
other facts. Never park findings in a note entry. Why: the next operations and the gate read the
statements and what backs them; a finding left in a note never reaches them.

## The tools

**agent-reach** — reads Reddit, YouTube (search and subtitles), GitHub, X, Instagram, LinkedIn,
Facebook and the web. Read-only, no credits.
- Here when `agent-reach version` answers JSON with `"ok": true`. It installs into `~/.local/bin`,
  which may not be on the path: try that folder too.
- `agent-reach web search "<words>"`, `agent-reach reddit search "<words>"` (after
  `agent-reach add reddit`), `agent-reach youtube search "<words>"`,
  `agent-reach youtube subtitle <video link> --lang en`, `agent-reach github search "<words>"`,
  `agent-reach read <link>`; `agent-reach channels` lists the rest.
- Always pass `--lang en` to subtitles: without it, it asks for Chinese first and YouTube can pause
  the channel for about an hour. When Reddit answers blocked, `agent-reach login reddit` copies the
  person's Chrome login into its own browser: only when the person says so.

**ScrapeCreators** — evidence from all time, not only recent: top Reddit posts and their comments,
YouTube transcripts and comments, TikTok, Instagram, X, LinkedIn, Threads, Facebook, GitHub, the
advertising libraries of Meta, Google, LinkedIn and TikTok (what competitors advertise, at what
price), and Creator Tools (a creator's audience by age and gender). About one credit a request;
status 402 means the credit ran out.
- Here when `SCRAPECREATORS_API_KEY` is set, or is a line of `~/.config/last30days/.env`.
- `curl -s -H "x-api-key: $SCRAPECREATORS_API_KEY" "https://api.scrapecreators.com/v1/reddit/search?query=<words>&sort=top&timeframe=all"`.
  On Windows PowerShell, `curl.exe` and `$env:SCRAPECREATORS_API_KEY`. Every other call is in
  https://docs.scrapecreators.com. Read the key from where it is kept; never print it.

**Firecrawl** — web search with whole pages, one page as clean text, every page of a site (a
competitor's pricing, a directory). Credits after a free allowance.
- Here when `firecrawl --version` answers, or a `firecrawl` skill is listed in this session.
- `firecrawl search "<words>"`, `firecrawl scrape <link>`, `firecrawl map <site>`,
  `firecrawl crawl <site>`.

**Treg** — data no page shows: search volume for keywords, search results pages, links into a site
and its authority, social profiles and trends, people and company data, advertising libraries. About
3,600 calls from 91 providers behind one server. Each call spends the founder's Treg credit.
- Here when the session has the `treg` tools.
- `catalog_search` with what you want done, then `catalog_get` on the answer it picks (its
  parameters, its price, how reliable it is), then `call`.

**last30days** — what people are saying now: recent discussion on Reddit, X, YouTube, TikTok,
Instagram, Hacker News, Polymarket, GitHub and the web, freshest first. It uses the ScrapeCreators key
when there is one, which matters when Reddit blocks direct reading.
- Here when a `last30days` skill is listed in this session.
- Run that skill with the topic. `--days 180` looks further back, `--deep` finds more,
  `--search youtube` keeps to one source, `--subreddits=<a>,<b>` names the communities, and
  `--plan '<json>'` passes your own plan of searches.

**The Claude in Chrome extension** — a real Chrome, the person's own, for pages nothing else can
read. Free. Claude Code only.
- Here when this session has the `claude-in-chrome` tools and its tabs context call answers.
- Load its tools, open a new tab on the link, read the page; never reuse the person's own tabs.

**playwright-cli** — a headless browser driven from the shell, for pages the free reads and
Firecrawl cannot open. Free, any harness, routines included.
- Here when `playwright-cli --version` answers.
- `playwright-cli open <link>`, `playwright-cli snapshot --filename=<file>`,
  `playwright-cli eval "() => document.body.innerText"`, `playwright-cli close`;
  `playwright-cli --help` lists the rest.

yt-dlp is the program that fetches YouTube subtitles. agent-reach's setup brings its own; last30days
needs it on the path for YouTube.
