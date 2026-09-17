---
title: "Unicode carriers — the other invisible layer that survives copy-paste"
description: "Marks made of bytes, not statistics: zero-width characters, bidi controls, tag characters, exotic spaces and homoglyphs. What they are, how they survive, how they are found and how they are removed without breaking legitimate text."
date: "2026-09-17"
tags: ["technology", "privacy", "research", "unicode"]
---

# Unicode carriers — the other invisible layer

> *Not every invisible mark is statistical. Some marks are not woven into the randomness of generation but written into the bytes of the text — the same characters, plus others that no one sees. Copy-paste carries both.*

This document covers the second family of marks that survive copy-paste, complementary to the statistical watermark of the rest of the dossier. The distinction matters for mitigation: a statistical mark can only be *degraded* (see [common mitigation](mitigation.md)), while a Unicode carrier is a byte and can be **removed exactly** — with the caveats in section 5.

## 1. Why they exist

Unicode has code points that occupy no visible width, change no glyph, or are unassigned/serving only as controls. Any of them can be inserted into text without altering what a reader sees. A sender (a platform, a service, a script) can encode an identifier or a payload in them while the text stays clean on screen and survives copy-paste, because they are ordinary characters of the string.

The `watermarks-remover` project formalizes this as *Layer A*: "invisible Unicode, exotic spaces, bidi, tag chars" — a layer **separate** from statistical watermarks (*Layer B*, which it treats as "best-effort"). It is not a theoretical class: it is the one a deterministic cleaner can handle.

## 2. The families of carriers

| Family | Examples | Notes |
|--------|----------|-------|
| **Zero-width** | U+200B zero width space, U+2060 word joiner, U+FEFF zero width no-break space (BOM) | No glyph, no width. The classic binary carrier: present/absent encodes 0/1 |
| **Joiners** | U+200C ZWNJ, U+200D ZWJ | **Legitimate**: required in Persian/Arabic/Indic orthography and in emoji sequences (`👨‍👩‍👧` is three emoji joined by ZWJ). Removing them changes real text |
| **Soft hyphen** | U+00AD | Invisible unless a line break happens there. Legitimate in typography and dictionaries |
| **Bidi controls** | U+202A–U+202E (LRE, RLE, PDF, LRO, RLO), U+2066–U+2069 (LRI, RLI, FSI, PDI) | **Legitimate**: needed to mix Arabic/Hebrew with Latin text. Also used for spoofing |
| **Variation selectors** | U+FE00–U+FE0F, U+E0100–U+E01EF | **Legitimate**: select emoji/text presentation and CJK ideographic variants |
| **Tag characters** | U+E0000–U+E007F | Deprecated by Unicode; invisible; used for hidden payloads (the "invisible tweet" trick). Almost never legitimate |
| **Invisible/filler** | U+2800 braille blank, U+3164 hangul filler, U+FFA0 halfwidth hangul filler, U+115F/U+1160, U+17B4/U+17B5, U+180E, U+034F | Render as blank in most fonts; some have real linguistic uses in their scripts |
| **Exotic spaces** | U+2000–U+200A (en/em/thin/hair…), U+202F narrow no-break space, U+205F, U+3000 ideographic space, U+00A0 NBSP | **Legitimate** in typography: French uses U+202F/U+00A0 before `;:!?` and inside `« »`. Replacing them with U+0020 is safe; deleting them is not |
| **Invisible operators** | U+2061–U+2064 | Mathematical notation; invisible |
| **Line/paragraph separators** | U+2028, U+2029 | Valid in text; some parsers treat them as bytes |
| **Combining marks** | U+0300 block, U+0483… | Invisible *alone* (dotted circle), visible attached. Used to encode but overlapping with accent marks |
| **Homoglyphs** | Cyrillic `а е о р с х`, Greek `Α Β Ε Ο Ρ`, fullwidth forms, mathematical alphanumerics | Not invisible but visually identical to Latin. Encode identity/watermark by substitution; used in phishing and spam filtering evasion |

## 3. Encoding schemes built on them

- **Presence/absence**: a binary payload in where invisible characters are placed (e.g. after each word encodes 1, nowhere encodes 0), or in the *choice* among zero-width variants (U+200B = 0, U+200C = 1, U+2060 = separator).
- **Spaces as symbols**: the sentinel characters of a paragraph order the payload (first-line space U+202F, second U+2009, etc.); invisible to the eye, stable through copy-paste.
- **Tag characters**: entire ASCII strings encoded in the U+E0000 block; invisible in browsers and editors that do not render the tag plane.
- **Homoglyph substitution**: replace selected Latin letters with identical-looking Cyrillic/Greek ones. The text looks normal, the string differs, and naive matching/similarity checks fail.
- **Variation-selector steganography**: annex bits in selector sequences after emoji or CJK characters.

None of this needs a key shared with a model sampler: it is written into the text *after* (or beside) generation, often by an interface, a downloader or a platform. That is why it is not covered by the "sampler" rule that structures the rest of the dossier.

## 4. How they are detected

Deterministic inspection, no key required:

1. **Unicode categories**: `Cf` (format, includes ZWSP, joiners, bidi, soft hyphen, BOM), `Mn` (nonspacing marks), `Zs` (spaces beyond U+0020), plus the tag block U+E0000–U+E007F.
2. **Name-based scan**: `unicodedata.name(ch, "")` exposes carriers like `ZERO WIDTH SPACE` or `TAG LATIN CAPITAL LETTER A`.
3. **Script mixing**: characters from two scripts inside one word are a homoglyph signal.
4. **Practical tooling**: editors with "show invisible characters", `cat -A`, hexdumps, `unicodedata` scripts, or dedicated scanners (the `watermarks-remover` Layer A, `inspect_text.py`). A quick Python check:

```python
import unicodedata
suspicious = [(hex(ord(c)), unicodedata.name(c, "?"), unicodedata.category(c))
              for c in text if unicodedata.category(c) in ("Cf", "Mn")
              or 0xE0000 <= ord(c) <= 0xE007F
              or ord(c) in (0x200B, 0x2060, 0xFEFF, 0x00AD)]
```

**Honest limits**: many of these characters are normal (ZWJ in every emoji family, bidi in Arabic/Hebrew, NBSP in French). Finding a carrier does not prove a watermark; not finding one does not prove the text is unmarked — the statistical layer, if present, leaves no bytes.

## 5. How they are removed (and the false positives)

Removal is exact — the bytes are there or they are not — but it is not "delete everything invisible":

1. **Targeted strip**: U+200B, U+2060, U+FEFF (except at the start of a file, where it may be a legitimate BOM), tag characters U+E0000–U+E007F, U+2028/U+2029 when unwanted, deprecated format controls U+206A–U+206F.
2. **Space normalization**: exotic spaces to U+0020 — **with the French/typographic caveat**: many tools expose a `normalize_spaces=false` option precisely to preserve U+00A0/U+202F in `« »` and before `;:!?`.
3. **Do not strip blindly**: ZWJ/ZWNJ are needed for emoji and Indic/Persian text; variation selectors select emoji presentation and CJK variants; bidi controls are required for bidirectional scripts; combining marks are accents. A generic `Cf` filter breaks real multilingual text — including the text of users who did nothing wrong.
4. **Normalization is not a silver bullet**: NFC preserves most carriers (it composes accents, it does not delete format characters); NFKC removes some compatibility forms (fullwidth, ligatures, superscripts) but **changes meaning and typography**; NFKC casefold is more aggressive still. Normalization should be a deliberate choice, not a reflex.
5. **Format matters**: in plain text/UTF-8 the strip is deterministic; in DOCX/ODT/PDF/HTML the carriers live inside a container (XML parts, content streams, comments) and the container must be rebuilt, not just the visible text.

## 6. How this relates to the statistical layer

| | Statistical watermark | Unicode carrier |
|---|---|---|
| Where it lives | Sampling randomness | Text bytes |
| Who applies it | The provider's sampler (generation) | Whoever writes the string (platform, app, script) |
| Keyed? | Yes (secret key) | Usually not; encoding may be trivial |
| Detection | Statistical, with key or detector access | Deterministic, anyone |
| Removal | Degradation (paraphrase, re-sampling) | Exact strip, with false-positive care |
| Survives copy-paste | Yes | Yes, unless the destination sanitizes |

A text can carry both at once: a statistical mark from the model and Unicode carriers from the platform. Removing one does not touch the other. A complete assessment of "what travels with this text" therefore looks at both layers — and at metadata, which travels with *files* but not with copied text (see [common mitigation](mitigation.md) §2.7).

## 7. Reading

The Unicode layer is the most transparent of all: it is bytes, it is testable, and its removal is exact. Its risk is not cryptographic but typographic — the collateral damage of deleting characters that legitimate languages and typography require. That is the same trade-off as the statistical layer, in a different key: **there is no eraser without users who pay for it.**

> **Note**: this document describes the existence, detection and removal of carriers. It does not recommend altering content that is not yours or breaking rules; it documents where the "invisible" of a text actually lives.

## References

- `watermarks-remover` — Layer A (invisible Unicode, exotic spaces, bidi, tag chars) vs Layer B (statistical, best-effort) — [GitHub](https://github.com/guillaumemeyer/watermarks-remover)
- Unicode Standard, code point categories and the U+E0000 tag block — [unicode.org](https://www.unicode.org/charts/)
- [Common mitigation](mitigation.md) — the statistical layer and the general framework
- [The sampler](sampler.md) — why the statistical mark lives in generation, not in the bytes
