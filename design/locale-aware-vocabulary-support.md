# Design Proposal: End-to-End Locale-Aware Vocabulary Support for Cicero Templates

## Summary

PR #882 adds foundational support for storing and loading Concerto vocabulary files (`.voc`) in Cicero templates, plus API and CLI access to those vocabularies.

Before landing that work, we should align on the full end-to-end design for how vocabulary support is meant to affect actual user-visible behavior in TemplateMark parsing and drafting. As currently implemented, templates can carry vocabularies, but `grammar.tem.md` remains singular and `cicero parse` / `cicero draft` do not yet become locale-aware in a way users can exercise directly.

This document is intended to capture the design and scope questions before continuing implementation.

## Problem

We need to define how locale-specific vocabularies should participate in the template lifecycle, not just how they are stored.

Questions that remain open:

- How should vocabularies influence parsing and drafting?
- Do grammars become locale-specific, or is there one grammar plus vocabulary substitution?
- How is locale selected at runtime?
- What are the fallback rules when a requested locale is missing?
- What user-facing CLI and API changes are required?

## Goals

- Define the intended end-to-end user experience for localized templates.
- Clarify the relationship between TemplateMark grammars and Concerto vocabularies.
- Specify locale selection and fallback behavior.
- Decide the appropriate implementation split across core, loader/saver, and CLI layers.
- Determine whether PR #882 should land as-is, be split, or be expanded.

## Non-goals

- This document does not implement the feature directly.
- This document does not attempt to finalize every naming detail up front if the behavioral direction is not yet agreed.
- This document is not about generic i18n beyond vocabulary and grammar behavior in Cicero templates.

## Design Questions

### 1. How should vocabularies connect to parsing and drafting?

Possible direction:

- `draft` should be able to render locale-specific natural language using the selected vocabulary.
- `parse` should be able to recognize locale-specific terms and phrases for the selected locale.
- Vocabulary should not just be queryable metadata; it should affect the TemplateMark flow used by end users.

Open questions:

- Is vocabulary substitution sufficient for both parse and draft?
- Are there cases where grammar structure itself differs by locale, beyond term substitution?
- Should parsing require an explicit locale, or attempt detection and fallback?

### 2. Should grammars be locale-aware?

Options worth discussing:

- Single grammar file plus locale-specific vocabulary files
- Locale-specific grammar files, for example:
  - `grammar_en.tem.md`
  - `grammar_fr.tem.md`
- Hybrid model:
  - shared base grammar plus optional locale-specific overrides

Tradeoffs:

- Single grammar is simpler, but may be too limited for languages with structural differences.
- Per-locale grammars are more flexible, but increase maintenance and authoring complexity.

### 3. How should locale be selected at runtime?

Candidate inputs:

- explicit API parameter
- CLI flag such as `--locale`
- template metadata default via `accordproject.defaultLocale`
- possibly document or request context in future

Questions:

- Should explicit locale always win over template default?
- Should absence of locale use `defaultLocale` automatically?
- Should parse and draft fail when no locale is specified and no default exists?

### 4. What should fallback behavior be?

Example questions:

- If locale `fr-CA` is requested, should lookup fall back to `fr`?
- If requested locale is missing, should the system fall back to `defaultLocale`?
- Should fallback behavior be strict by default, or permissive by default?
- Should parse and draft use the same fallback semantics?

### 5. What CLI and API surface should exist?

Possible CLI additions:

- `cicero draft --locale <locale>`
- `cicero parse --locale <locale>`
- keep `cicero vocabulary` for inspection and debugging

Possible API additions:

- locale option on parse and draft entry points
- documented fallback behavior in template APIs

## Proposed Acceptance Criteria

We should consider the design settled when we can answer the following clearly:

- A template author knows where to place locale-specific assets.
- A runtime caller knows how to select locale explicitly.
- The behavior when locale is omitted is defined.
- The behavior when locale is unavailable is defined.
- It is clear whether grammars are single-locale, multi-locale, or hybrid.
- It is clear which parts belong in a first implementation phase versus later phases.

## Suggested Implementation Split

One possible phased approach:

### Phase 1

- finalize design
- settle asset layout and runtime locale model

### Phase 2

- loader and saver support for vocabulary and any locale-specific grammar assets
- stable metadata and fallback behavior

### Phase 3

- locale-aware drafting
- locale-aware parsing
- CLI support for locale selection
- end-to-end tests

## Relation To Existing Work

- PR #882 provides the current foundational implementation for `.voc` loading and saving, vocabulary access, and related tests.
- This document is meant to define whether that work should be landed directly, split into smaller pieces, or incorporated into a broader locale-aware implementation.

## Request For Feedback

Feedback on these points would be especially helpful:

- whether grammar should be single-file or locale-specific
- whether parse and draft should require explicit locale or use `defaultLocale`
- whether fallback should be strict or permissive
- what the minimal user-visible feature set should be for a first mergeable version
