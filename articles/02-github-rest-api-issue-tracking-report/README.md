# Portfolio Record for Article 2

**Article title:** How to Build a Simple Issue-Tracking Report with the GitHub REST API

**Portfolio role:** Document

**Article type:** Beginner practical API workflow tutorial

**Primary technology / tools:** GitHub REST API • Postman • `github/docs` repository

**Audience:** Beginner developers and technical learners with basic familiarity with APIs and JSON

**URL slug:** `github-rest-api-issue-tracking-report`

## What this demonstrates

Document a practical developer workflow that turns GitHub Issues API data into a compact, useful repository report.

## Core workflow

Retrieve repository records → control the dataset → inspect the JSON structure → classify Issues and Pull Requests → extract selected fields → build the report → verify the result

## Hands-on / evidence basis

- Baseline retrieval with fields: `per_page=10`, `state=open`, `sort=updated`, `direction=desc`.
- Issue/Pull Request classification via the `pull_request` field.
- Fields extraction.
- Final report verification.

## Sources

- GitHub REST API Issues documentation
- Hands-on Postman experiments against the public `github/docs` repository.

## Article

Read the full article here: [Article 2: How to Build a Simple Issue-Tracking Report with the GitHub REST API](Article-02.md)

## Assets

See the [`assets/README.md`](assets/README.md) file for an inventory of the article's supporting evidence and output files.

**Note:** The report reflects the repository data observed during testing; issue numbers, titles, labels, and timestamps can change after publication.
