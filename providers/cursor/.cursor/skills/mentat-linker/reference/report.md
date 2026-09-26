# The report

## In session — one line per link, handed to the calling skill

One line each, in the order typed, then the ones left alone:

```
Typed 2 links from this run:
  H-14 → CH-03: depends_on (0.91) — the reorder hypothesis cannot hold unless the email channel exists
  H-14 → CS.JOBS-02: answers (0.87) — the hypothesis answers the job "keep the bar stocked"
Left alone 1:
  KR-01 ~ KR-04: possible duplicate — "Rented warehouse in Otopeni" and "The Otopeni warehouse we rent"
```

The arrow shows the direction as typed; `~` an untyped pair. The number is the confidence the
embedder gave. The clause after the dash is your sentence, so the person can disagree with it in
one read. Report what `type_link` answered, by code.

## Nightly — the digest, for the summary `mentat-agent` writes at `end_run`

Counts first, then the lists:

```
Nightly linker: 14 links read, 11 typed, 2 possible duplicates left for you, 1 left untyped.
Words used: depends_on 5, answers 3, increases 2, stored_at 1 (new — "the first row is kept at the place the second names").
Typed:
  <one line per link, as above>
Possible duplicates (rule on the links screen):
  <one line per pair>
Left untyped:
  R-02 ~ CO-07: rejected by a person under decreases; no other word fits
```

After every page, `observe_run` with one line: "page 3: 25 read, 21 typed, 2 duplicates, 2 left".
The digest is what the person reads in the morning as the run's report; the lines in it are what
they will see on the links screen. Nothing else is sent: no message, no escalation.
