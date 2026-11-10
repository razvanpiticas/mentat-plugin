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

## Say what it will cost, before calling

Before the first paid call of a step, list the paid calls it will make and their rough cost, then
ask, and call nothing paid until the person says which. A step that says so in its own words, such as
`list-the-competitors`, writes the answer in its checkpoint.

- One line per call: the service, what it fetches, for which competitors or segments, how many
  requests, and the rough cost: Treg's `catalog_get` answers each call's price; ScrapeCreators is
  about one credit a request; Firecrawl spends credits after its free allowance. A total at the end,
  in money where the price says it, else in credits.
- In a session, the person chooses. STOP and call the question tool to clarify. Offer the full list, a smaller one, or
  none: research without paid calls is still research.
- Unattended, nobody can approve: call only the paid services the routine's instructions name, and
  leave the rest waiting, named in the summary with what they would cost.

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

yt-dlp is the program that fetches YouTube subtitles. agent-reach's setup brings its own; last30days
needs it on the path for YouTube.
