# Business Requirements Document (BRD)
## Holocron Command Center — Production Estimation Platform

**Document Version:** 1.0  
**Date:** February 25, 2026  
**Project:** Holocron V1  
**Prepared For:** Engineering Team  
**Authority Documents:**  
- Holocron_IPR_022026_v1.md (Product Vision)  
- bimteq_command_center_explanation_022026_v1.md (Design Reference)  
- bimteq-command-center_022026_v1.html (Interface Mockup)

---

## 1. Executive Summary

### 1.1 Purpose
This BRD translates the Holocron Internal Press Release and Command Center design documentation into actionable engineering requirements. It defines the functional and non-functional requirements for V1, incorporating stakeholder feedback from the Q&A clarifications.

### 1.2 Product Vision
Holocron eliminates the document hunt and tribal knowledge bottleneck in HVAC estimation by providing estimators with organized project documents, HVAC-relevant content navigation, client-specific rules access, and automated conflict detection — reducing the 15-20% rework rate and transforming 2-hour spec reviews into 30-minute sessions.

### 1.3 Core User Groups

| Role | Primary Function | V1 Scope |
|------|-----------------|----------|
| Jr Estimator | Spec review, takeoff execution (Pakistan) | ✓ Full access |
| Sr Estimator | Complex takeoffs, mentorship | ✓ Full access + QC override |
| QC Engineer | Post-takeoff validation (India + Pakistan) | ✓ QC panel access |
| Coordinator | Project intake, assignment, tracking | ✓ Dashboard + assignment |
| Manager | Production oversight, metrics | ✓ Admin/Ops panel |
| Account Manager | Client SOP management | ⚠ V2 scope (MRSM excluded) |

### 1.4 Out of Scope (V1)
- **MRSM integration:** Basis Board workflow deferred to V2
- **Pricing functionality:** Holocron is quantity-only; cost alternates are explicitly out of scope
- **Highlighted PDF tooling:** Manual creation required, upload only
- **QuoteSoft deep integration:** V1 uses export validation only
- **Mobile interface:** Desktop-only workflow
- **Automated takeoff:** QuoteSoft remains manual

---

**Navigation:**  
Next: [System Architecture Overview](02_System_Architecture.md)