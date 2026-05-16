# Project: Quill — AI Code Authorship Provenance Platform

## What This Project Is

We are building **Quill**, a developer tool that creates legally defensible evidence of human creative authorship in AI-assisted software development workflows. The core problem: copyright law now firmly requires human authorship for IP protection, but the majority of production code is AI-assisted with no systematic record of the human decisions that make it protectable. We are building the chain-of-custody system for code IP.

## Why This Exists

- The Supreme Court (March 2026) finalized that AI-generated works cannot receive copyright protection without human authorship
- 51%+ of code committed to GitHub in 2026 is AI-generated or AI-assisted
- No purpose-built tool exists to capture and structure the human authorship record during development
- The EU AI Act (Article 50, enforceable August 2026) mandates machine-readable provenance disclosure
- M&A diligence, fundraising, and enterprise procurement are already asking these questions

## What We Are Building (Initial Hypothesis)

A lightweight developer tool — IDE plugin + git layer — that passively captures:
- Prompts used, model versions, timestamps
- Human edits, refactoring decisions, architectural choices made to AI output
- A structured, exportable "authorship artifact" suitable for copyright registration support and legal due diligence

## What This Project Folder Contains

This folder contains the working documents for product discovery, strategy, and engineering planning for Quill. Claude should use these documents as the primary context for all responses in this project. New documents will be added as the project evolves.

## How Claude Should Behave in This Project

- Treat all questions through the lens of: **what helps us validate, build, or go to market with Quill**
- Default to PM/product thinking for strategy questions; engineering architecture for technical questions; legal grounding for IP/compliance questions
- Flag when assumptions in the existing docs need revisiting based on new information
- Be direct about gaps and risks — we are in early discovery and need honest signal, not confirmation bias
- When generating artifacts (specs, plans, research), match the format of the existing project documents
