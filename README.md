*Arab artistic heritage put to the test of digital recognition models*

# Beyond the Single Label

Zero-shot evaluation of CLIP, SigLIP and XLM-R CLIP on Turath-Art (Kiyasseh & El-Bouri, 2022), and on an independent corpus of 5,158 works built for this project from three Arab art institutions.

Master's thesis, DUSDA7, Université Paris 1 Panthéon-Sorbonne, 2026.

---

## The question

Turath-150K showed in 2022 that vision models fail on Arab artistic heritage. Four years later, one question remains open. Does that gap come from the way we query these models, or from what they were never shown?

Turath-Art was designed for a supervised classifier, with a single label: the artist's name. Since then, zero-shot has become a common mode of use. Which raises a question the original protocol could not ask. Is a poor label really a disadvantage for these models? And if so, would enriching it be enough?

I tested both hypotheses. The answer closes the path that looked cheapest, the one that runs through better descriptions and better metadata.

---

## Results

### Author attribution plateaus around 2.5% Top-1

425-way classification, 12,750 images. Ten times chance. One tenth of the supervised baseline.

| | Top-1 (%) |
|---|---|
| Chance (1/425) | 0.24 |
| Supervised CNN, 2022 baseline | 16.5 |
| CLIP ViT-B/32 | 2.45 |
| SigLIP base | 2.71 |
| XLM-R CLIP | 2.27 |

The comparison with the supervised baseline is not paired. It is a reference point, not an equality of conditions.

### Enriching the prompt does not improve performance

The candidate label moves from `"a painting by {artist}"` to a text carrying biography, medium, dimensions and movement. On the 235 artists shared with the independent corpus, 7,050 images.

| | Generic | Enriched | Δ | p |
|---|---|---|---|---|
| CLIP | 2.14 | 1.69 | −0.45 | 0.052 |
| SigLIP | 3.28 | 2.50 | −0.78 | **0.004** |
| XLM-R CLIP | 2.10 | 2.51 | +0.41 | 0.124 |

One technical objection, worth stating upfront: CLIP truncates at 77 tokens and degrades on long text. Part of the effect may come from length rather than content.

Top-5 moves the other way. SigLIP goes from 5.28 to 5.41, XLM-R CLIP from 5.07 to 5.88. The right artist often climbs into the top five without settling at the top. The added context seems to improve global ranking without carrying enough discriminative signal to separate close candidates.

**More text does not mean better performance.**

### Query language changes nothing measurable

EN 2.27%, FR 2.61%, AR 2.55%, with XLM-R CLIP. No pairwise difference is significant.

This is not evidence that language has no effect. At this performance level the test sits near the floor, and a floor cannot reveal a gap. The exact result is: no detectable difference here.

### Retrieval works two orders of magnitude better

Different task. Not attributing an author, but finding which caption matches which image among 5,011 pairs from the independent corpus.

| | Image→text | Text→image |
|---|---|---|
| CLIP | 5.99 | 4.91 |
| SigLIP | 5.59 | 4.43 |
| XLM-R CLIP | **7.96** | **6.49** |

Chance sits at 0.02%. So 7.96% is roughly 400 times chance, against 10 times for classification. A different order of magnitude. Significant in both directions, p < 0.001.

91% of the corpus text is in English, verified by language detection, which is why retrieval is evaluated in English only.

**These models are not blind to this heritage. They fail at one precise task, author attribution, which is exactly the task Turath-Art imposes through the structure of its labels.**

### Documentation coverage: a signal, not a proof

If the deficit comes from what the models were never shown, artists well documented on the web should be better recognised. 43.5% of the 425 artists have no Wikidata entry at all.

Documented artists reach 2.67% Top-1 against 2.00% for the others, in the same direction across all four tested conditions. The protocol samples exactly 30 images per artist in both groups, so the gap cannot come from uneven representation inside the dataset.

At the artist level, which is the correct unit of observation since 30 images share one artist, the gap does not reach significance. With 425 artists, it cannot.

---

## The independent corpus

Turath-Art inherits the curatorial scope of one private collection, the Barjeel Art Foundation. To test what that scope is worth, this project builds a second corpus from three institutions with divergent editorial logics.

| Source | Works | Text |
|---|---|---|
| Mathqaf, Doha | 3,059 | Instagram captions |
| Dalloul Art Foundation, Beirut | 1,999 | Title and medium |
| Kamel Lazaar Foundation, Tunis | 100 | Title and biography |

5,158 works, 1,017 distinct artists after name normalisation, against 1,082 before merging duplicates.

**Only 55.3% of artists are shared between the two corpora.** 235 matches out of 425, 214 of them exact. Almost one artist in two is not shared. Nothing suggests a stable canon of modern Arab art exists, which means there is no ground truth here the way an object benchmark has one.

Text length differs sharply by source. Median 34 characters for the DAF, 132 for Mathqaf, 1,023 for the KLF. Any per-source comparison of retrieval scores is confounded by this, and none is reported as a result.

---

## Repository

```
notebooks/
  01_turath_zeroshot_eval.ipynb     classification, trilingual test, Wikidata analysis
  02_enrichment.ipynb               generic against enriched prompts
  03_retrieval_eval.ipynb           image-text retrieval on the independent corpus
  04_qualitative_case_study.ipynb   single-example appendix
  collection/                       Mathqaf merge and image download, DAF image download
data/
  merged_arab_art_dataset.csv       5,158 works, full metadata
  dalloul_art_foundation.csv        DAF catalogue as scraped
  artist_overlap_matches.csv        the 235 matches behind the 55.3% figure
results/
figures/
```

Metadata is included, images are not. Running the evaluations means re-downloading images from the `image_url` column, then pointing `local_image_path` at your own copies.

Models load through `open_clip` and `transformers`. Every comparison uses bootstrap confidence intervals and permutation tests at 2,000 resamples.

### Access and reuse

Images are not redistributed. Mathqaf material was collected with the explicit agreement of the platform's co-founder. The DAF catalogue was harvested for academic research under the text and data mining exception; images were downloaded locally for feature extraction only, and that download script is not included here. The Kamel Lazaar Foundation collection is openly accessible.

Turath-Art itself is not redistributed. See Kiyasseh & El-Bouri (2022) for access.

---

## Limits

The documentation test is underpowered. 425 artists at 30 images each cannot establish the effect, whichever way it points.

The independent corpus is itself constructed. It aggregates three editorial logics that were never harmonised, and there is no neutral reference to appeal to.

Only 98 records out of 5,158 carry a biography, so the specific contribution of narrative text against formal description stays untested.

Every image is resized to 224 pixels. A three-metre canvas and a twenty-centimetre drawing are treated at the same scale, and brushwork disappears.

---

## Where this points

Raising the number of images per artist comes first. Only 12,750 of the 105,539 available images are used here, and this is what would settle the documentation question.

Two-stage re-ranking, generic retrieval first and then reordering candidates with the enriched text, exploits the Top-1 against Top-5 gap directly.

Targeted fine-tuning with hard negative mining, drawing negatives from culturally proximate artists rather than retraining generically over 425 classes.

And the negative result is the one that matters. Text-level intervention does not correct cultural under-representation. Curation has to act on the composition of corpora, not on prompt engineering.

---

## Reference

Kiyasseh, D. & El-Bouri, R. (2022). *Turath-150K: Image Database of Arab Heritage.*

---

## En français

Ce dépôt accompagne un mémoire de Master soutenu à l'Université Paris 1 Panthéon-Sorbonne en 2026, « Au-delà du label unique : audit, curation et paradoxes de l'enrichissement sémantique dans les espaces multimodaux du patrimoine artistique arabe ».

Mémoire disponible sur demande.

---

Omar Zeroual
