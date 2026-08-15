# AI Demand-to-Content Generator — sanitized export

This directory documents the public-safe architecture of the Potik AOS demand-to-content workflow.

## What is included

- workflow name, node types and positions;
- high-level node roles;
- graph connections between nodes;
- non-sensitive operating contract;
- retry/timeout indicators for external calls.

## What is intentionally removed

The sanitized export does not contain:

- OpenAI prompts or system instructions;
- JavaScript source code;
- n8n expressions and field mappings;
- credentials or tokens;
- Google Sheets IDs, Drive folder IDs or private resource identifiers;
- private source data.

The JSON is documentation-only and is not directly importable into n8n. The complete workflow remains in the private n8n workspace.

## Functional summary

The workflow manually selects up to three demand topics, generates Ukrainian LinkedIn content through three HTTP calls to OpenAI, runs a strict quality check, creates a branded visual asset, stores the result for review in Social Content Studio, updates the source status and records usage in Automation_Runs.

Quality failures keep the topic selected for revision. Social publishing is outside the MVP.

## Brand and visual direction

The visual direction follows Potik AOS: deep navy, yellow primary accent, cyan/violet technology accents, and the visual metaphor “manual chaos → clear process → business result”.

The current n8n implementation uses an image-generation step. A future implementation may render deterministic visuals with HTML/CSS/JS for exact typography and repeatable layouts.
