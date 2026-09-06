# Shawn Life OS — WeChat Test Data Manifest

Status: **Active corpus manifest for the first WeChat vertical slice.**

This file records accepted real-data fixtures, upstream structural fixtures, candidates, provenance, license/usage status, annotation source, and corpus gaps. It is normative for DATA-GATE accounting in `WECHAT_REAL_DATA_TEST_SPEC.md`.

## 1. Accounting rules

A fixture counts toward the **real-data gate** only when all of the following are true:

- it is a real-world WeChat screenshot/export or user-authorized real sample;
- its origin is attributable;
- immutable source identity is recorded (blob/content hash, commit/path, or equivalent);
- privacy review is acceptable;
- license/usage status is recorded;
- required labels are available or can be independently/manual annotated from visible evidence;
- it is not merely synthetic/upstream-generated test UI.

Synthetic fixtures and generated mutations are useful but never increase the real-original count.

## 2. Current corpus summary

| Category | Confirmed count | DATA-GATE contribution |
|---|---:|---:|
| Real positive chat screenshots with transcript/labels | 1 | 1 |
| Real positive/candidate screenshots without verified labels | 1 candidate | 0 until reviewed |
| Real negative/non-chat screenshots | 0 confirmed | 0 |
| Upstream synthetic/structured parser fixtures | multiple | 0 |
| User-authorized redacted real screenshots | 0 | 0 |

**Current status: DATA-GATE-0 is only partially satisfied; DATA-GATE-1 is NOT satisfied. Production WeChat implementation remains blocked.**

## 3. Accepted real fixture — WX-RD-001

### Identity

- Fixture ID: `WX-RD-001`
- Origin repository: `marsoyang1/weixin_ocr`
- Repository path: `image/5e5eb526-90e7-4915-8e00-b363f8bce2b2.jpg`
- Git blob SHA: `561cb001aed2ba5809b2056a30953976e49678ad`
- Published paired transcript: `result.txt`
- Transcript blob SHA: `024af68901231e94f48dccc6ede80dbeab57f319`
- Repository license: GPL-3.0
- Data type: real WeChat direct-message screenshot published by the project as its demonstration/input sample
- Privacy class for testing: `PUBLICLY_PUBLISHED_TEST_SAMPLE`, but content is still personal-conversation-like and must be handled conservatively

### Ground truth

The paired project transcript contains the visible message sequence with speaker-side labels (`好友` / `自己`). This is acceptable as **source-published ground truth** for initial parser/OCR benchmarking, subject to visual/manual verification before promotion into a long-lived bundled fixture.

Expected transcript sequence:

```text
好友:日为朝，月为暮。囡为朝朝暮暮。
好友:400自提得行不
自己:太低了哦
好友:我还要去买个显示器。唉。
自己:显示器便宜
自己:如果要大,就用电视
自己:我都是用的电视
好友:不合适的嘛。我要玩梦幻西游搬砖
自己:噢噢
好友:怎么样。400自提行不
自己:在高点呢
```

### Intended tests

- `WX-OCR-001`
- `WX-OCR-002`
- `WX-PARSE-001`
- `WX-PARSE-002`
- `WX-SCHEMA-001`
- `WX-E2E-001`
- `WX-ADV-001..004` through deterministic derivatives

### Redistribution decision

**Do not copy the image into the Life OS public source repository yet.**

Reason: the code repository has GPL-3.0 licensing, but the copyright/privacy status of conversation screenshot content should be treated separately from source-code licensing. For now, reference the upstream path + immutable blob SHA and fetch it only in an explicitly documented test-data preparation workflow if legally appropriate. A later data-license/provenance review can approve bundling or require replacement with user-authorized redacted samples.

## 4. Candidate real fixture — WX-RD-CAND-002

### Identity

- Candidate ID: `WX-RD-CAND-002`
- Origin repository: `ai4evt/wechat-ai-reply`
- Repository tree snapshot observed: `ed133a618d09ff00ccf184d70370e606c7606cf7`
- Repository path: `demo.png`
- Git blob SHA: `6ab81c53358dc369251810e31f46fc391149d720`
- Repository license: MIT

### Status

`CANDIDATE / NOT COUNTED`

The file is intentionally published by a WeChat screenshot/OCR project, but before counting it as real test data we must visually verify:

- whether it is an actual WeChat UI screenshot rather than a composite/marketing graphic;
- whether it contains a chat page or another page;
- whether any personal content requires redaction or exclusion;
- which labels can be established without using the system under test.

Until that review is complete, it contributes **0** to DATA-GATE-1.

## 5. Upstream structural fixtures — WX-UPSTREAM-001

### Origin

Project: `iwxyi/WeChat-WhoChat`

Observed repository tree snapshot:
`2f32e57bf8f72c0719f8993e0f747d1c3325d846`

Useful fixture assets include:

- `fixtures/ocr/golden_chat_dm.json`
- `fixtures/ocr/golden_chat_dm_time.json`
- `fixtures/ocr/golden_chat_group.json`
- `fixtures/ocr/golden_news_article_page.json`
- `fixtures/ocr/golden_official_account_page.json`
- `fixtures/ocr/golden_settings_page.json`
- `fixtures/ocr/golden_unknown_partial.json`
- `fixtures/screenshot_samples/wechat_dm_synthetic/...`

The upstream screenshot-sample directory explicitly includes a synthetic sample and its replay manifest/layout structure.

### Use

These are valuable for:

- learning/reusing the fixture format;
- parser-port contract design;
- page-classification cases;
- speaker/order/timestamp replay structure;
- validating our thin adapter against an upstream parser implementation.

### DATA-GATE accounting

**0 real-data originals.**

They must never be counted as proof that Life OS works on real WeChat screenshots.

### License/reuse caution

No repository `LICENSE` file was confirmed during the current review. Therefore do not copy WhoChat source/fixtures into Life OS merely because they are publicly visible. Prefer adapter/reuse only after license status is clarified. Publicly observable interface/fixture structure may inform our independent contract design.

## 6. Required corpus additions before DATA-GATE-1

We still need at least **7 additional accepted real screenshots** and a second independent real-data origin.

Priority acquisition order:

1. **User-authorized redacted real WeChat screenshots** — preferred because provenance/privacy authorization is strongest and layouts represent the actual target environment.
2. Additional intentionally published public open-source WeChat OCR/parser demo screenshots with clear license/usage context.
3. Real WeChat UI screenshots that can be used only as negative/layout/page-classification samples where textual ground truth is not needed.

Required coverage to add:

- direct-message screenshot with explicit visible timestamp separator;
- direct-message screenshot with no visible timestamp;
- long Chinese message / line wrapping;
- mixed Chinese + digits + Latin text;
- left/right message alternation;
- partial top/bottom message;
- overlapping scroll pair;
- non-chat/settings/search/article page;
- unsupported content (image/sticker/file/voice/system message) if legally available.

## 7. Annotation protocol

For a real screenshot without a source-published transcript, create a sidecar annotation manually from visible evidence.

Required fields per visible message:

```text
message_index
speaker_side = SELF | OTHER | UNKNOWN
visible_text_exact
message_type
visible_timestamp_text (nullable)
timestamp_association (nullable/unknown allowed)
partial = true|false
bounding_region (when used for OCR geometry tests)
annotation_method
reviewer_status
```

Ground-truth annotation must not call the OCR provider/parser under test to decide the expected answer.

For release-quality labelled fixtures, require an independent second review for speaker side, ordering, timestamp association, and exact text.

## 8. Mutation lineage schema

Every mutation derived from a real original receives its own manifest record:

```yaml
fixtureId: WX-RD-001-JPEG-Q75
parentFixtureId: WX-RD-001
mutation:
  type: jpeg_reencode
  parameters:
    quality: 75
  implementation: <library/tool + version>
sourceHash: <original content hash>
outputHash: <mutated content hash>
expectedInvariant:
  - no fabricated messages
  - speaker order preserved for accepted messages
  - provenance points to parent + mutation
```

Mutations do not increase the count of independent real originals.

## 9. Planned test-data directory contract

When executable tests are added to `lazerta/shawn-life-os`, use a structure equivalent to:

```text
src/test/resources/realdata/wechat/
  manifest/
  originals/          # only fixtures whose redistribution is explicitly approved
  annotations/
  ocr-results/        # versioned provider output for replay
  mutations/
  expected/
```

Externally referenced fixtures that cannot be redistributed remain represented by manifest metadata + immutable origin/hash and are fetched/prepared only through a documented opt-in test-data preparation step.

## 10. Exit criteria for test-data phase

Before production implementation starts, this manifest must show:

- DATA-GATE-1 satisfied;
- every accepted real fixture has immutable identity and usage/privacy status;
- every positive fixture has deterministic ground truth for ordering and speaker side;
- negative fixtures have explicit expected non-publication behavior;
- mutation plan is reproducible;
- parser/OCR expected outputs are independent of the implementation under test.

Until then, **do not treat the WeChat vertical slice as implementation-ready**.