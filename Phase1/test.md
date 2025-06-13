---
title: "AffLink Agent Design Specification"
status: "Draft"
audience: "Internal – AI Engineering, DevOps, Phase 1 Agent Developers"
authors: "VirtualAgentics AI Engineering Team"
version: "0.1"
date: "2025-06-13"
gpt_model: "Drafted by ChatGPT-4.1"
---

# AffLink Agent Design Specification

## 1. Overview

### 1.1 Agent Purpose and Goals

The **AffLink agent** is responsible for enriching generated content with affiliate links. Its primary goal is to automatically integrate monetization and referral tracking into content produced by VirtualAgentics’ content pipeline. By scanning content for relevant keywords or product mentions and inserting the appropriate affiliate hyperlinks, the AffLink agent ensures that published content can generate affiliate revenue without requiring manual editing. This agent thereby streamlines the content monetization process, guaranteeing consistency in how affiliate links are applied and freeing content creators from the tedious task of adding links by hand. In summary, the AffLink agent exists to **add value to content** (through monetization) while maintaining content quality and compliance with affiliate program requirements.

### 1.2 Context within VirtualAgentics

Within the **VirtualAgentics Phase 1** content automation pipeline, the AffLink agent operates as a mid-pipeline service that transforms *plain content* into *monetized content*. It works in concert with other Phase 1 agents such as the Content Generation agent, Review agent, and Publish agent. After content is generated (and optionally reviewed for quality or compliance), the AffLink agent is invoked to embed affiliate tracking links. This positioning ensures that by the time content reaches the publishing stage, it has been augmented with any relevant affiliate codes. The AffLink agent plays a crucial role in VirtualAgentics’ strategy to monetize AI-generated content, acting as the bridge between content creation and content publication by adding revenue-generating links. It is an **event-driven, autonomous microservice** (implemented as a Lambda function) that listens for content-ready events and responds by enriching the content data accordingly.

### 1.3 Scope & Assumptions

This design specification focuses on the **Phase 1 implementation** of the AffLink agent, outlining its current capabilities and limitations. The scope covers how the agent processes textual content (e.g., blog articles in Markdown or HTML format) to insert affiliate hyperlinks for known products or keywords. It assumes that content to be processed is in a suitable format and has passed any necessary initial quality checks (i.e., the content is final or near-final draft). Key assumptions include:

- **Pre-Defined Affiliate Mapping:** Phase 1 relies on a predetermined set of affiliate programs or link templates (e.g., a fixed affiliate code for a specific e-commerce site). The agent assumes that it has access to these mappings (via configuration or code) and does not need to dynamically fetch affiliate info from external services in this phase.
- **Single-Pass Enrichment:** The content is processed for affiliate linking **once** at the appropriate point in the pipeline. It is assumed that content will not require multiple rounds of affiliate link insertion; the AffLink agent’s output is considered the final monetized content ready for publishing.
- **Content Characteristics:** It is assumed that content items are of moderate length (e.g., a few hundred to a few thousand words) and contain identifiable product names or keywords that correspond to affiliate links. Content without any recognizable affiliate keywords will simply pass through unchanged (the absence of affiliate links is acceptable if nothing relevant is found).
- **Pipeline Integration:** The AffLink agent operates under the assumption that upstream components (ContentGen, Review) and downstream components (Publish) handle their respective concerns. For example, the agent expects that by the time it runs, the content is approved for publication and only needs link enrichment. Likewise, it assumes a downstream component will take the enriched content and actually publish it, including handling any disclosure or formatting requirements for affiliate content (e.g., adding an “affiliate links included” disclaimer if required on the publishing platform).

By limiting the scope to automated affiliate link insertion for textual content, this design ensures a clear focus on the agent’s core responsibility. Future phases may broaden this scope (see **Chapter 12**), but this document confines itself to Phase 1 behavior and design constraints.

## 2. Architecture & System Context

### 2.1 High-Level Context Diagram

```mermaid
flowchart LR
    subgraph "Phase 1 Content Pipeline"
        CMO["CMO Agent (Orchestrator)"]
        CG["ContentGen Agent (Lambda)"]
        RV["Review Agent (Lambda)"]
        AL["AffLink Agent (Lambda)"]
        PB["Publish Agent (Lambda)"]
    end
    CMO -->|ContentRequest event| CG
    CG -->|ContentGenerated event| RV
    RV -->|ContentReviewed event| AL
    AL -->|ContentEnriched event| PB
    S3[(Content Repository - S3 Bucket)]
    CG -.->|write content| S3
    RV -.->|update content| S3
    AL -.->|read & write content| S3
    PB -.->|read content| S3
