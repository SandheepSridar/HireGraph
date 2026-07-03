# HireGraph

A LangGraph-based pipeline that checks how well a candidate's resume matches a specific job posting, and tells them what's missing.

## Planned flow (MVP)

1. **Input:** a candidate profile (resume text, plus `location` and approximate `role` as filters) and a link to one job description.
2. **Validate** the inputs.
3. **Fetch and parse** the JD from the link.
4. **Score the match** between resume and JD — a hybrid of rule-based signals and an LLM judgment.
5. **Recommend** the role if the match score is 75% or higher.
6. **Surface gaps:** list must-have keywords from the JD that are missing from the resume.
7. **Log submissions:** keep a record (JD + timestamp) of what's been checked.

This is the first slice of a larger scope — see `PROJECT_PLAN.md` for the full multi-candidate, bias-detection, and ranking curriculum this is building toward, and `instructions.md` for how this project is meant to be worked on (teaching mode, progress log).

## Stack

Python, LangGraph, LLM APIs (local via LM Studio during development).
