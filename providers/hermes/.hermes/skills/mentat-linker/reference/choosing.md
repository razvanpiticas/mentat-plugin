# Choosing the word

Load this at step 5 of the loop. The words come from the page's `words`; this file is how to pick
one, not a list of them.

## The test

Put the two ends into one sentence that a word's meaning describes, with the titles in it:

- "*The rented warehouse in Otopeni* cannot hold unless *the credit line with Banca X* holds." —
  that is the sentence `depends_on`'s meaning describes, from the dependent to what it depends on.
- "*The self-service channel* makes *the load on customer relationships* smaller." — `decreases`,
  from the cause to what it lowers; a magnitude when the text says how much.
- "*The second supplier* reduces the risk named by *R-03 single supplier*." — `mitigates`, from the
  mitigation to the risk.
- "*The interview evidence E-07* answers *Q-012 do gift buyers reorder*." — `answers`, from the
  answer to the question.

The meaning is read whole, not only its verb. Where it says what the first row or the second is,
that end must be one, or the word does not fit, however well the rest of the sentence reads; a
nearest fit is no fit. Go on to the next word, then to a new word, then to leaving the link untyped.

One sentence, one word. When two words both fit, the narrower one wins (`mitigates` over
`decreases` when the second row is a risk; `answers` over `related` when the second row is a
question). When no sentence comes after reading both ends whole, the word is `related` and the
link is already that: leave it untyped and report it as "read both, no relation beyond likeness".

## Direction

Every meaning names a first row and a second: "the first row cannot hold unless the second holds",
"more of the first means more of the second". Decide which of the two ends is the first in your
sentence. The page lists the pair as `from` and `to`:

| Your sentence's first row is | Send |
| --- | --- |
| the listed `from` | `FromTo` |
| the listed `to` | `ToFrom` |
| — the word is undirected (`isDirected` false) | `FromTo` |

`ToFrom` on an undirected word is refused; there is no direction to turn.

## Magnitude

Only on a word whose meaning says it may carry one, and only when the text states the size:
"cuts delivery cost by 15%" → `magnitude` 15, `unit` %. A guess is not a magnitude. Both fields or
neither.

## A new word

Only when no word on the page makes a sentence, after reading both ends whole. Then send the code
(lower case, letters, digits, underscores: `stored_at`) with `newWord`:

- `name`: what a person reads, "Stored at";
- `description`: one sentence of meaning naming the first row and the second, one of direction,
  one example — the shape of every shipped word's description on the page: "The first row is kept
  at the place the second row names. Directed, from the thing to the place. A pallet stored at the
  Otopeni warehouse.";
- `isDirected`: whether A→B differs from B→A.

The word exists for the organisation from then on; say so in the report. A code the product
ships is refused: use the shipped word.

## The duplicate flag

`isPossibleDuplicate` true: the two rows may be one thing written twice. Not typed, not merged, not
retired — reported, with both codes, so the person rules on the links screen.

## A rejected pair

A `CONFLICT` saying a person rejected this pair under this word is their verdict on that guess.
Type the pair with a different word only when that word passes the test on its own; never as a
way past the refusal. Otherwise leave it and report "rejected by a person under `<word>`; no other
word fits".
