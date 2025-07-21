# Merizia Financial Statement API — Client API Documentation

Welcome to the **Merizia Financial Statement API**. This API allows you to extract structured financial insights from PDF bank statements via a simple, secure API.

---

## **Base URL**

```
POST https://merizia-financial-api.onrender.com/analyze-statement/
```

---

## **Authentication**

**Required Header:**

```
Authorization: Bearer YOUR_CLIENT_TOKEN
```

You will receive your **unique API token** from Merizia during onboarding.

---

## **Request Details**

| Method       | POST                  |
| ------------ | --------------------- |
| Endpoint     | `/analyze-statement/` |
| Auth         | Bearer Token (Header) |
| Content-Type | `multipart/form-data` |

### **Request Parameters**

| Field           | Type       | Required   | Description                                          |
| --------------- | ---------- | ---------- | ---------------------------------------------------- |
| `file`          | File (PDF) | Yes        | Bank statement PDF                                   |
| `password`      | String     | Optional   | PDF password if locked                               |
| `min_months`    | Integer    | Optional   | Minimum months for recurring detection (default = 3) |
| `exclude_below` | Integer    | Optional   | Exclude transactions below amount (default = 50000)  |

---

## **Example: Python Usage**

```python
import requests

url = "https://your-api-name.onrender.com/analyze-statement/"
headers = {"Authorization": "Bearer YOUR_CLIENT_TOKEN"}

files = {"file": open("statement.pdf", "rb")}
data = {
    "password": "",
    "min_months": 3,
    "exclude_below": 50000
}

response = requests.post(url, headers=headers, files=files, data=data)
print(response.status_code)
print(response.json())
```

---

## **Example: JavaScript (Fetch API)**

```javascript
const formData = new FormData();
formData.append("file", yourFile);
formData.append("password", "");
formData.append("min_months", "3");
formData.append("exclude_below", "50000");

fetch("https://your-api-name.onrender.com/analyze-statement/", {
    method: "POST",
    headers: {
        "Authorization": "Bearer YOUR_CLIENT_TOKEN"
    },
    body: formData
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error('Error:', error));
```

---

## **Sample Response Structure**

```json
{
  "month_analysis_info": {"start_date": "2024-01-01", "end_date": "2024-06-30"},
  "salary_analysis": {"monthly_average": 150000, "stability_score": "High"},
  "cash_flow": {"total_inflow": 1200000, "total_outflow": 1100000},
  "loan_repayments": {"monthly_average": 25000},
  "loan_disbursements": {"latest_amount": 100000},
  "gambling_activity": {"detected": false},
  "recurring_patterns": [
    {"amount": 50000, "months": 5, "description": "Consistent transfer"}
  ]
}
```

---

## **Error Codes**

| Status Code | Meaning                                |
| ----------- | -------------------------------------- |
| `400`       | Invalid or unsupported document        |
| `401`       | Unauthorized or missing token          |
| `500`       | Internal server error, contact support |

---

## **How to Test**

1. Use **Postman**:

   * Method: `POST`
   * Authorization: Bearer Token
   * Body: `multipart/form-data` with `file`, `password`, `min_months`, `exclude_below`
2. Use the **Python or JavaScript examples** provided above.
3. **Success** = JSON response with financial insights.

---

## **Support**

For API access, token setup, or support:

* Email: **[aije@merizia.com](mailto:aije@merizia.com)**

---

### Notes:

* Data processed securely; no data stored after processing.
