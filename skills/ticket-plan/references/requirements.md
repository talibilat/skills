# Requirements and project knowledge

Every plan, build and review starts from the original requirement documents, not from the ticket text alone. Tickets are summaries. The requirement documents are the truth.

## Find the requirement documents

Follow the links from the ticket until you reach the source. Stop at 3 hops from the ticket for each branch.

1. The ticket: description, attachments, comments, remote links (`getJiraIssueRemoteIssueLinks`), linked issues.
2. The parent chain: sub-task, task, story, epic, initiative. Read each one's description and links.
3. Confluence pages linked from any of them. Fetch the page, its child pages and its version history titles. Search Confluence for the epic key and the feature name (`searchConfluenceUsingCql`).
4. Attached files (PDF, Word, spreadsheets, designs, API specs). Read what you can. Name what you cannot open.
5. Sibling tickets in the epic that touch the same feature, for decisions that were made later.

## Make sense of versions

Requirement documents often exist in several versions.

- Order the versions by date and version number. The newest approved one wins. Say which one you used and its date.
- A later ticket, comment or change request can override a document. Record the override and where it came from.
- When two sources disagree and nothing says which is newer, that is a conflict. Write it under Blockers in the plan. In autopilot, take the newest source and flag the conflict in the PR and the Jira comment. The QA verdict "Needs BA input" is for conflicts that change what gets built.
- Never invent a requirement. A gap is written as a gap.

## Keep project knowledge

Keep one notes file per epic (or per project when there is no epic) at `~/.agents/ticket-knowledge/<EPIC-or-PROJECT-KEY>.md`. Create the folder when missing. It is private and outside every repo.

Layout, kept under 150 lines:

```
# <EPIC-KEY> knowledge
Updated: <date> by <ticket key>

## Requirement sources
| Document | Version / date | Status (current, superseded, unclear) | Link |

## What the requirements say
<Short bullets, each with its source. The rules, limits, roles and flows that matter.>

## Decisions and overrides
<What changed later, who said it, where.>

## Known gaps and conflicts
<Open questions.>
```

Rules:

- Read the file first. If it exists and its Updated date is recent and no source changed, use it and skip re-reading every document. Check the version list of each source for changes.
- Update it whenever a search finds something new or something changed: a new version, an override, a gap closed. Edit the existing line, do not append duplicates.
- Put a one-line "Requirements used" entry in the plan, listing the document, version and date. Give reviewers and `aqa` the knowledge file path as well as the ticket and plan paths.
- A change you learn about mid-build (a comment, a new version) goes into the knowledge file and the plan before the next batch starts. If it changes the acceptance criteria, treat it as a failed round and replan.
