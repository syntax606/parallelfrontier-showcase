# pf-names: Chinese names, done properly

## The problem

A Chinese name reaches an English-language database through a lossy pipeline. 王伟 and 王薇 both become "Wang Wei".
One person can appear as 张家欣, "Jiaxin Zhang", "Chia-Hsin Chang" (Taiwan, Wade-Giles), "Cheung Ka Yan" (Hong Kong,
Cantonese) or "Teo Kah Hin" (Singapore, Teochew). Surname order flips, and English given names get added ("Jasper
Lee Po Lung"). Tools that treat names as strings either merge different people or split one person into many.

## The approach

Every name is parsed into all the readings it could have, and each comparison returns three things:

- **a level**: identical characters, a reading that fits, initials that fit, uncertain, partial, or a conflict;
- **a weight of evidence** (a log-likelihood ratio) that the identity matcher can add to other evidence;
- **a reason** a reviewer can read.

Three rules keep it honest:

1. **A match counts in proportion to how rare the name is.** "Wang Wei" matching "Wang Wei" is weak evidence; a rare
   name is strong evidence.
2. **A mismatch counts in proportion to how reliable both forms are.** Two pinyin forms that differ are strong
   evidence against a match. A mismatch through an approximate romanisation is weak.
3. **A spelling the engine can't read is never a mismatch.** It is "uncertain", which means absence of evidence.

## Romanisation systems covered

| System | Where | Example |
|---|---|---|
| Hanyu Pinyin | mainland China | 张家欣 → Zhang Jiaxin |
| Wade-Giles | Taiwan, older literature | 张家欣 → Chang Chia-Hsin |
| Cantonese (HK government style) | Hong Kong, Macau | 鄧樹榮 → Tang Shu-wing |
| Hokkien, Teochew, Hakka | Singapore, Malaysia | 陈文庆 → Tan Boon Keng |

- **Cantonese:** readings come from the Unicode Han Database (Jyutping for 30,096 characters). They are converted to
  the Hong Kong government spellings and their common variants: dang6 → Tang, zoeng1 → Cheung / Tseung,
  jyun4 → Yuen.
- **Dialects:** a table of Singapore and Malaysia spellings was learned from people with both an English and a Chinese
  name in Wikidata, then reviewed row by row.

## Results on held-out people

Wikidata people with both an English and a Chinese name. The dialect table was learned from half of them and
everything below is measured on the **other half**.

| | China | Taiwan | Hong Kong | Singapore | Malaysia |
|---|---|---|---|---|---|
| People tested | 1,932 | 1,954 | 612 | 455 | 822 |
| Same person matched | 82% | 56% | 37% | 25% | 14% |
| Same person wrongly ruled out | 2.1% | 5.0% | 3.9% | 4.6% | 5.0% |
| Different person, same surname, wrongly matched | 0.1% | 0.1% | 0.3% | 0% | 0% |
| Separation from name alone (AUC) | 0.98 | 0.86 | 0.80 | 0.75 | 0.67 |

Two additions moved these numbers:
- **Dialect table:** Singapore and Malaysia same-person pairs wrongly ruled out fell from 12.5% and 15.2% to 5% or
  less.
- **Cantonese readings:** Hong Kong matches rose from 13% to 37%, and separation from 0.72 to 0.80.

**What the numbers mean.** For mainland names, the name alone separates people well. In Hong Kong, Singapore and
Malaysia, many people publish under an English given name plus a surname, so there's no Chinese given name to check.
There the engine's job is to avoid ruling out the right person, and other evidence (email, co-authors, institution)
decides.

## Known limits

- Stage names, pen names, married names and Malay transliterations are not recognised.
- The test set is Wikidata, which skews towards public figures and older generations.
- The dialect table was reviewed by a non-native speaker of those dialects.
