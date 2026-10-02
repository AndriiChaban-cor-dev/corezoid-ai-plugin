# Smart API Connector registration

After a successful positive test, send ONE task to the registration receiver. The receiver
(Simulator side) creates or updates the Smart API actor, its Service, Path, accounts and
dashboards. The plugin needs no knowledge of Simulator — only the receiver reference.

## Receiver reference

Set per environment. Current environment:

| Field | Value |
|---|---|
| `company_id` | `a58d969b-4b2f-42ce-add5-0972c4f45421` |
| `project` | Smart API project (id 623461) |
| `stage` | develop (id 623463) |
| `alias` | `api-gw-create-smart-api` |
| `callback_hash` | only for the Direct URL call |

If the reference is not configured — skip registration with a warning.

## 1. Check the project exists (never block)

Before calling, check that the project, the stage and the alias exist and are accessible
(see `/corezoid-alias-manager` for alias lookups). If any is missing or access is denied:

> The connector is ready and tested. Smart API was not registered: <reason>.

Finish the skill normally — this is not an error.

## 2. Call the receiver

**Primary — `run-task`, synchronous.** Resolve the alias to the receiver process id
(`/corezoid-alias-manager`, or `run-task` with `alias` + `project` + `stage` once the plugin
supports it), then `run-task` with the task data below and wait for the Reply.
Requires the right to run tasks in the receiver process.

**Alternative — Direct URL, asynchronous** (no run-task rights):

```
POST /api/2/json/public/@<alias>/<project>/<stage>/<company_id>/<callback_hash>
```

There is no answer: tell the user the request was accepted.

## 3. Task data

Task ref = `conv_id` of the connector: a repeated send updates the Smart API, it never
creates a duplicate.

| Field | Value |
|---|---|
| `processUuid` | system UUID of the process (same on all stages; field `uuid` of the process object). Until the plugin can read it, send empty — the receiver falls back to `processId` |
| `processId` | `conv_id` of the connector on this stage |
| `stage` | stage short name (`develop`, `production`, …) |
| `processName` | current process name |
| `providerName` | provider (Service) name from the contract |
| `host` | base URL of the Target API (resolved value of the host variable) |
| `path` | endpoint path |
| `method` | HTTP method |
| `args` | input params from the contract (names, types, required) |
| `nodeId` | canonical id of the API Call node (after `pull-process`) |
| `userId` | author — becomes Process Owner |
| `workspaceId` | Simulator workspace |
| `contract` | the confirmed contract (input, output, errors) |
| `test` | `task_ref`, `tested_at`, `result`, `http_code` from the test |

## 4. Answer

Receiver Reply (format of `reply-format.md`): `result` + `smart_api_id` — show the id to the
user. Accounts and dashboards are created asynchronously after the answer.

If the receiver returns an error or does not answer — the connector stays ready; tell the user
that Smart API was not registered and why. Do not retry more than once.
