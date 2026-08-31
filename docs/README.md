# Precise High Bleeding Risk Platform

[![HL7 FHIR R4](https://img.shields.io/badge/FHIR-R4-green.svg)](https://hl7.org/fhir/R4/)
[![SMART on FHIR](https://img.shields.io/badge/SMART-v2.0.0-orange.svg)](https://smarthealthit.org/)
[![Python Version](https://img.shields.io/badge/python-3.11%20%7C%203.12-blue.svg)](https://www.python.org/)
[![Docker](https://img.shields.io/badge/docker-ready-blue.svg)](https://www.docker.com/)
[![Standards](https://img.shields.io/badge/standards-IEC%2062304%20%7C%20ISO%2014971-blueviolet.svg)](https://www.iso.org/)

## Project Overview

An enterprise-grade, regulatory-compliant **SMART on FHIR Clinical Decision Support (CDS) platform** designed specifically for interventional cardiologists. It evaluates post-percutaneous coronary intervention (PCI) bleeding risk using the validated **PRECISE-HBR** scoring model and provides interactive **Bleeding vs. Thrombosis Risk Trade-off Analysis** based on the ARC-HBR model.

## Key Features

* **PRECISE-HBR Scoring Model**: Validated mechanism calculating 1-year major bleeding risk (BARC 3–5) to guide dual antiplatelet therapy (DAPT) duration decisions.
* **Bleeding vs. Ischemic Trade-Off Engine**: Incorporates the ARC-HBR trade-off model with excess mortality weighting (1.9× mortality ratio) for tailored clinical decisions.
* **Interactive What-If Simulation**: Clinicians can adjust clinical parameters and observe dynamic risk recalculations in real time[cite: 1].
* **Automated FHIR Data Ingestion**: Seamlessly extracts patient demographics, laboratory values (LOINC), clinical conditions (ICD-10/SNOMED CT), and medications via FHIR R4 APIs.
* **CDS Hooks v1.2 Support**: Context-aware alerts integrated during EHR workflows (`medication-prescribe`, `patient-view`).
* **Medical Device Quality & Safety**: Fully aligned with **IEC 62304:2006+A1:2015** (Class C Software Safety Classification) and **ISO 14971:2019** (Risk Management) standards.
* **ePHI Redaction & Security Hardening**: Strict allowlist-based logging filter preventing protected health information leakage into stdout/audit logs (HIPAA §164.312(b)).

## Technical Architecture

```text
Precise-High-Bleeding-Risk-Platform/
├── APP.py                      # Application factory and security header pipeline
├── extensions.py               # Shared extensions (Limiter, CSRF)
├── version.py                  # Single source of truth for semantic versioning
├── routes/
│   ├── auth_routes.py          # SMART on FHIR OAuth2 / PKCE launch and callback
│   ├── web_routes.py           # Web UI controllers (/main, /docs, /report-issue)
│   ├── api_routes.py           # REST APIs (/api/calculate_risk, /api/feedback)
│   ├── tradeoff_routes.py      # Trade-off calculation and config endpoints
│   └── hooks.py                # CDS Hooks endpoints and discovery services
├── services/
│   ├── precise_hbr_calculator.py      # Core PRECISE-HBR mathematical scoring logic
│   ├── tradeoff_model_calculator.py # Trade-off scoring and predictor evaluation
│   ├── fhir_client_service.py       # Resilient FHIR client with Circuit Breaker
│   ├── fhir_normalizer.py           # Canonical data models and normalization layer
│   ├── condition_checker.py         # Rule engine for ICD-10, SNOMED, and medication matching
│   ├── unit_conversion_service.py   # Lab value conversion and CKD-EPI eGFR calculation
│   ├── audit_logger.py              # 5W-compliant structured audit logging
│   └── circuit_breaker.py           # Server fault tolerance pattern
├── utils/
│   ├── input_validator.py      # SSRF, URL, and parameter sanitization
│   ├── patient_context.py      # BOLA protection and session validators
│   └── logging_filter.py       # Strict ePHI regex and structural scrubber
├── config/
│   ├── cdss_config.json        # LOINC/SNOMED definitions and clinical coefficients
│   └── cds-services.json       # CDS Hooks discovery metadata
├── templates/                  # Jinja2 templates
└── static/                     # CSS, JS modules, assets, and documentation resources
```

## Getting Started

### Local Development Setup

1. Clone the repository
   ```bash
   git clone [https://github.com/Justine11289/Precise-High-Bleeding-Risk-Platform.git](https://github.com/Justine11289/Precise-High-Bleeding-Risk-Platform.git)
   cd Precise-High-Bleeding-Risk-Platform
   ```
2. Set up virtual environment
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   pip install --upgrade pip
   pip install -r requirements.txt
   ```
3. Configure environment variables
   ```bash
   # Copy the template and configure your local settings
   cp config/local.env.template .env
   # Key environment variables
   FLASK_ENV=development
   FLASK_DEBUG=true
   FLASK_SECRET_KEY=your-secure-random-secret-key
   PORT=9000
   SMART_CLIENT_ID=your-smart-client-id
   SMART_REDIRECT_URI=http://localhost:9000/callback
   SKIP_ID_TOKEN_VALIDATION=true  # Use only in local development/sandboxes
   ```
   Adjust settings in `.env` (e.g., `PORT=9000`, `SMART_CLIENT_ID`, `SMART_REDIRECT_URI`).
4. Run the application
   ```bash
   python APP.py
   ```
   Access the UI and documentation at http://localhost:9000/docs
   
### Docker Deployment
```bash
# Development Environment
docker compose up --build

# Production Environment
docker compose -f docker-compose.prod.yml up --build -d  
```

### Testing & Clinical Verification
The test suite includes unit tests, penetration/security checks, and clinical golden-dataset verification
```bash
# Run all automated test suites
pytest

# Run security verification tests only
pytest -m security

# Run clinical algorithmic verification against golden datasets
python -m pytest tests/test_golden_dataset_csv.py tests/test_tradeoff_golden_dataset.py

# Generate test coverage report
pytest --cov=. --cov-report=html --cov-config=.coveragerc
```

## Clinical Validation & Traceability
* Primary Reference: Gragnano F, van Klaveren D, Heg D, et al. PRECISE-HBR Score for High Bleeding Risk Assessment in Patients Undergoing Percutaneous Coronary Intervention. Circulation. 2025;151:693–706.
* Trade-off Model: Costa F, Valgimigli M, et al. Assessing the Risks of Bleeding vs Thrombotic Events in Patients at High Bleeding Risk After Coronary Stent Implantation: The ARC-High Bleeding Risk Trade-off Model. JAMA Cardiology. 2021;6(4):410–419.
* Consensus Document: Urban P, Mehran R, Colleran R, et al. Defining High Bleeding Risk in Patients Undergoing Percutaneous Coronary Intervention: A Consensus Document from the Academic Research Consortium for High Bleeding Risk (ARC-HBR). Circulation. 2019;140:240–261.

## Acknowledgements & Attribution
This project is an extended and refactored edition of the PRECISEHBR_CGU research project, originally co-developed with the Chang Gung University (CGU) team
* Original Collaboration Repository: PRECISEHBR_CGU (Co-developed with Lusnaker0730 / Chang Gung University)
### Key Enhancements in This Edition
Building upon the collaborative foundation, this repository represents an independently maintained version focusing on robust EHR integration and production-ready SMART on FHIR workflows:
* Standardized EHR Launch Workflow: Streamlined the platform to focus purely on native EHR Launch flows, deprecating obsolete standalone calculator pages to provide a seamless clinical embedding experience.
* SMART on FHIR & OAuth 2.0 Hardening: Upgraded the authentication and authorization lifecycle with enhanced PKCE verification, resilient token refresh handling, and strict patient-session context validation.
* Optimized FHIR Ingestion: Improved error recovery, condition code mappings, and data normalization pipelines when communicating with live FHIR R4 servers.
* Harmonized Clinical UI: Modernized the user interface and interactive risk trade-off visualizations to ensure consistent layouts, responsive loading feedback, and intuitive EHR navigation.

## Credits, Governance & License
* This project is an extended and refactored edition of the `PRECISEHBR_CGU` research project, originally co-developed with the Chang Gung Memorial Hospital (CGMH) team
[`PRECISEHBR_CGU`](https://github.com/Lusnaker0730/PRECISEHBR_CGU)
* Delopment: Yu-Ying Lu, Tzu-Ting Huang and Chang Gung Memorial Hospital (CGMH).
* Upstream Blueprint Attribution: Portions of this platform's architecture, launch workflow orchestration, and core components are derived from the open-source [SMART Launcher v2](https://github.com/smart-on-fhir/smart-launcher-v2) (Copyright © Boston Children's Hospital).
* Industry Standards: HL7 FHIR Standard (v4.0.1), SMART App Launch Framework (v2.0.0), OAuth 2.0 (RFC 6749), OpenID Connect Core 1.0.
* License: Distributed under the terms of the [MIT License](LICENSE).

Copyright © 2026 Tzu-Ting Huang, CGMH. All rights reserved.
