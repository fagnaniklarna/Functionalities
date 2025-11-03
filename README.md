# 🧩 Klarna Acquiring Partner Integration Availability  
### Model Specification for UI Generation

---

## 🧱 Purpose  
This model describes the **Wikibase data structure** that powers the *Sub-PSP* and *Scope* sections of Klarna’s **Acquiring Partner Project Management UI**.

The UI must allow users to:
- View which Acquiring Partner sub-PSPs and gateways are controlled by Klarna  
- See which Partner Integrations are available via an Acquiring Partner, Sub-PSP, or Gateway  
- See which Functionalities each Acquiring Partner Integration supports per Integration Method  
- Track **Lifecycle Status** and **Planned Live Date** per Functionality  
- Identify required functionalities that are missing or not implemented  
- Edit and save these values directly to Wikibase via the API  

---

## 🧩 Core Entities  

### 1️⃣ Acquiring Partner (`Q60167`)  
Represents the company or integrator that connects to Klarna (e.g., Paytrail, JP Morgan, Nexi).  

**Example:**  
Paytrail (Q1970909)
instance of → Acquiring Partner
accountable → LP Win Top MoRs Black (Org Unit)
has integration (P4497) → “PayTrail Payment API (Embedded) – Server Side Partner Integration (Q2036949)”
planned go-live (P2043) → 2025-12-01

yaml
Copy code
Each Acquiring Partner may have multiple **Partner Integrations**, one per Integration Pattern (Server-Side, Web SDK, etc.).

---

### 2️⃣ Partner Integration (`Q2031136`)  
Represents a specific technical integration for an Acquiring Partner, or its Sub-PSP or Gateway, scoped to one Integration Pattern.  

**Example:**  
PayTrail Payment API (Embedded) – Server Side Partner Integration (Q2036949)
instance of → Partner Integration
integration pattern (P496) → Server-Side
supports (P460) → multiple Functionality items

yaml
Copy code

Each `supports` statement has qualifiers:
- `lifecycle status (P4430)` → e.g., *General Availability*  
- `planned live date (P2043)` → e.g., *2025-09-01*  

Each Partner Integration corresponds to one row of data per Integration Pattern.

---

### 3️⃣ Functionality (`Q28107`)  
Represents a discrete technical capability that Klarna provides.

**Example 1:**  
Read the state of a payment request (Q2036543)
instance of → Functionality
accountable → LP GKD Payment API (Org Unit)
channel → Server-Side
has lifecycle status → General Availability
integration requirement level → required

markdown
Copy code

**Example 2:**  
Support Tokenization on Authorize (Q2036551)
integration requirement level → recommended
channel → Server-Side
has lifecycle status → General Availability
launch release (P4504) → r2511 (Release Q2019614)

yaml
Copy code

Some functionalities are required; others are recommended or optional.

---

### 4️⃣ Integration Method (`Q28136`)  
Represents the integration channel used by the partner.  

Typical values:
- **Server-Side** (`Q1988840`)
- **Web SDK** (`Q1988696`)
- **Mobile SDK** (`Q1644812`)  

Referenced by Partner Integrations via `integration pattern (P496)`.

---

### 5️⃣ Lifecycle Status (`Q58914`)  
Enum values for rollout readiness:  
- **General Availability** (`Q1989271`)  
- **Not Applicable** (`Q59131`)  
- **Early Release (Beta)** (`Q2010296`)  
- **Not available** (`Q1658534`)  
- **Deprecated** (`Q78116`)  

Used as a qualifier on `supports (P460)`.

---

## ⚙️ Relationship Model  

Acquiring Partner (Q60167)
└── has integration (P4497) → Partner Integration (Q2031136)
├── integration pattern (P496) → Integration Method (Server-Side / Web SDK / Mobile SDK)
└── supports (P460) → Functionality (Q28107)
├── qualifier: lifecycle status (P4430)
└── qualifier: planned live date (P2043)

yaml
Copy code

---

## 🧠 UI Requirements  

### 1. Partner-level Navigation  
- Dropdown selector: Choose Acquiring Partner (`Q60167`)  
- Query all Partner Integrations (`Q2031136`) where `belongs to = selected partner`

---

### 2. Matrix-Style View  
- **Rows:** Functionalities (`Q28107`)  
- **Columns:** Integration Patterns (Server-Side, Web SDK, Mobile SDK)  
- **Cells:** Lifecycle Status + Planned Live Date  

Each cell corresponds to one *(Partner Integration → Functionality)* edge.

---

### 3. Editing Flow  
- User edits **Lifecycle Status** or **Planned Live Date** in a cell.  
- UI updates or creates a `supports (P460)` statement with qualifiers:  
  - `lifecycle status (P4430)`  
  - `planned live date (P2043)`  
- If a Partner Integration node doesn’t exist for a pattern, it is created automatically.

---

### 4. “Required but Not Implemented” View  
UI must highlight or list:  
- Functionalities where `integration requirement level = required`  
- **No** `supports` statement exists for the selected Partner Integration  
- Or the `lifecycle status` qualifier ≠ “Available”  

This view helps PSTs identify which mandatory capabilities the partner still lacks.

---

### 5. Optional Filters  
- Filter by **Lifecycle Status** (e.g., show only *In Progress*)  
- Filter by **Integration Requirement Level** (*required / recommended / optional*)  
- Sort by **Planned Live Date**

---

## 🧩 Example SPARQL Query for UI  

```sparql
SELECT ?partnerIntegration ?integrationPatternLabel ?functionality ?functionalityLabel ?statusLabel ?plannedDate ?requirementLevelLabel WHERE {
  ?partnerIntegration wdt:P4497 wd:Q1970909 .        # Paytrail
  ?partnerIntegration wdt:P496 ?integrationPattern .
  ?partnerIntegration p:P460 ?stmt .
  ?stmt ps:P460 ?functionality .
  OPTIONAL { ?stmt pq:P4430 ?status . }
  OPTIONAL { ?stmt pq:P2043 ?plannedDate . }
  OPTIONAL { ?functionality wdt:Pxxx ?requirementLevel . }  # integration requirement level
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
🧩 Example Visual Mapping
Functionality	Server-Side	Web SDK	Mobile SDK
Read payment state	✅ Available (Live: 1 Sep 2025)	❌ Not available	❌ Not available
Tokenization on Authorize	🚧 In progress (Live: 1 Mar 2026)	❌ Not available	❌ Not available
Refund processing	✅ Available	✅ Available	🕓 Planned (Q2 2026)

⚠️ Required-but-missing functionalities appear with a warning indicator.

🎯 Expected UI Outputs
Framework: React + TypeScript

Features:

Partner selection dropdown

Inline editable matrix (dropdown for Lifecycle Status, date picker for Planned Live Date)

“Required but not implemented” tab or badge

Live save via Wikibase REST API

Filterable by any field

🧩 Summary
This project powers the Sub-PSP and Scope sections of Klarna’s Acquiring Partner Project Management UI using React + TypeScript.
It enables Klarna teams to manage Partner Integrations, track rollout progress, and identify missing or required functionalities directly through a unified Wikibase-powered interface.
