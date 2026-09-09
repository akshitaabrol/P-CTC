# P-CTC evaluation set

The exact 6,444-utterance test set used in *P-CTC: Decoder-Free Multi-Task Phone
and Speech Recognition*, released so that the numbers in the paper can be
reproduced and so that new phone recognisers can be compared against them on
identical data.

Published phone-recognition results are hard to compare: papers use different
test splits, different language mixes, and different metric implementations. The
point of this release is to remove those three variables.

**This repository contains data only.** Training and evaluation code is not
released yet, and neither are model weights. We intend to release the trained
checkpoint separately.

## What is here

| file | contents |
|---|---|
| `test_manifest.jsonl` | 6,444 utterances: id, language, duration, reference text, reference phones |
| `languages.csv` | utterance and hour counts per language |

One JSON object per line:

```json
{"id": "10048737995934939579-196",
 "lang": "afr",
 "duration": 15.0,
 "text": "lodin het ook ...",
 "phones": "luədən hɛt uək ...",
 "phones_segmented": "l u ə d ə n | h ɛ t | u ə k | ..."}
```

**Read this before scoring.** The two phone fields are different things:

* `phones` is the IPA transcription with **words** separated by spaces.
  Splitting it on whitespace gives words, not phones.
* `phones_segmented` is the same transcription **segmented into phones** — one
  space between phones, `|` marking each word boundary. This is the field to
  score against.

The distinction is not cosmetic. Segmenting IPA by Unicode code point splits
affricates and diacritic-bearing segments into pieces, which inflates the token
count and produces error rates that cannot be compared with any paper that
segments properly. We therefore ship the segmentation rather than leaving it to
you: `phones_segmented` was produced by the same code path that scored every
number in the paper, using panphon's longest-match inventory — the same
inventory PFER is computed over, so segmentation and scoring cannot disagree.

The set holds **683,891 phone tokens** over **146 distinct phones**. The model's
inventory is 156 symbols, learned from the training split; the test set
exercises 146 of them. Segmenting the references drops 61 characters in total
(0.009%), across three symbols absent from panphon's inventory.

## Audio

Audio is **not** included, to avoid redistributing corpora under their own
licences. Every `id` is the identifier used in
[`anyspeech/ipapack_plus_2`](https://huggingface.co/datasets/anyspeech/ipapack_plus_2),
so the audio can be fetched from there and matched by id. The set is the FLEURS
portion of IPAPack++, filtered to utterances of 20 seconds or less.

## The set

12 languages, 19.71 hours, 6,444 utterances.

| code | language | utterances | hours |
|---|---|---:|---:|
| afr | Afrikaans | 191 | 0.59 |
| aze | Azerbaijani | 682 | 2.23 |
| bos | Bosnian | 730 | 2.37 |
| cmn | Mandarin Chinese | 637 | 1.91 |
| deu | German | 609 | 1.86 |
| eng | English | 518 | 1.34 |
| mkd | Macedonian | 753 | 2.29 |
| orm | Oromo | 37 | 0.11 |
| pan | Punjabi | 449 | 1.35 |
| slv | Slovenian | 669 | 1.72 |
| spa | Spanish | 727 | 2.34 |
| tgk | Tajik | 442 | 1.60 |

Oromo is small at 37 utterances and its per-language scores are correspondingly
noisy; we report it for completeness rather than as a result.

## Results on this set

Every system below was decoded by us on this manifest and scored with one metric
implementation. PER is phone error rate. PFER is phone feature error rate, which
weights each substitution by how different the two sounds are in articulatory
features. RTF is one phone-recognition decode per system at batch size 1 on an
otherwise idle H200.

| system | PER | PFER | RTF |
|---|---:|---:|---:|
| Allosaurus | 59.1 | 18.7 | 0.035 † |
| MultIPA | 53.1 | 14.9 | 0.0018 |
| Allophant | 61.0 | 14.4 | 0.0012 |
| Wav2Vec2Phoneme (xlsr-53) | 42.6 | 11.6 | 0.0015 ‡ |
| ZIPA-CR-NS-large | 29.9 | 7.2 | 0.0142 |
| POWSM | 12.4 | 4.9 | 0.0581 |
| **P-CTC (ours)** | **10.9** | **4.4** | **0.0018** |

† CPU only; Allosaurus exposes no GPU interface.
‡ Timed under an earlier protocol; to be re-measured.

Four things matter if you compare against these numbers:

* **POWSM is decoded greedily** (beam size 1), matching its own paper. ESPnet's
  wider default beam is slower and, in our measurements, slightly less accurate.
* **ZIPA's filterbank front end is inside its timer.** Leaving feature
  extraction out of an RTF makes a model look faster than it is.
* **RTF is one decode of one task.** POWSM consumes its task token in the
  decoder, so each of its four tasks costs a decode of its own; the single P-CTC
  pass timed here also produces the speech-recognition and language outputs.
* **Symbols outside the inventory are removed and counted, not scored**, so no
  system is charged an error for emitting a symbol the inventory has no segment
  for. This can only work in a baseline's favour, never in ours.

## Unseen languages

The paper also evaluates on 44 languages from
[DoReCo](https://doreco.huma-num.fr/) that appear in no training set. That
manifest is **not** included here: DoReCo's terms require each constituent
dataset to be credited to its creators, and redistribution is a separate
question from the FLEURS-derived set above. Follow the link to obtain it.

## Citing

The paper is under review; a citation will be added on acceptance. If you use
this evaluation set, please also cite the sources it derives from:

* **FLEURS** — Conneau et al., *FLEURS: Few-shot Learning Evaluation of
  Universal Representations of Speech*, SLT 2023.
* **IPAPack++ / ZIPA** — Zhu, Samir, Chodroff and Mortensen, *ZIPA: A Family of
  Efficient Models for Multilingual Phone Recognition*, ACL 2025.
* **PanPhon**, for the articulatory features PFER is computed over — Mortensen
  et al., COLING 2016.

## Licence

The manifest derives from FLEURS via IPAPack++ and remains subject to those
corpora's licences; consult them before redistributing. The per-language counts
and the results table in this README may be reused freely with attribution.
