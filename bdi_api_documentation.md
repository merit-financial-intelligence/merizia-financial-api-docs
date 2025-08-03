# Merizia Financial Statement API — Client API Documentation

Welcome to the **Merizia Financial Statement API**. This two-step API lets you enqueue a bank statement for analysis and then poll for structured financial insights once processing completes.

---

## **Base URL**

```
https://merizia-bdi.up.railway.app
```

---

## **Authentication**

All endpoints require a Bearer token in the `Authorization` header:

```
Authorization: Bearer YOUR_CLIENT_TOKEN
```

You will receive your **unique API token** from Merizia during onboarding.

---

## **Endpoints**

### 1. Enqueue Statement for Analysis

```
POST /enqueue-statement/
Content-Type: multipart/form-data
Auth:   Bearer Token
``` 

**Request Parameters:**

| Field           | Type       | Required | Description                                          |
| --------------- | ---------- | -------- | ---------------------------------------------------- |
| `file`          | File (PDF) | Yes      | Bank statement PDF                                   |
| `password`      | String     | No       | PDF password if locked (default: empty)              |
| `min_months`    | Integer    | No       | Minimum months for recurring detection (default = 3) |
| `exclude_below` | Integer    | No       | Exclude transactions below this amount (default = 50000) |

**Response (200):**

```json
{ "job_id": "<uuid>", "status": "queued" }
```

---

### 2. Check Job Status & Retrieve Results

```
GET /jobs/{job_id}
Auth: Bearer Token
```

**Path Parameter:**

| Name     | Type   | Description                          |
| -------- | ------ | ------------------------------------ |
| `job_id` | String | The UUID returned by the enqueue call |

**Response:**

- **Queued** (200):
  ```json
  { "status": "queued" }
  ```

- **Done** (200):
  ```json
  {
    "status": "done",
    "result": {
      "month_analysis_info": { "start_date": "2024-01-01", "end_date": "2024-06-30" },
      "salary_analysis": { "monthly_average": 150000, "stability_score": "High" },
      "cash_flow": { "total_inflow": 1200000, "total_outflow": 1100000 },
      "loan_repayments": { "monthly_average": 25000 },
      "loan_disbursements": { "latest_amount": 100000 },
      "gambling_activity": { "detected": false },
      "recurring_patterns": [
        { "amount": 50000, "months": 5, "description": "Consistent transfer" }
      ]
    }
  }
  ```

- **Error** (400, 401, 404, 500):
  ```json
  { "status": "error", "error": "Error message or HTTP status detail" }
  ```

---

## **Examples**

### Python (requests)

```python
import time
import requests

API_TOKEN = "YOUR_CLIENT_TOKEN"
BASE_URL = "https://merizia-bdi.up.railway.app"
HEADERS = {"Authorization": f"Bearer {API_TOKEN}"}

# 1) Enqueue
with open("statement.pdf", "rb") as f:
    files = {"file": f}
    data = {"password": "", "min_months": 3, "exclude_below": 50000}
    resp = requests.post(
        f"{BASE_URL}/enqueue-statement/", headers=HEADERS, files=files, data=data
    )
    resp.raise_for_status()
    job_id = resp.json()["job_id"]
    print(f"Job queued: {job_id}")

# 2) Poll for results
interval = 5  # seconds
timeout = 300  # seconds
elapsed = 0
while elapsed < timeout:
    resp = requests.get(f"{BASE_URL}/jobs/{job_id}", headers=HEADERS)
    resp.raise_for_status()
    payload = resp.json()
    status = payload.get("status")
    if status == "queued":
        print(f"Still queued...({elapsed}s)")
        time.sleep(interval)
        elapsed += interval
    elif status == "done":
        insights = payload["result"]
        print("Insights:")
        from pprint import pprint; pprint(insights)
        break
    else:
        print(f"Error: {payload.get('error')}")
        break
```

### JavaScript (Fetch API)

```javascript
const API_TOKEN = "YOUR_CLIENT_TOKEN";
const BASE_URL = "https://merizia-bdi.up.railway.app";

// 1) Enqueue
const form = new FormData();
form.append("file", yourFile);
form.append("password", "");
form.append("min_months", "3");
form.append("exclude_below", "50000");

const enqueueResp = await fetch(`${BASE_URL}/enqueue-statement/`, {
  method: 'POST',
  headers: { 'Authorization': `Bearer ${API_TOKEN}` },
  body: form
});
const { job_id } = await enqueueResp.json();
console.log('Job queued:', job_id);

// 2) Poll for results
let status;
do {
  await new Promise(r => setTimeout(r, 5000));
  const statusResp = await fetch(`${BASE_URL}/jobs/${job_id}`, {
    headers: { 'Authorization': `Bearer ${API_TOKEN}` }
  });
  const payload = await statusResp.json();
  status = payload.status;
  console.log('Status:', status);
  if (status === 'done') {
    console.log('Result:', payload.result);
    break;
  }
} while (status === 'queued');
```

---

## **Error Codes**

| HTTP Status | Meaning                                                |
| ----------- | ------------------------------------------------------ |
| `400`       | Invalid document or parameters                         |
| `401`       | Unauthorized or missing token                          |
| `404`       | Job ID not found                                       |
| `500`       | Internal server error — contact Merizia support         |

---

## **Health Check**

Railway pings:

```
GET /health
```

returns `{ "status": "ok" }` immediately.

---

## **Support**

For API access, token setup, or troubleshooting:

* Email: **[aije@merizia.com](mailto:aije@merizia.com)**

---

> **Note:** Jobs are persisted in Redis. If you redeploy or run multiple workers, all will share the same job state.
