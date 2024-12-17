Flow Diagram

<img width="861" alt="image" src="https://github.com/user-attachments/assets/506982ef-29b2-4b26-9128-86b5b81acbea" />

ERD

<img width="1097" alt="image" src="https://github.com/user-attachments/assets/06dfecce-7b3d-4360-bbad-92ea06f531ad" />

Here are the **updated table structures** and **sample data** for all tables, incorporating the enhancements with clear relationships and relevant fields.

---

## **1. `PARTNERS` Table**
Stores external partner details.

| Column         | Type         | Description                   |
|----------------|--------------|-------------------------------|
| partner_id     | INT (PK)     | Unique identifier for partner |
| name           | VARCHAR      | Partner name                  |
| protocol_type  | ENUM         | `JSON`, `ISO8583`, `SOAP`     |
| endpoint_url   | VARCHAR      | Service endpoint              |

**Sample Data:**

| partner_id | name            | protocol_type | endpoint_url                     |
|------------|-----------------|---------------|----------------------------------|
| 1          | Service A       | JSON          | https://api.servicea.com/inquiry |
| 2          | Service B       | JSON          | https://api.serviceb.com/inquiry |
| 3          | Bank ISO        | ISO8583       | tcp://bankiso.com:8583           |
| 4          | Utility SOAP    | SOAP          | https://utility.com/wsdl         |

---

## **2. `PROCESS_TYPES` Table**
Defines different process types (Inquiry, Payment, Reversal).

| Column         | Type         | Description                      |
|----------------|--------------|----------------------------------|
| process_id     | INT (PK)     | Unique identifier for process    |
| name           | VARCHAR      | Process name (Inquiry, Payment)  |

**Sample Data:**

| process_id | name          |
|------------|---------------|
| 1          | Inquiry       |
| 2          | Payment       |
| 3          | Reversal      |

---

## **3. `TRANSFORMATION_RULES` Table**
Holds rules for transforming external responses into internal payloads.

| Column         | Type          | Description                               |
|-----------------|---------------|-------------------------------------------|
| rule_id        | INT (PK)      | Unique identifier for rule                |
| partner_id     | INT (FK)      | References `PARTNERS` table               |
| process_id     | INT (FK)      | References `PROCESS_TYPES` table          |
| external_key   | VARCHAR       | Key from external response                |
| internal_key   | VARCHAR       | Key in the internal payload               |
| transformation | VARCHAR       | Logic for transformation (e.g., rename)  |

**Sample Data:**

| rule_id | partner_id | process_id | external_key    | internal_key  | transformation |
|---------|------------|------------|-----------------|---------------|----------------|
| 1       | 1          | 1          | description     | Layanan       | rename         |
| 2       | 1          | 1          | amount          | Jumlah        | rename         |
| 3       | 2          | 1          | name            | Layanan       | rename         |
| 4       | 2          | 1          | nominal         | Jumlah        | rename         |
| 5       | 3          | 2          | FIELD_4         | Jumlah        | map ISO field  |
| 6       | 4          | 1          | soap:bill_name  | Layanan       | xpath          |
| 7       | 4          | 1          | soap:amount     | Jumlah        | xpath          |

---

## **4. `WORKFLOW_SEQUENCE` Table**
Defines the process dependencies and valid transitions.

| Column          | Type         | Description                                  |
|-----------------|--------------|----------------------------------------------|
| workflow_id     | INT (PK)     | Unique workflow identifier                   |
| partner_id      | INT (FK)     | References `PARTNERS` table                  |
| current_process | INT (FK)     | References `PROCESS_TYPES` (current process) |
| next_process    | INT (FK)     | References `PROCESS_TYPES` (allowed process) |
| mandatory       | BOOLEAN      | Whether the current process is required      |

**Sample Data:**

| workflow_id | partner_id | current_process | next_process | mandatory |
|-------------|------------|-----------------|--------------|-----------|
| 1           | 1          | 1 (Inquiry)     | 2 (Payment)  | TRUE      |
| 2           | 1          | 2 (Payment)     | 3 (Reversal) | TRUE      |
| 3           | 2          | 1 (Inquiry)     | 3 (Reversal) | FALSE     |

---

## **5. `PROCESS_STATES` Table**
Tracks the current status of each process execution for a transaction.

| Column          | Type         | Description                              |
|-----------------|--------------|------------------------------------------|
| state_id        | INT (PK)     | Unique identifier for the state          |
| transaction_id  | VARCHAR      | Unique transaction identifier            |
| partner_id      | INT (FK)     | References `PARTNERS` table              |
| process_id      | INT (FK)     | References `PROCESS_TYPES` table         |
| status          | ENUM         | `PENDING`, `SUCCESS`, `FAILED`           |
| created_at      | TIMESTAMP    | Process start time                       |
| updated_at      | TIMESTAMP    | Process update time                      |

**Sample Data:**

| state_id | transaction_id | partner_id | process_id | status    | created_at          | updated_at          |
|----------|----------------|------------|------------|-----------|---------------------|---------------------|
| 1        | TX12345        | 1          | 1          | SUCCESS   | 2024-08-01 10:00:00 | 2024-08-01 10:02:00 |
| 2        | TX12345        | 1          | 2          | SUCCESS   | 2024-08-01 10:03:00 | 2024-08-01 10:05:00 |
| 3        | TX12345        | 1          | 3          | PENDING   | 2024-08-01 10:06:00 | NULL                |
| 4        | TX54321        | 2          | 1          | SUCCESS   | 2024-08-02 09:00:00 | 2024-08-02 09:02:00 |
| 5        | TX54321        | 2          | 3          | SUCCESS   | 2024-08-02 09:05:00 | 2024-08-02 09:07:00 |

---

## **6. `PROTOCOL_DETAILS` Table**
Holds protocol-specific configurations for partners using **ISO8583** or **SOAP/WSDL**.

| Column         | Type          | Description                                |
|-----------------|---------------|--------------------------------------------|
| protocol_id    | INT (PK)      | Unique identifier for protocol configuration |
| partner_id     | INT (FK)      | References `PARTNERS` table                |
| protocol_type  | ENUM          | Protocol type (`ISO8583`, `SOAP`, `JSON`)  |
| details        | TEXT          | Protocol-specific details/configuration    |

**Sample Data:**

| protocol_id | partner_id | protocol_type | details                             |
|-------------|------------|---------------|------------------------------------|
| 1           | 3          | ISO8583       | {"FIELD_4": "amount", "FIELD_7": "date"} |
| 2           | 4          | SOAP          | "<wsdl><operation>Inquiry</operation></wsdl>" |

---

## **Summary of Tables**

1. **PARTNERS**: External partner information.
2. **PROCESS_TYPES**: List of all process types.
3. **TRANSFORMATION_RULES**: Dynamic mapping rules for transforming responses.
4. **WORKFLOW_SEQUENCE**: Defines allowed process transitions and mandatory dependencies.
5. **PROCESS_STATES**: Tracks the current state of each process for a transaction.
6. **PROTOCOL_DETAILS**: Stores protocol-specific configurations like ISO8583 fields or WSDL details.

This structure ensures **dynamic configurability**, **process validation**, and **scalable response handling** for any external service or protocol.
