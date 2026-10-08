# ParlaTO
[![CC BY-NC-SA 4.0][cc-by-nc-sa-shield]][cc-by-nc-sa]

- [ParlaTO](#parlato)
	- [Repository organization](#repository-organization)
	- [Metadata](#metadata)
	- [Verticalized content](#verticalized-content)
	- [Data access](#data-access)
	- [How to cite](#how-to-cite)
	- [Changelog](#changelog)

The ParlaTO corpus is part of the larger [KIParla collection](https://www.kiparla.it),
which can be freely queried through the [NoSketch Engine interface](https://kiparla.it/search/).

The ParlaTO corpus was was funded by the CRT Foundation ("ParlaTO - Corpus del Parlato di Torino" project).

It consists of about 50 hours of interactions collected in Turin and its province through semi-structured interviews. The interviews, conducted between 2018 and 2020, involved 88 speakers with different origins, ages, education levels, and types of occupation, and addressed personal life experiences in the city (study, work, leisure activities, retirement, memories of the past, etc.).

The transcriptions have been anonymized.

Overall, the module is made up of 68 conversations and includes 100 speakers.

## Repository organization

This repository contains:

* metadata for both speakers and conversations, in the [`metadata`](./metadata/) subfolder (see [metadata](#metadata) section below)
* descriptions of the set of transcription conventions used for this module ([Transcription conventions](./jefferson-notation.md))
* statistical summaries of each conversation (token counts, speaking time per speaker, token/time rates, overlaps) are not in this repository: they are in the folder of this module in [KIParla-summaries](https://github.com/KIParla/KIParla-summaries)

For each conversation you will find:

* `.eaf` file in [`eaf/`](./eaf/) folder: time-aligned Jefferson-style transcriptions (open with [ELAN](https://archive.mpi.nl/tla/elan)). These files are regenerated from the verticalized `tsv/` files, so they contain the normalized transcription (see [Verticalized content](#verticalized-content)); the audio is linked by relative path (`<code>.mp3`).
* `.txt` file in [`linear-jefferson/`](./linear-jefferson/) folder: linearized Jefferson-style transcription.
* `.txt` file in [`linear-orthographic/`](./linear-orthographic/) folder: linearized transcription retaining only orthographic words.
* `.tsv` file in [`tsv/`](./tsv/) folder: verticalized version of the transcription, with Jefferson-style information decoupled from the text as features See [verticalized-content](#verticalized-content) for more information.

Linear files in [`linear-jefferson/`](./linear-jefferson/) and [`linear-orthographic/`](./linear-orthographic/) contain one Transcription Unit (TU) per line. Each line has two columns: the first is the speaker code, and the second is the transcription. TUs are sorted by their start time.

## Metadata

Each participant and each conversation are associated to a series of metadata, that can be found in the
[`metadata/participants.tsv`](metadata/participants.tsv) and [`metadata/conversations.tsv`](metadata/conversations.tsv) fils.
Metadata is to be interpreted as follows:

1. Participants metadata:
	- `code`: unique anonymized 5-char identifier for each participant. Unknown, occasional participants
	 to conversations are associated with a special `???` code.
	- `gender`: either `M` for masculine or `F` for feminine
	- `age-range`: 5 years range including the participant’s age.
	- `birth-region`: Italian region[^1] where the participant was born. If outside Italy, the label `estero` is used.
	- `occupation`: occupation of the participant, according to [ISTAT categories](https://professioni.istat.it/). For more information see [occupation label](./occupation-levels.md)
	- `study-level`: highest completed level of education[^3]
	- Additionally, the [`metadata/participants.tsv`](metadata/participants.tsv) also contains a `conversations` colum that summarizes the conversations in which the participant appears.

2. Conversations metadata:
   - `code`: unique identifier for conversation
   - `type`: type of interaction, for this module all conversation are `semistructured-interview`
   - `duration`: duration of the conversation, expressed in `hh:mm:ss` format
   - `participants-number`: number of participants in the conversation
   - `languages`: languages spoken in the conversation, can be either `italian` or `dialect`, or both.
   - `participants-relationship`: relation, either symmetrid or asymmetric
   - `moderator`: presence of a moderator
   - `topic`: fixed for this module
   - `year`: year of collection
   - `collection-point`: two-letter code of the collection area: `TO` for Turin for this module.
   - Additionally, the [`metadata/conversations.tsv`](metadata/conversations.tsv) also contains a `participants` field that recaps the codes of the participants to that conversation

[^1]: `abruzzo`, `basilicata`, `calabria`, `campania`, `emilia-romagna`, `friuli-venezia-giulia`, `lazio`, `liguria`, `lombardia`, `marche`, `molise`, `piemonte`, `puglia`, `sardegna`, `sicilia`, `toscana`, `trentino-alto-adige`, `umbria`, `valle-d-aosta`, `veneto`
[^3]: `elementary-school`, `liceo-diploma`, `middle-school`, `phd`, `technical-vocational-diploma`, `university-degree`, `university-degree-ongoing`

## CSVW metadata

The tabular files of this module are described by a [CSV on the Web (CSVW)](https://www.w3.org/TR/tabular-data-primer/) metadata document, [`csv-metadata.json`](./csv-metadata.json), placed at the root of the repository. It documents `metadata/conversations.tsv`, `metadata/participants.tsv` and every `tsv/<code>.vert.tsv` file, listing for each column its name, datatype (and format, where relevant) and a short description, together with the foreign key linking the `participants` column of the conversations table to the participants table. The file is generated from the shared schemas in the KIParla `tools` repository (`tools/csvw/generate_csvw.py`) and should not be edited by hand. CSVW-aware tools (e.g. the `csvw` Python package, `csvlint`, or R's `csvwr`) can use it to validate the data and load it with the correct types.

## Verticalized content

Conversations are also available in a vertical, pseudo-tokenized version in [`tsv/`](./tsv/).
Tokenization is obtained by validating the Jefferson transcription using custom [tools](https://github.com/LaboratorioSperimentale/kiparla-tools)and splitting on token boundaries: whitespaces, prosodic links (`=`), and apostrophes used for elision in Italian orthography. Each transcription-derived token is then documented on one row.

Each token is represented as 20 columns, as follows:

1. `token_id`: unique token identifier within the conversation (`<tu_id>-<token_index>`)
2. `speaker`: speaker `code` as it can be found in [`metadata/participants.tsv`](metadata/participants.tsv)
3. `tu_id`: progressive identifier assigned to transcription units
4. `id`: token index within the transcription unit (0-based)
5. `span`: portion of the original jefferson transcription containing the token, including any variation marker that precedes it (`#`, `#*`, `$`, a unit-initial `# ` or `#_ `, or a `#_ ` later in the unit), so the original text can be rebuilt from this column
6. `form`: orthographic form of the token. This differs from the `span` as special symbols are stripped out and represented as `jefferson_feats`. Moreover, shortpauses (`(.)` in the transcription) are represented as `[PAUSE]` and unintelligible tokens (sequences of `x` in the transcription) are represented as `x`
7. `lemma`: reserved for a future lemmatization step; `_` for now
8. `upos`: reserved for a future POS-tagging step; `_` for now
9. `xpos`: reserved; `_` for now
10. `feats`: reserved; `_` for now
11. `deprel`: reserved for a future dependency-parsing step; `_` for now
12. `type`: one of
    - `linguistic`: everything that is considered to be a content linguistic token
    - `nonverbalbehavior`: used for transcribed non verbal behaviors, such as laughing or sighing
    - `shortpause`: identifies pauses
    - `unknown`: identifies unintelligible spans in transcription
    - `error`: residual class to mark cases where the transcription is not well formed according to Jefferson format. Therefore, the token is not analyzed and transcription will be corrected in future releases.
13. `meta_label`: reserved; `_` for now
14. `variation`: all variation features of the token, as pipe-separated `Key=Value` pairs (never empty, as the first one is always present):
    - `ContainsVariation`: unit-level flag, repeated on every token of the unit. `Yes` if any token of the unit has a `Variety`, `No` otherwise
    - `Variety`: token-level variety other than standard Italian. Values:
      - `Other`: the token is in another language or variety — a `#word`, or any token after a `#_` marker (which covers the rest of the transcription unit, or the whole unit when it is unit-initial)
      - `Unassignable`: the variety of the token cannot be assigned (`#*word`, probably Italian)
      - `Unsure`: the token is in a unit that starts with `# `, where the transcriber marked that another variety is present without saying on which words
    - `Language=<ISO code>`: present on tokens marked as another variety (or `NO_ISO_CODE` when the specific variety wasn't identified)
    - `Nonce=Yes`: nonce or non-standard form of the token, marked with `$` in the transcription (not a variety)

    Example: `ContainsVariation=Yes|Variety=Other|Language=NO_ISO_CODE`
15. `jefferson_feats`: pipe-separated list of word-level features derived from the transcription in Jefferson format. More specifically:
    - `SpaceAfter=No`: no whitespace between this token and the next (e.g., `l'` in `l'anno`)
    - `ProsodicLink=Yes`: a prosodic link (`=`) to the following token
    - `Intonation` can assume values `Falling`, `Rising` or `WeaklyRising` and translates word final punctuation sign in Jefferson transcriptions (i.e., `.`, `?` and `,` respectively)
    - `Interrupted=Yes`: words interrupted in speech, transcribed with final `~` or `-`
    - `Truncated=Yes`: truncated forms (e.g., `anda'` for `andare`, common in some Italian varieties)
    - `Volume` can assume values `High` or `Low` and translates Jefferson's uppercase and `°` respectively
    - `Syllables=N`: number of syllables for `unknown` tokens (sequences of `x`)
16. `align`: alignment features for the first and last token of each TU, through `Begin=` and `End=` features expressed as seconds
17. `prolongations`: positions of sound prolongations (colons `:`) within the word, encoded as a comma-separated list of `<char_id>x<count>` pairs
	- `char_id` is the zero-based index of the character in the token's orthographic form
	- `count` is the number of consecutive colons immediately following that character in the original span
	- example: for span = `ese::mpio:`, its orthographic form is `esempio` and the prolongations field would assume value `2x2,6x1` (the 3rd letter `e` has 2 colons; the 7th letter `o` has 1 colon)
18. `pace`: marks whether the token participates in a fast or slow paced span within the word
	- Format: `Fast=<char_id_start>-<char_id_end>` or `Slow=<char_id_start>-<char_id_end>`
	- Indices are zero-based, inclusive, and refer to character positions in `form`
19. `guesses`: character span(s) transcribed as uncertain (i.e., in round brackets in the Jefferson transcription)
	- Format: `<char_id_start>-<char_id_end>(<guess_id>)` (zero-based, inclusive, over `form`)
20. `overlaps`: comma-separated list of character spans participating in simultaneous speech, with an overlap group identifier
	- Format: `<char_id_start>-<char_id_end>(<overlap_id>)`, where indices are zero-based, inclusive indices over `form` and `overlap_id` is the progressive number of the overlapping group within the TU
	- Examples: the span `e[se]mp[i` would be encoded as `1-3(2),5-6(3)` meaning that characters from position one (inclusive) to three (exclusive) participate to span number 2 while the last character (with id 5) participates to the third overlapping span of the transcription unit. When the overlapping id was not decidable, a `?` is used

## Data access

Due to GDPR restrictions, pseudo-anonymized audio files (MP3) are available under a restricted-access license. To request access, please contact the corpus coordinators through the KIParla website and follow the provided procedure.

## How to cite

To cite this module please include

> Cerruti, M., & Ballarè, S. (2020). ParlaTO: corpus del parlato di Torino. _Bollettino dell’Atlante Linguistico Italiano (BALI)_, 44, 171–196

in your references

```bibtex
@article{Cerruti_ParlaTO_corpus_del_2020,
	author = {Cerruti, Massimo and Ballarè, Silvia},
	journal = {Bollettino dell’Atlante Linguistico Italiano (BALI)},
	pages = {171--196},
	title = {{ParlaTO: corpus del parlato di Torino}},
	volume = {44},
	year = {2020}
}
```

If you use the ParlaBO module in your research, please also reference this repository (commit/tag) in your data statement or appendix.

## Changelog

* 2025-10-07 v1.0.0
  * First release

* 2025-11-28 v1.1.0
  * Major fix: wrong speaker attribution in linear-jefferson and linear-orthographic
  * Minor fix: empty turns in linear-orthographic were removed

* YYYY-MM-DD v2.0.0
  * Breaking: the `variation` column of the `tsv/` files is now a feature list holding all variation features of a token: `ContainsVariation=Yes|No` (unit level), `Variety=Other|Unassignable|Unsure`, `Language=<ISO code>` and `Nonce=Yes`. It replaces the unit-level label (`none`/`some`/`unspecified`/`all`); `Language` and the `Variation=`/`Orthography=` features moved out of `jefferson_feats`
  * Breaking: the unit-initial `# ` / `#_ ` marker is now part of the `span` of the unit's first token, so the original transcription can be rebuilt from the `span` column
  * Breaking: the `unit` column was removed from the `tsv/` files (they now have 20 columns)
  * Normalization: spelling variants of interjections and discourse markers are standardized (e.g. `m` → `mh`, `mhm` and `mmh` → `mhmh`, `he` → `eh`, `va beh` → `vabbè`) and elision typos are corrected (e.g. `dell~` → `dell'`)
  * All files are UTF-8 (no BOM), LF line endings and Unicode NFC; `eaf/` is regenerated from `tsv/` (audio linked by relative path)
  * The 20 columns of the `tsv/` files are now fully documented, and `csv-metadata.json` and the Jefferson notation reference are updated

-----

This work is licensed under a
[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License][cc-by-nc-sa].

<!-- [![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa] -->

[cc-by-nc-sa]: http://creativecommons.org/licenses/by-nc-sa/4.0/
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png
[cc-by-nc-sa-shield]: https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg