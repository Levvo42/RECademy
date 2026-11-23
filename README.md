# LIA Search Tracker

A relational database that helps students organise their search for an internship (LIA – Lärande i arbete). It keeps track of the companies you contact, the people you talk to, the applications you send and what needs to happen next.

## Background

Searching for LIA usually means contacting dozens of companies over several months. The information ends up spread across email, LinkedIn, notes and memory, and it gets hard to answer simple questions:

- Which companies have I already contacted, and when?
- Who did I talk to, and what did we say?
- Which applications are still waiting for an answer?
- Who should I follow up with this week?

The LIA Search Tracker gives the student one place for the whole search, from the first contact to a signed placement.

## Target user

A student at a YH or university programme who is looking for a LIA placement. Several students can use the same database, and each student only works with their own data.

## Features

- Save companies you are interested in, with location and what technologies they work with
- Save contact persons at each company
- Track each application through its status (planned → sent → interview → offer / rejected)
- Log every interaction: emails, calls, meetings and interviews, with date and notes
- Set follow-up dates, so you know who to contact next
- Keep track of which CV and cover letter version you sent to which company
- Prevent invalid data with constraints (e.g. no duplicate application to the same company)

## Data model

### Tables

| Table | Purpose |
| --- | --- |
| `Students` | The users of the tool |
| `Companies` | Companies the student is interested in |
| `ContactPersons` | People at a company (recruiters, developers, managers) |
| `Technologies` | Technologies, e.g. C#, .NET, Azure, React |
| `CompanyTechnologies` | Bridge: which technologies a company works with |
| `Documents` | The student's CVs and cover letters, with version names |
| `Applications` | A student's application to a company |
| `Interactions` | A logged contact: email, call, meeting or interview |

### Relationships

- `Students` 1 → many `Applications`
- `Students` 1 → many `Documents`
- `Companies` 1 → many `ContactPersons`
- `Companies` 1 → many `Applications`
- `Companies` many ↔ many `Technologies` (via `CompanyTechnologies`)
- `Applications` 1 → many `Interactions`
- `ContactPersons` 1 → many `Interactions` (optional: an interaction can be without a named person)
- `Documents` 1 → many `Applications` (the CV that was sent)

### Constraints

| Constraint | Rule |
| --- | --- |
| `UQ_Students_Email` | A student's email is unique |
| `UQ_Companies_Name` | A company is only registered once |
| `UQ_Technologies_Name` | A technology is only registered once |
| `PK_CompanyTechnologies` | Composite key `(CompanyId, TechnologyId)`: a technology is only linked once per company |
| `UQ_Applications_Student_Company` | A student can only have one application per company |
| `CK_Applications_Status` | Status is one of `Planned`, `Sent`, `Interview`, `Offer`, `Accepted`, `Rejected`, `Withdrawn` |
| `CK_Interactions_Type` | Type is one of `Email`, `Call`, `Meeting`, `Interview`, `Other` |
| `CK_Documents_Type` | Type is `CV` or `CoverLetter` |
| `CK_Applications_FollowUp` | `FollowUpDate` is not earlier than `SentDate` |

### ER diagram

![ER diagram](docs/erd.png)

## Example queries

Applications that need a follow-up today or earlier:

```sql
SELECT c.Name, a.Status, a.FollowUpDate
FROM Applications a
JOIN Companies c ON c.Id = a.CompanyId
WHERE a.StudentId = @StudentId
  AND a.FollowUpDate <= CAST(GETDATE() AS date)
  AND a.Status IN ('Sent', 'Interview')
ORDER BY a.FollowUpDate;
```

Overview of the search, number of applications per status:

```sql
SELECT Status, COUNT(*) AS Applications
FROM Applications
WHERE StudentId = @StudentId
GROUP BY Status
ORDER BY Applications DESC;
```

Companies that work with C# and have not been contacted yet:

```sql
SELECT c.Name, c.City
FROM Companies c
JOIN CompanyTechnologies ct ON ct.CompanyId = c.Id
JOIN Technologies t ON t.Id = ct.TechnologyId
WHERE t.Name = 'C#'
  AND NOT EXISTS (
      SELECT 1
      FROM Applications a
      WHERE a.CompanyId = c.Id
        AND a.StudentId = @StudentId
  );
```

## Tech

- Microsoft SQL Server
- SQL Server Management Studio (SSMS) or Azure Data Studio

## Getting started

1. Create an empty database called `LiaSearchTracker`.
2. Run `sql/01_create_tables.sql` to create tables and constraints.
3. Run `sql/02_seed_data.sql` to insert example data.
4. Try the queries in `sql/03_example_queries.sql`.

## Project structure

```
LiaSearchTracker/
├── README.md
├── docs/
│   └── erd.png
└── sql/
    ├── 01_create_tables.sql
    ├── 02_seed_data.sql
    └── 03_example_queries.sql
```

## Possible extensions

- Reminders by email when a follow-up date is reached
- Import of companies from job ads or LinkedIn
- Shared company list for a whole class, with private applications per student
- A web interface (ASP.NET Core) on top of the database

## Authors

- *Names*