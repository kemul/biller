Flow Diagram
<img width="861" alt="image" src="https://github.com/user-attachments/assets/506982ef-29b2-4b26-9128-86b5b81acbea" />

ERD

<img width="686" alt="image" src="https://github.com/user-attachments/assets/451d95e5-7ba9-49a5-a22e-4e9fbf23eb9a" />

* PARTNERS: Stores partner details such as protocol type and endpoint URL.
* PROCESS_TYPES: Defines process types like "Inquiry," "Payment," or "Reversal."
* TRANSFORMATION_RULES: Holds dynamic mapping rules for converting external keys to internal keys.
* PROTOCOL_DETAILS: Contains protocol-specific configurations (e.g., ISO8583 templates, WSDL details).

Example data
Here is an example data set for each table based on the provided schema:

---

### **1. `PARTNERS` Table**

| partner_id | name            | protocol_type | endpoint_url                     |
|------------|-----------------|---------------|----------------------------------|
| 1          | Service A       | JSON          | https://api.servicea.com/inquiry |
| 2          | Service B       | JSON          | https://api.serviceb.com/inquiry |
| 3          | Bank ISO        | ISO8583       | tcp://bankiso.com:8583           |
| 4          | Utility SOAP    | SOAP          | https://utility.com/wsdl         |

---

### **2. `PROCESS_TYPES` Table**

| process_id | name          |
|------------|---------------|
| 1          | Inquiry       |
| 2          | Payment       |
| 3          | Reversal      |

---

### **3. `TRANSFORMATION_RULES` Table**

| rule_id | partner_id | process_id | external_key   | internal_key  | transformation |
|---------|------------|------------|----------------|---------------|----------------|
| 1       | 1          | 1          | description    | Layanan       | rename         |
| 2       | 1          | 1          | amount         | Jumlah        | rename         |
| 3       | 2          | 1          | name           | Layanan       | rename         |
| 4       | 2          | 1          | nominal        | Jumlah        | rename         |
| 5       | 3          | 2          | FIELD_4        | Jumlah        | map ISO field  |
| 6       | 4          | 1          | soap:bill_name | Layanan       | xpath          |
| 7       | 4          | 1          | soap:amount    | Jumlah        | xpath          |

---

### **4. `PROTOCOL_DETAILS` Table**

| protocol_id | partner_id | protocol_type | details                             |
|-------------|------------|---------------|------------------------------------|
| 1           | 3          | ISO8583       | {"FIELD_4": "amount", "FIELD_7": "date"} |
| 2           | 4          | SOAP          | "<wsdl><operation>Inquiry</operation></wsdl>" |

---

### **Data Explanation:**

1. **`PARTNERS` Table:**
   - Contains details about the external partners, their protocol type (`JSON`, `ISO8583`, or `SOAP`), and their endpoint URLs.

2. **`PROCESS_TYPES` Table:**
   - Lists various types of processes such as "Inquiry," "Payment," and "Reversal."

3. **`TRANSFORMATION_RULES` Table:**
   - Holds mapping rules for transforming external payloads into internal formats:
     - `external_key`: Key from the external partner's response.
     - `internal_key`: Key in the internal payload.
     - `transformation`: Logic like `rename`, `map ISO field`, or `xpath` for SOAP.

4. **`PROTOCOL_DETAILS` Table:**
   - Contains protocol-specific configurations for partners using `ISO8583` or `SOAP`.  
   - `details` include field mappings for ISO8583 or WSDL configurations for SOAP.

---

This example data can be used to simulate the system's behavior and test the transformation logic for different protocols and partners.
