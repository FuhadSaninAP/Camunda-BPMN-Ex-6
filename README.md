# Camunda 8 Leave Request: Start a Process Using the REST API

Practice question for **21CSE735P – Workflow Automation Engineering**, Department of Computing Technologies, SRM Institute of Science and Technology.

A simple Leave Request process modelled in Camunda Modeler, deployed to a local Camunda 8 (C8 Run) cluster, and started with a REST call from Postman.

## Process

```
(Leave requested) → [Approve Leave] → (Leave decided)
```

| Element | Type | Detail |
|---|---|---|
| Process ID | `leave-request` | Used in the API call |
| Approve Leave | User task | Completed in Tasklist |

## Decision table (DMN)

`leave-approval-level.dmn` is an optional decision that maps the number of leave days to an approver. It is not called by the process; deploy it separately to try it.

| `days` | `approver` |
|---|---|
| ≤ 2 | `AUTO_APPROVE` |
| 3 to 5 | `MANAGER` |
| > 5 | `HR` |

## Repository contents

| File | Description |
|---|---|
| `leave-request.bpmn` | BPMN process model |
| `leave-approval-level.dmn` | DMN decision table |
| `README.md` | This file |

## Requirements

- Camunda Modeler (desktop)
- Camunda 8 Run (C8 Run) running locally on `http://localhost:8080`
- Postman (or `curl`)

## How to run

**1. Deploy.** Open `leave-request.bpmn` in Camunda Modeler, confirm the Process ID is `leave-request`, then click **Deploy** and choose your local cluster. Deploy `leave-approval-level.dmn` the same way if you want to try the decision.

**2. Start an instance.** In Postman, send:

```
POST http://localhost:8080/v2/process-instances
Content-Type: application/json
```

```json
{
  "processDefinitionId": "leave-request",
  "variables": {
    "employeeName": "Ravi",
    "days": 3
  }
}
```

Or with `curl`:

```bash
curl -X POST http://localhost:8080/v2/process-instances \
  -H "Content-Type: application/json" \
  -d '{"processDefinitionId":"leave-request","variables":{"employeeName":"Ravi","days":3}}'
```

The response includes a `processInstanceKey`. If the API asks for credentials, use Basic Auth in Postman (default local setup: `demo` / `demo`).

**3. Verify in Operate** (`http://localhost:8080/operate`):
- the `leave-request` instance is **Active**
- the token is waiting at **Approve Leave**
- the Variables panel shows `employeeName = "Ravi"` and `days = 3`

**4. Finish the instance** (optional): open Tasklist at `http://localhost:8080/tasklist`, assign the task to yourself and complete it.

## Screenshots to submit

- [ ] BPMN diagram
- [ ] Postman request and response, showing the `processInstanceKey`
- [ ] Operate showing the running instance

## Viva notes

- **`processDefinitionId` vs `processInstanceKey`:** the first is the ID of the process model (`leave-request`) and is the same for every instance. The second is the unique number of one running instance.
- **Wrong Process ID:** the API returns an error (404, no process definition found) and no instance is created.
- **Starting the process 3 times:** Operate shows 3 separate instances of the same process.

## Troubleshooting

| Symptom | Likely cause and fix |
|---|---|
| 404, no process definition found | `processDefinitionId` is misspelled or the process was not deployed. It must be exactly `leave-request`. |
| 401 Unauthorized | Add Basic Auth credentials in Postman. |
| Connection refused | C8 Run is not running or uses a different port. |
| Instance missing in Operate | Wait a few seconds for indexing, then refresh. |
