---
title: "Project Documentation Guidelines – Virtual Agentics"
status: "Stable Draft"
audience: "Internal (All Contributors, AI/LLM, Auditors, Stakeholders)"
authors: "VirtualAgentics Core Team (generated by ChatGPT GPT-4.5, curated by Ben)"
version: "1.0"
date: "2025-06-06"
gpt_model: "Drafted by GPT-4.5 Deep Research"
---

# Project Documentation Guidelines: Virtual Agentics

## Purpose

This file defines the documentation, style, and maintenance standards for all markdown (.md) files in the Virtual Agentics project. It ensures clarity, auditability, modularity, and consistency, regardless of whether docs are written by humans or LLMs.

---

## 1. General Principles

- **Source of truth:** All architectural, operational, policy, and reference docs must be stored in the main documentation repo (`virtualagentics-documents`).
- **Everything as Markdown:** All docs are in `.md` format with UTF-8 encoding and must render properly in GitHub.
- **Consistent headers:** Every file begins with a YAML frontmatter block: title, status, audience, authors, version, date, gpt_model (if applicable).
- **Structured content:** Use H1 for document title, H2 for top-level sections, H3 for subsections; follow a clear hierarchy.
- **Explicit references:** Always link to other docs/files using relative links.
- **Searchable:** Use descriptive, consistent filenames and headings for easy searchability and navigation.

---

## 2. Formatting Standards

- **Section order:** Purpose, Scope, Table of Contents (if long), then content sections, then References.
- **Tables:** Use for mappings, resource lists, config matrices.
- **Code blocks:** Use fenced code (```) and indicate language (e.g., ```hcl, ```bash, ```python).
- **Lists:** Use bullets for unordered, numbers for ordered; no mixed-style lists.
- **Metadata in code:** All code or config samples must include in-line comments describing intent.
- **References:** Final section in each doc; use explicit Markdown links to related files or sources.

---

## 3. Detail and Clarity

- **No ambiguity:** All steps, parameters, and requirements must be explicit—avoid “optional” or “example.com” if specifics are known.
- **Repeat for audit:** If a process/step/decision is important for reproducibility, it should be fully documented even if cross-referenced elsewhere.
- **Minimal placeholders:** Placeholders should be marked as such and replaced as soon as the real value is known.

---

## 4. Tone, Language, and Style

- **Clear and professional:** Use present tense, active voice, and direct instructions where appropriate.
- **Audience-aware:** Technical docs may assume cloud/infra proficiency; policy or vision docs should explain acronyms and context.
- **No jargon without definition:** Define all acronyms and project-specific terms at first use.
- **No marketing language:** Documentation is technical and factual, not promotional.
- **LLM/AI authorship:** Clearly mark AI-generated content via `gpt_model` metadata.

---

## 5. Review and Update Policy

- **PR/review required:** All documentation changes must go through pull request (PR) and at least one reviewer approval.
- **Change logs:** Major docs (specs, policies, guides) must maintain a changelog section or PR history.
- **Versioning:** Increment version in frontmatter on substantive changes.
- **Obsolete docs:** Mark with `status: Deprecated` and reference successor file.

---

## 6. Special Notes for AI/LLM-Generated Docs

- **Model/version in header:** Specify LLM/model and workflow in YAML header.
- **Deep research:** Use explicit citation formatting for external knowledge or sources.
- **Responsibility:** All AI-generated docs must be reviewed by a human before merging to `main`.
- **Reasoning:** Where relevant, note where LLM has made a recommendation based on best practices or external context.

---

## 7. Table of Example Metadata (Frontmatter)

| Field       | Description                                   | Example                        |
|-------------|-----------------------------------------------|--------------------------------|
| title       | Document title                                | "Technical Specification – ..."|
| status      | Draft, Stable Draft, Final, Deprecated        | "Stable Draft"                 |
| audience    | Who should read this doc                      | "Internal (Engineering...)"    |
| authors     | Contributor(s), model(s), reviewers           | "Core Team (GPT-4.5, Ben)"     |
| version     | Doc version                                   | "1.0"                          |
| date        | ISO date                                      | "2025-06-06"                   |
| gpt_model   | Model name/version if LLM generated           | "GPT-4.5 Deep Research"        |

---

## 8. References

- [Technical_Specification_v1.md](Technical_Specification_v1.md)
- [High-Level_Framework.md](High-Level_Framework.md)
- [Naming_Conventions.md](Naming_Conventions.md)
- Source: "Deep Research Task Guidelines" and Virtual Agentics project practices

---

*End of document*
