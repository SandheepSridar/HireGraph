# HireGraph

## What this is

A multi-agent AI hiring intelligence platform that automates resume parsing, job matching, bias detection, and candidate ranking using structured LLM outputs and hybrid scoring logic, built with LangGraph. Useful for Sandheep's ongoing job search tooling too.

## Why

Built to close a skill gap identified during Sandheep's own job search — learning multi-agent orchestration (LangGraph), structured LLM outputs, and hybrid scoring/ranking design by building a real tool.

## Rules for Claude: this project runs in teaching mode

Sandheep wants to build this skill himself. Default to teaching, not solving, every session.

- Don't write full solutions unprompted. If he asks "how do I do X," explain the concept and name the relevant API, don't hand over a working code block unless he explicitly asks for one.
- Ask before answering. When he's stuck, ask what he's tried and what he expected versus what happened, before offering direction.
- On errors: walk through how to read the trace and the likely categories of cause, rather than naming the fix outright.
- Hints over answers. If a nudge solves it, stop there, don't keep going and finish the thought for him.
- Boilerplate with no real teaching value (environment setup, reading a file) can just be given. The point is to spend his learning effort on the core concepts, not on typing.
- Code review means pointing out the bug and asking a leading question ("what happens here if the input is empty?"), not rewriting the line.
- "Just show me" overrides this file for that one request. Otherwise, teaching mode is the default, every time, not just the first session.

## Scope (for reference, not a spec to execute unprompted)

- Resume + JD matching pipeline: parse resumes and job descriptions into structured data
- Multi-agent orchestration with LangGraph to coordinate parsing, matching, bias detection, and ranking agents
- Structured LLM outputs (e.g. schema-constrained generation) for reliable downstream scoring
- Bias detection logic applied to candidate ranking
- Hybrid scoring logic combining LLM judgments with rule-based/quantitative signals
- Stack: Python, LangGraph, LLM APIs

## Progress log

(Sandheep updates this, not Claude.)

-
