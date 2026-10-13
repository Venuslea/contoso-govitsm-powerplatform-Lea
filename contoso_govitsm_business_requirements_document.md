# BUSINESS REQUIREMENTS DOCUMENT (BRD)
## GovITSM & Hardware Asset Compliance Portal

**Target Organization:** Contoso Financial Corporation – Information Technology Systems Management Department (ITSMD)  
**Document Identifier:** `BRD-ITSMD-2026-V1`  
**Version:** 1.0.0  
**Status:** Approved for Implementation / Portfolio Baseline  
**Author:** Platform Developer  
**System Architecture:** Microsoft Power Platform (Dataverse, Model-Driven Apps, Canvas Apps, Power Automate, Power BI)

---

## 1. Executive Summary & Context

Contoso Financial Corporation operates a highly regulated internal infrastructure across multiple enterprise sites. The Information Technology Systems Management Department (ITSMD) oversees all hardware assets, network infrastructure, software provisioning, and compliance reporting. 

Currently, asset tracking and maintenance requests rely on disparate systems, leading to audit overhead, potential compliance gaps, and prolonged Mean Time to Resolution (MTTR) for critical IT incidents.

The **GovITSM & Hardware Asset Compliance Portal** provides an integrated, enterprise-grade solution built natively on Microsoft Dataverse. It unifies back-office governance, mobile field inspection, automated compliance escalations, and executive telemetry into a single security-enforced environment.

---

## 2. Business Objectives & Key Results (OKRs)

| Strategic Goal | Target Metric | Solution Capability |
| :--- | :--- | :--- |
| **Audit Readiness** | 100% real-time asset traceability | Centralized Dataverse repository with immutable system log records. |
| **Operational Efficiency** | Reduce maintenance log entry time by 60% | Mobile Canvas App equipped with barcode scanning and photo uploads. |
| **Governance Enforcement** | 0 unauthorized lifecycle stage transitions | Model-Driven Business Process Flows (BPF) with field-level security. |
| **SLA Tracking** | < 2-hour alert turnaround on high-risk flags | Automated Power Automate Cloud Flows with Microsoft Teams Adaptive Cards. |

---

## 3. Stakeholder Persona & Security Access Matrix

Access control is governed by Dataverse Security Roles and Business Unit hierarchies to align with enterprise risk standards.

```
                   +------------------------------------+
                   |     Contoso Business Hierarchy     |
                   |       (Global Root Unit)           |
                   +-----------------+------------------+
                                     |
             +-----------------------+-----------------------+
             |                                               |
+------------v------------+                     +------------v------------+
|  ITSMD Governance Unit  |                     |  Field Operations Unit  |
|  (Auditors / Managers)  |                     |  (Helpdesk Technicians) |
+-------------------------+                     +-------------------------+
```

| User Role | Target Group | Privilege Scope | Key Permissions |
| :--- | :--- | :--- | :--- |
| **IT Governance Auditor** | Internal Audit & ITSMD Leads | Business Unit | **Read/Write/Append** on Asset & Audit tables; **No Delete**. |
| **Back-Office IT Manager** | System Ops & Approvers | Organization | **Full Access** on all entities, Business Process Flow execution. |
| **Field Support Tech** | Floor Techs & On-Site Engineers | User Level | **Read** on Assets, **Create/Update** on Maintenance Logs & Photos. |
| **System Administrator** | Platform Engineering | Organization | **System Customizer / Full Admin**. |

---

## 4. System Architecture Diagram

```
+-----------------------------------------------------------------------------------+
|                            MICROSOFT DATAVERSE BACKEND                            |
|  Tables: IT Asset Hardware | Maintenance Logs | Change Requests | Audit Trail     |
|  Security: Business Unit Inheritance | Field-Level Security | Custom Security Roles|
+-----------------------------------------+-----------------------------------------+
                                          |
         +--------------------------------+--------------------------------+
         |                                |                                |
+--------v-------+               +--------v-------+               +--------v-------+
|  MODEL-DRIVEN  |               |   CANVAS APP   |               | POWER AUTOMATE |
|      APP       |               | (MOBILE FIELD) |               |  CLOUD FLOWS   |
| Back-Office    |               | Barcode Scan   |               | Escalations &  |
| Governance BPF |               | Photo Logging  |               | Adaptive Cards |
+--------+-------+               +--------+-------+               +--------+-------+
         |                                |                                |
         +--------------------------------+--------------------------------+
                                          |
                                 +--------v-------+
                                 |    POWER BI    |
                                 |   DASHBOARD    |
                                 | Real-time KPIs |
                                 +----------------+
```

---

## 5. Dataverse Entity Relationship Diagram (ERD) & Schema

### Table: `IT Asset Hardware` (`contoso_itassethardware`)

| Column Display Name | Schema Name (`contoso_`) | Data Type | Requirement Level | Notes / Business Logic |
| :--- | :--- | :--- | :--- | :--- |
| **Asset Tag Code** | `assettagcode` | Text (Primary) | Mandatory | Unique Identifier (e.g., `AST-99201`) |
| **Device Category** | `devicecategory` | Choice | Mandatory | *Server*, *Workstation*, *Network Switch*, *Mobile* |
| **Serial Number** | `serialnumber` | Text | Mandatory | Manufacturer S/N |
| **Assigned Department**| `assigneddepartment` | Text / Lookup | Optional | Organizational unit reference |
| **Compliance Status** | `compliancestatus` | Choice | Mandatory | *Compliant*, *Needs Audit*, *Decommissioned* |
| **Purchase Date** | `purchasedate` | Date Only | Mandatory | Lifecycle start date |
| **Maintenance Notes** | `maintenancenotes` | Multiple Lines | Optional | Append-only text log |

---

## 6. Functional Specifications

### 6.1 Model-Driven App: Back-Office Governance
* **App Name:** `Contoso ITSMD Administration`
* **Core Functionality:** Provides structured entity management, grid filtering, and lifecycle enforcement.
* **Business Process Flow (BPF) Lifecycle:**
  1. **Stage 1: Intake & Inspection** — Validate physical hardware specs and initial invoice data.
  2. **Stage 2: Technical Audit** — Verify patch compliance, security configuration, and assigned owner.
  3. **Stage 3: Governance Approval** — Require Manager sign-off prior to production deployment.
  4. **Stage 4: Deployed / Active** — Final active status; lock core fields against non-admin editing.

### 6.2 Canvas App: Field Technician Mobile Tool
* **App Name:** `Contoso Asset Tech Tool`
* **Target Layout:** Phone / Handheld Tablet
* **Key Features:**
  * **Barcode / QR Code Scanner:** Leverages device camera to scan physical tags.
  * **Dynamic Query:** Executes Dataverse lookup on scan.
  * **Photo Capture:** Captures condition photos and attaches them directly to the Dataverse record.

#### Power Fx Implementation (Barcode Scan & Routing):
```powerfx
Set(
    gblScannedAsset,
    LookUp(
        'IT Asset Hardwares',
        'Asset Tag Code' = BarcodeScanner1.Value
    )
);
If(
    IsBlank(gblScannedAsset),
    Notify("Asset record not found in Contoso Database.", NotificationType.Error),
    Navigate(ScreenAssetDetail, ScreenTransition.Cover)
)
```

### 6.3 Power Automate Cloud Flow: Compliance Escalation
* **Flow Name:** `Contoso-EscalateNonCompliantAsset`
* **Trigger:** Dataverse Connector — *When a row is added, modified, or deleted* (`contoso_itassethardware`).
* **Condition:** `compliancestatus == 800000001` (Needs Audit).
* **Action:** Send a Microsoft Teams Adaptive Card to the Governance Auditor group requesting immediate field review.

---

## 7. Technical Design & Deployment Steps

1. **Publisher & Solution Setup:**
   * Create Publisher: `Contoso Platform Team` (`contoso`).
   * Create Unmanaged Solution: `ContosoGovITSM`.
2. **Schema & Security Provisioning:**
   * Provision Dataverse tables and choice columns.
   * Assign `Contoso IT Governance Auditor` security role to target test users.
3. **Application Development:**
   * Build BPF process and attach to `Contoso ITSMD Administration` Model-Driven App.
   * Generate `Contoso Asset Tech Tool` Canvas App and bind Dataverse connector.
4. **Export & Solution Lifecycle Management (ALM):**
   * Export solution as **Managed** for production deployment verification.
   * Store unmanaged artifacts and solution `.zip` in the primary GitHub source repository.

---

## 8. Verification & Test Plan

- [x] **Dataverse Security Validation:** Verify Field Support Tech cannot delete asset rows.
- [x] **Barcode Lookup Test:** Validate camera scan correctly returns Dataverse entity attributes within 2 seconds.
- [x] **Business Process Flow Lock Test:** Confirm user cannot advance BPF to *Deployed* without filling mandatory audit fields.
- [x] **Flow Escalation Test:** Trigger status change to *Needs Audit* and verify Teams Adaptive Card delivery.