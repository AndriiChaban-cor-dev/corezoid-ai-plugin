# Smart API Connector registration

After a successful positive test, send ONE task to the registration receiver. The receiver
(Simulator side) creates or updates the Smart API actor, its Service, Path, accounts and
dashboards. The plugin needs no knowledge of Simulator — only the receiver process id,
resolved fresh every time.

## 1. Find the receiver (always in the current workspace, by names — no hardcoded ids)

Nothing here is environment-specific or hardcoded — resolve all of it by name, in the
workspace the user is currently working in:

- **Project** — `cz-structure` `list-projects`, match `short_name = "smart-api"`.
- **Stage** — `cz-structure` `list-stages` inside that project, match `short_name =
  "production"` — always production, regardless of which stage the connector itself is
  being built on.
- **Alias** — `cz-structure` `list-aliases` inside that project/stage with `short_name =
  "api-gw-create-smart-api"` (see `/corezoid-alias-manager`) → its `obj_to_id` is the
  receiver process.
- **Receiver process** — the `obj_to_id` returned for that alias → its `conv_id`.

If the project, the stage, the alias, or access to any of them is missing — **never block**:

> The connector is ready and tested. Smart API was not registered: <what is missing>.

Finish the skill normally — this is not an error.

## 2. Call the receiver

`run-task` into the receiver process resolved in Step 1 (by `conv_id`; directly by `alias` +
`project` + `stage` once the plugin supports that). Synchronous — wait for the Reply.

No right to run tasks in the receiver process — tell the user which access is needed (see
`/corezoid-access`) and finish without blocking: the connector stays ready, Smart API is not
registered.

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
