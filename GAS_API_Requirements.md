# KRA Portal API Documentation

## 1. Overview
This API is built with Google Apps Script and uses Google Sheets as the database backend.

**Base URL**
`https://script.google.com/macros/s/AKfycbyfqN_8FKN_Fmbkz4sacy51_VRIGDFTWsVwyG-OhmIcqDw-ze23H636xMC-gcTc-MsQ/exec`

**Supported Methods**
* `GET` — health check
* `POST` — main API actions

**Content Type**
`Content-Type: application/json`

## 2. Database Structure

### Sheet 1: Employees
Stores employee profile and login details.

| Column Name | Description |
|---|---|
| employee_id | Unique employee ID |
| employee_name | Employee full name |
| mobile_number | Mobile number used for login |
| passcode | Passcode used for login |
| reporting_manager | Reporting manager name |
| designation | Employee designation |
| department | Department name |
| is_active | TRUE/FALSE status |

### Sheet 2: KRA_Master
Stores assigned KRA definitions.

| Column Name | Description |
|---|---|
| kra_row_id | Unique KRA row ID |
| employee_id | Linked employee ID |
| kra_id | KRA identifier |
| kra | KRA title |
| objective | Objective description |
| measurement | Measurement / KPI |
| is_active | TRUE/FALSE status |

### Sheet 3: KRA_Submissions
Stores submitted KRA updates.

| Column Name | Description |
|---|---|
| submission_id | Unique submission ID |
| employee_id | Linked employee ID |
| mobile_number | Mobile number of employee |
| kra_row_id | Linked KRA row ID |
| status | Submitted status |
| comment | Submitted comment |
| submitted_date | Submission date |
| submitted_time | Submission time |

## 3. Authentication
Authentication is based on:
* mobile
* passcode

The API checks these values against the Employees sheet.

## 4. Endpoints

### 4.1 Health Check
**Request**
`GET /exec`

**Response**
```json
{
  "status": "success",
  "message": "KRA API is running."
}
```

### 4.2 Fetch Employee and KRA Data
**Purpose**: Loads employee details and assigned KRAs after validating mobile number and passcode.

**Request**: `POST /exec`
**Content-Type**: `application/json`

**Request Body**
```json
{
  "action": "fetch",
  "mobile": "9876543210",
  "passcode": "1234"
}
```

**Process**
* Validate mobile number and passcode from Employees
* Get employee details
* Get active KRAs from KRA_Master
* Get latest submitted status and comment from KRA_Submissions
* Return merged response

**Success Response**
```json
{
  "status": "success",
  "message": "Employee data fetched successfully.",
  "employee": {
    "employee_id": "EMP001",
    "employee_name": "Rahul Das",
    "mobile_number": "9876543210",
    "reporting_manager": "Anil Kumar",
    "designation": "Sales Executive",
    "department": "Sales",
    "is_active": true
  },
  "kras": [
    {
      "kra_row_id": "KRA001",
      "kra_id": "1",
      "kra": "Lead Follow-up",
      "objective": "Improve response speed",
      "measurement": "30 calls/day",
      "status": "Ongoing",
      "comment": "In progress",
      "submitted_date": "2026-04-14",
      "submitted_time": "10:45:00"
    }
  ]
}
```

**Error Response**
```json
{
  "status": "error",
  "message": "Invalid credentials"
}
```

### 4.3 Submit KRA Updates
**Purpose**: Saves KRA status and comment updates into KRA_Submissions.

**Request**: `POST /exec`
**Content-Type**: `application/json`

**Request Body**
```json
{
  "action": "update",
  "mobile": "9876543210",
  "passcode": "1234",
  "employee_id": "EMP001",
  "updates": [
    {
      "kra_row_id": "KRA001",
      "kra_id": "1",
      "status": "Complete",
      "comment": "Done"
    }
  ]
}
```

**Process**
* Re-validate mobile number and passcode
* Confirm employee_id belongs to authenticated employee
* Match each update with employee KRA
* Insert new records into KRA_Submissions
* Keep previous records for history

**Success Response**
```json
{
  "status": "success",
  "message": "KRA updates saved successfully.",
  "saved_count": 2,
  "skipped_count": 0,
  "skipped_items": []
}
```

**Error Response**
```json
{
  "status": "error",
  "message": "Employee ID does not match authenticated user."
}
```

## 5. Status Values
Expected string texts for status:
* Complete
* Not Started
* On Hold
* Ongoing

## 6. General Response Format
**Success**
```json
{
  "status": "success",
  "message": "Operation completed successfully."
}
```
**Error**
```json
{
  "status": "error",
  "message": "Detailed error message here"
}
```

## 7. Important Notes
* **Column Order**: The API uses dynamic header mapping, so sheet column order can change.
* **Header Names**: Header names must remain correct.
* **Submission History**: KRA_Submissions stores every save as a new row. This keeps full history.
* **KRA Master Safety**: KRA_Master is not overwritten during update.
* **Active Records**: Inactive employees or KRAs can be controlled using is_active.

## 8. Frontend Integration Example
### Fetch
```javascript
const API_URL = "https://script.google.com/macros/s/AKfycbyfqN_8FKN_Fmbkz4sacy51_VRIGDFTWsVwyG-OhmIcqDw-ze23H636xMC-gcTc-MsQ/exec";

async function loadEmployee(mobile, passcode) {
  const response = await fetch(API_URL, {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      action: "fetch",
      mobile,
      passcode
    })
  });
  return await response.json();
}
```

### Update
```javascript
async function submitKraUpdates(mobile, passcode, employeeId, updates) {
  const response = await fetch(API_URL, {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      action: "update",
      mobile,
      passcode,
      employee_id: employeeId,
      updates
    })
  });
  return await response.json();
}
```

## 9. Deployment Notes
Deploy Apps Script as:
* Type: Web app
* Execute as: Me
* Who has access: Anyone
This allows the frontend webpage to call the API.
