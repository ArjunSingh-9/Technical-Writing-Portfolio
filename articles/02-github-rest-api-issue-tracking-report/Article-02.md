# How to Build a Simple Issue-Tracking Report with the GitHub REST API

## Introduction

In this workflow tutorial, you’ll use Postman to retrieve issues from a GitHub repository and turn the returned data into a small, useful issue-tracking report.

The Issues endpoint can return both issues and pull requests, so the workflow needs to distinguish between them before the report is built.
After building the report, you’ll compare it with the API response to make sure the extracted information is correct.

## What You’ll Build

Transform the JSON data received from GitHub’s Issues endpoint into a compact tracking report in the following format:

| # | Type | Title | State | Labels | Updated |
| --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |

Each column is based on selected data from the records returned by the GitHub REST API:

- #: `number`
- Type: derived from the presence or absence of `pull_request`
- Title: `title`
- State: `state`
- Labels: names from the `labels` array
- Updated: `updated_at`

The API gives us structured JSON containing many properties. The report will use only a small subset and present them in a compact format.

By the end, you’ll have a compact report generated from live repository data and will have verified that each reported value matches the API response.

## What You’ll Need

This tutorial uses Postman to send the GitHub REST API request and inspect the returned JSON. You can either use Postman’s web version or install the desktop version beforehand.

Use the `github/docs` repository for this workflow. Because the repository is public, you don’t need to authenticate the request for this tutorial.

You should be comfortable making a basic API request and reading JSON responses.

## Choose a Repository and Set Up the Request

In the Postman workspace, create an HTTP request to the GitHub Issues endpoint:

1. **Log in to your Postman workspace.**
2. **Create a new HTTP request.**
3. **Select the `GET` method** in the dropdown list next to the URL text box. The Issues endpoint retrieves repository data with `GET`.
4. **Enter the GitHub Issues endpoint** in the URL text box: `https://api.github.com/repos/github/docs/issues`. The URL targets the `issues` resource for the `github/docs` repository.
5. **Switch to the `Headers` tab** below the URL text box.
6. **Add these key-value pairs** in the Headers table:
    - `Accept`: `application/vnd.github+json`
    - `X-GitHub-Api-Version`: `2026-03-10`
    
    These headers configure the request for the GitHub API.
    
7. **Leave the authentication unset.** Authentication tokens are not required for querying a public repository.
8. **Leave the query parameters empty** for this baseline request.

The request is now configured as a baseline request and is ready to run. In the next section, you’ll send it and inspect the response.

## Retrieve a Small Set of Records

### Run the baseline request

Send the baseline request prepared in the previous section:

1. Click the **Send** button next to the URL text box.
2. Inspect the JSON response returned by the Issues endpoint in the Response pane.

The Response pane displays:

- A `200 OK` status indicating a successful request
- A JSON array containing 30 objects (at the time of testing)

**Notice that:**

- The response already contains the information we’ll eventually need for the report, along with many other properties.
- The response is an array of objects, with each object representing a record returned by the Issues endpoint.
- The records can either be issues or pull requests. We’ll distinguish between them later when we build the report.

### Limit the response

The baseline request returns 30 records by default, but we only need a small set while learning the workflow.

To limit the response size, add a new parameter to the baseline request:

1. Open the **Params** tab beneath the URL text box.
2. Add a parameter with the key `per_page` and the value `10` to the Params table.
3. Click the **Send** button next to the URL text box.
4. Confirm that the modified request succeeds and that the JSON response now contains 10 records.

**Note:** The `per_page` parameter controls how many records are returned in the response.

## Filter and Sort the Results

The 10-record response now needs to contain only `open` records, with the most recently updated records first. Add three parameters to the modified request:

1. Open the **Params** tab beneath the URL text box.
2. Add the following three parameters (key-value pairs) to the Params table:
    - `state=open`: Filters the records to return only those whose state is `open`.
    - `sort=updated`: Tells the API to sort the results by their last-updated timestamp, represented by the `updated_at` field.
    - `direction=desc`: Orders the sorted records in descending order, putting the most recently updated records first.
3. Verify that the previously added `per_page=10` parameter stays intact.
4. Click the **Send** button next to the URL text box.
5. Inspect the JSON response to verify that it matches the expected result. Verify explicitly that the Response pane contains:
    - A `200 OK` status indicating a successful request
    - A JSON array containing 10 records, where:
        - Every returned record has `state` set to `open`
        - The 10 records are arranged by the `updated_at` field in descending order

The latest version of the request, including its Params table, should contain the following four parameters:

- `state=open`
- `sort=updated`
- `direction=desc`
- `per_page=10`

**Note:** The response can still contain both issues and pull requests. We’ll classify those records in the next stages of the workflow.

## Read the Returned JSON

The latest request returns a small, filtered set of records in JSON format. The next step is to inspect the JSON structure before extracting useful values.

### Examining a single record

The JSON response contains the returned records in the form of an array of JSON objects. Each object corresponds to one record.

A JSON object is a collection of named properties or fields (key-value pairs). Each object contains named properties that describe the repository record, including its number, title, state, labels, and update information.

Each object contains many fields, but only some are relevant to our reporting workflow.

Here’s a brief excerpt of a single JSON object in the returned response:

```json
{
    ...
    "number": 45785,
    "title": "fix: prevent install failure when CLI has authenticated with GHE",
    "user": {
        ...
    },
    "labels": [
        {
            ...
            "name": "triage",
            ...
        }
    ],
    "state": "open",
    ...
    "created_at": "2026-09-07T12:35:38Z",
    "updated_at": "2026-09-07T12:39:06Z",
    ...
    "type": null,
    ...
    "pull_request": {
        ...
    },
    ...
}
```

### Identifying the relevant fields

The fields relevant for the issue-tracking report are:

| Report field | JSON source |
| --- | --- |
| # | `number` |
| Title | `title` |
| Type | Derived from the returned JSON |
| State | `state` |
| Labels | `labels[].name` |
| Updated | `updated_at` |

The `Type` report field is different because it is a derived value rather than a directly copied field from the JSON source.

**Note:** The top-level `type` property is not used for the report’s `Type` value. We’ll determine the record type from another property in the next section.

The `labels` property contains an array of label objects, and the report will use the `name` value from each label.

### Fields We Don’t Need

We don’t need fields such as `user`, `comments`, or `created_at` for this report. Other response properties, including URLs, repository metadata, and reaction information, are also outside the report’s scope.

**Note:** The raw JSON response is richer than the final, simple report. The API returns many fields, but the report uses only a subset.

## Check Whether Each Record Is an Issue or a Pull Request

The JSON response contains both issues and pull requests. We need to classify each record before putting it into the issue-tracking report.

### Central Rule for Classifying Records

The classification rule is:

| JSON source observation | Report’s `Type` field |
| --- | --- |
| `pull_request` property is present | Pull Request |
| `pull_request` property is absent | Issue |

### Pull Request record example

Record `#45785`, shown previously, contains the `pull_request` property and is therefore a pull request:

```json
{
    ...
    "html_url": "<https://github.com/github/docs/pull/45785>",
    ...
    "number": 45785,
    ...
    "state": "open",
    ...
    "pull_request": {
        ...
    },
    ...
}
```

**Secondary Observation:** The `html_url` for a pull request record contains `/pull/`.

### Issue record example

Record `#45773` does not contain the `pull_request` property and is therefore an issue:

```json
{
    ...
    "html_url": "<https://github.com/github/docs/issues/45773>",
    ...
    "number": 45773,
    ...
    "state": "open",
    ...
}
```

**Secondary Observation:** The `html_url` for an issue record contains `/issues/`.

**Note:** This is a useful visual confirmation, but the report’s Type classification is based on the presence or absence of `pull_request`.

### Note about the top-level `type` property

**Remember:** Do not use the top-level `type` property to determine the report’s `Type` column. In the returned records we inspected, that property is `null` and does not distinguish issues from pull requests.

**Correct Rule:** The Issues endpoint can return pull requests alongside issues. Pull-request records include a `pull_request` property in the response.

### Filling in the report’s `Type` field

Determine each record’s `Type` field as follows:

1. Select one record in the JSON response.
2. Look for the `pull_request` property:
    - If it is present, the record’s `Type` is Pull Request.
    - If it is absent, the record’s `Type` is Issue.
3. Repeat this check for the records in the current 10-record response.

**Conclusion:** The report’s `Type` field is derived from the presence or absence of the `pull_request` property in each JSON record.

As each record is now classified, we can extract the selected fields and assemble the issue-tracking report.

## Build the Issue-Tracking Report

Now that the records are identified and classified, we can build the issue-tracking report.

The workflow follows this core process:

1. Take a JSON record.
2. Classify it as a pull request or issue.
3. Select the required fields.
4. Create the corresponding report row.
5. Repeat steps 1–4 for the remaining records to compile the report.

### Transforming one JSON record into a report row

Taking the previously shown record `#45785` in the JSON source as an example:

```
number         → 45785
Type (derived) → Pull Request
title          → fix: prevent install failure when CLI has authenticated with GHE
state          → open
labels         → triage
updated_at     → 2026-09-07T12:39:06Z
```

The corresponding report row will be:

| # | Type | Title | State | Labels | Updated |
| --- | --- | --- | --- | --- | --- |
| 45785 | Pull Request | fix: prevent install failure when CLI has authenticated with GHE | open | triage | 2026-09-07T12:39:06Z |

### Record-to-row transformation process

Apply the following process to transform each JSON record into its corresponding report row:

| Report field | JSON source | Process |
| --- | --- | --- |
| # | `number` | Copy the value of `number` |
| Type | Derived from `pull_request` | Apply the classification rule from the previous section |
| Title | `title` | Copy the value of `title` |
| State | `state` | Copy the value of `state` |
| Labels | `labels[].name` | Take the `name` value from each object in the `labels` array |
| Updated | `updated_at` | Copy the value of `updated_at` |

**Note:** The `labels` property is an array that can contain multiple objects. If a record contains several labels, the report should include their `name` values rather than the entire label objects.

### The issue-tracking report

After applying the record-to-row transformation process to all 10 records in the JSON response, the resulting report was:

| # | Type | Title | State | Labels | Updated |
| --- | --- | --- | --- | --- | --- |
| 45785 | Pull Request | fix: prevent install failure when CLI has authenticated with GHE | open | triage | 2026-09-07T12:39:06Z |
| 44586 | Pull Request | docs: clarify dependency graph/review enabled/default status | open | content, ready to merge, code security, needs SME, SME reviewed | 2026-09-07T07:27:09Z |
| 45773 | Issue | Wrong translation in French version of the “Manually running a workflow” article | open | content, localization | 2026-09-07T07:22:17Z |
| 45781 | Pull Request | Refine instructions for creating a branch via the branches overview | open | triage | 2026-09-07T02:16:02Z |
| 45780 | Issue | Update branch creation instructions to match current GitHub UI | open | content, triage | 2026-09-07T02:10:04Z |
| 45779 | Pull Request | Fix grammar in required profile-name note | open | triage | 2026-09-07T01:28:51Z |
| 45767 | Pull Request | docs: clarify matrix context property names are case-sensitive | open | triage | 2026-09-06T23:36:04Z |
| 45766 | Pull Request | docs: clarify path filters for changes to git submodule files | open | triage | 2026-09-06T19:16:42Z |
| 45765 | Pull Request | Add Dependabot ecosystem support for Kotlin Toolchain | open | triage | 2026-09-06T19:05:24Z |
| 45762 | Pull Request | Recommend GitHub App authentication in the ARC quickstart | open | content | 2026-09-06T07:40:18Z |

**Note:** Since this report comes from a live repository, the exact records and timestamps will change over time.

### Purpose of the report

The report is intentionally smaller than the richer API response: it keeps only the information needed to scan the current work items.

It discards properties that are irrelevant to the issue-tracking workflow and presents six pieces of information that are useful for quickly scanning repository work.

The report is now assembled. In the next section, we’ll verify its values against the API response.

## Verify the Report Against the API Response

A successful API request and a completed table are not enough; the transformed report must also be checked against the JSON source.

**Note:** The API response is the source of truth, and the report is its transformed representation. The report must therefore align completely with the API response.

### Verifying one row against its source record

Taking the previously shown record `#45785` as an example:

Its report row is:

| # | Type | Title | State | Labels | Updated |
| --- | --- | --- | --- | --- | --- |
| 45785 | Pull Request | fix: prevent install failure when CLI has authenticated with GHE | open | triage | 2026-09-07T12:39:06Z |

The corresponding JSON source record is:

```json
{
    "number": 45785,
    ...
    "title": "fix: prevent install failure when CLI has authenticated with GHE",
    ...
    "labels": [
        {
            "name": "triage"
        }
    ],
    ...
    "state": "open",
    ...
    "updated_at": "2026-09-07T12:39:06Z",
    ...
    "pull_request": {
        ...
    }
}
```

Observe that every report value can be traced to the JSON source:

```
#       ← number
Type    ← pull_request presence
Title   ← title
State   ← state
Labels  ← labels[].name
Updated ← updated_at
```

### Procedure for verifying all 10 rows

Verify every report row against its corresponding JSON record as follows:

1. Select a report row.
2. Locate the corresponding record in the API response by matching the report’s `#` with the record’s `number`.
3. Compare the report’s `Type` with the presence or absence of `pull_request`.
4. Compare the `Title`, `State`, `Labels`, and `Updated` values with the corresponding source data.
5. Repeat for the remaining rows.
6. Confirm that the report rows remain in the same order as the API response.

**Note:**

- Each report row corresponds to a single record in the JSON source. For example: report row `#45785` ↔︎ source record `"number": 45785`.
- Ensure that the report rows are also arranged in descending order of the `updated_at` values. Verification should cover both the data and the transformation behavior.

### Handling a mismatch

If a report value does not match the source response:

1. Return to the corresponding JSON record.
2. Correct the extracted value or classification.
3. Recheck the report row against the source.

**Note:** Do not consider the report complete until any mismatches with the source have been corrected.

### Conclusion

The report is verified when every row matches its corresponding API record and the report preserves the intended ordering.

At the time of testing, the generated report matched the live API response.

The report is now verified against the API response.

The workflow covered retrieval, transformation, and validation of repository data.

## What You Learned

This workflow demonstrated more than retrieving data from a GitHub endpoint. It showed how a useful API workflow moves from raw data to a verified, human-readable result:

1. **Retrieve:** Use a REST endpoint to obtain repository data.
2. **Control:** Use query parameters to shape the size, contents, and ordering of the returned dataset.
3. **Inspect:** Read the JSON source structure to identify the properties relevant to the reporting task.
4. **Classify:** Distinguish different record types using reliable characteristics of the response.
5. **Extract:** Select only the fields needed for the intended output.
6. **Transform:** Turn the richer data into a compact issue-tracking report.
7. **Verify:** Compare the transformed report with the API response before considering it complete.

### Key Lesson

A useful API workflow involves more than successfully retrieving JSON. The response often contains more information than the final output requires. The useful work begins when you:

- **Interpret** the response
- **Select** the data that matters
- **Transform** it into a useful representation
- **Verify** the result against its source

In this workflow, the API response served as the source of truth, while the report was its deliberately selected and transformed representation. Apply the same pattern to other API-driven reporting and data workflows.

## Next Steps

Once the basic retrieve-and-transform process is clear, you can extend the reporting workflow.

Possible extensions to the workflow include:

- **Try another repository**
    
    Apply the same workflow to another public repository. This step changes only the data source without changing the fundamental workflow.
    
- **Experiment with query parameters**
    
    Experiment with other supported query parameters to create different filtered and sorted views of the repository’s records.
    
- **Explore pagination**
    
    Learn to work with datasets larger than a single response page by exploring GitHub’s pagination parameters to retrieve additional pages of results.
    
- **Automate the report**
    
    Once the manual workflow is clear, you can automate the data extraction and report generation process with a script or application.
    

These extensions build on the same underlying pattern: retrieving data, interpreting its structure, transforming it, and verifying the result.

## Sources

- GitHub Docs: REST API endpoints for issues — Used to verify the Issues endpoint, supported headers and query parameters, response behavior, and the Issue/Pull Request distinction.
- The example report and response values shown in this article were produced from hands-on Postman testing of the `github/docs` repository. Because the repository is live, those observed records and timestamps can change over time.
