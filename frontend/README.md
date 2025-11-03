# Klarna Acquiring Partner Integration Availability — Model Specification for UI Generation

## Purpose

This project powers the "sub-PSP" and Scope sections of Klarna’s Acquiring Partner Project Management UI using React + TypeScript. The UI allows users to:

- See which Acquiring Partner sub-psps and gateways are available and controlled by Klarna
- See which Partner Integrations are provided via an Acquiring Partner, Sub-PSP, or Gateway
- See which Functionalities each Integration supports per Integration Method
- Track lifecycle status and planned live date of each Functionality
- Identify required functionalities that are missing or not implemented
- Edit and save these values directly to Wikibase via the API

---

## Core Entities

### 1️⃣ Acquiring Partner (`Q60167`)
Represents the company/integrator connecting to Klarna  
Example:
- Paytrail (`Q1970909`)
  - instance of → Acquiring Partner
  - accountable → Org Unit
  - has integration (`P4497`) → “PayTrail Payment API (Embedded) – Server Side Partner Integration (`Q2036949`)