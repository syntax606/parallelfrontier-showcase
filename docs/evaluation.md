# Evaluation

Every headline number comes from a random sample or a held-out set, never from examples picked by hand.

## Scope and funding: blind random sample

- **Sample:** papers drawn at random (fixed seeds) from all 13,324 papers listed in August 2026, in two draws of 100
  and 300 with no overlap. 366 of them could be labelled and are scored below.
- **Labelling:** each paper was labelled by reading its PDF (author block, footnotes, acknowledgements) on a sheet
  that showed nothing the pipeline had decided. This round was labelled by a separate model pass. A human re-labels
  a random subset, and the agreement rate will be reported here.
- **Scoring:** precision and recall with Wilson 95% intervals.

| Question | Precision | Recall |
|---|---|---|
| Any author affiliation in mainland China or Hong Kong | 99% (116/117; 95–100) | 97.5% (116/119; 93–99) |
| The paper has a funding statement | 92% (125/136) | 94% (125/133) |
| A Chinese funder supports the paper | 100% (29/29; 88–100) | 76% (29/38; 61–87) |

**Where it fails, honestly:**
- Funder names are fully right for only about half of funding mentions. The name is found, but its boundaries are
  often too long or too short.
- Chinese-funder recall depends on how much of the funder registry has been labelled. The 220 most frequent names are
  labelled so far, out of about 5,800.
- The three affiliation misses are two papers whose HTML version loses the author block and one Chinese state
  research programme named without its location.

## Names: held-out people

- **Data:** Wikidata people with both an English and a Chinese name, from mainland China, Taiwan, Hong Kong,
  Singapore and Malaysia (5,775 in the test half).
- **Split:** 50/50 by a hash of each person's ID. The dialect table was learned only from the build half, and the
  test half was never inspected.
- **Pairs:**
  - Same person: a person's English name against their own characters.
  - Different person: the English name against another person's characters with the same surname.

Results are in [names](names.md).

## Identity matching

- **Calibration:** weights come from pairs of records that share an email address (the same person), set against
  random pairs.
- **Human review:** in a review of 30 pairs the matcher had left unresolved, 20 were the same person, 10 were unclear
  and none were different people.
- **Why caution:** the grading (confirmed, probable, unresolved) is deliberately conservative. A false merge is worse
  than a missed one when the output feeds analysis of real people.

## Engineering checks

- **Tests:** 92 automated tests on invented papers, people and organisations, run on every push against Python 3.10
  and 3.13.
- **Privacy:** no real researcher appears in any test or in this repository.
