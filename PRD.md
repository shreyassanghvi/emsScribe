# emsScribe - Product Requirements Document (PRD)

**Version:** 1.0

**Date:** January 2026

**Author:** Shreyas

**Status:** Draft - Pending Review

---

## Table of Contents
1. [Executive Summary](#1-executive-summary)
2. [Problem Statement](#2-problem-statement)
3. [User Personas](#3-user-personas)
4. [Jobs to Be Done](#4-jobs-to-be-done)
5. [User Stories](#5-user-stories)
6. [Feature Requirements](#6-feature-requirements)
7. [Out of Scope](#7-out-of-scope)
8. [Assumptions](#8-assumptions)
9. [Success Metrics](#9-success-metrics)
10. [Risks & Mitigations](#10-risks--mitigations)
11. [Dependencies](#11-dependencies)
12. [Release Criteria](#12-release-criteria)
13. [Timeline & Milestones](#13-timeline--milestones)

---

## 1. Executive Summary

emsScribe is an **offline-first audio transcription and documentation tool** for emergency medical services (EMS) crews. It reduces incident charting time from 20+ minutes to approximately 5 minutes per call while ensuring HIPAA compliance and accurate medication data capture.

**Key Value Proposition:**
- **Fast:** 75% reduction in documentation time
- **Accurate:** Automated drug/dosage extraction (>80% F1)
- **Compliant:** HIPAA-first, local processing, NEMSIS-ready
- **Offline:** Works in ambulances without internet

**MVP Scope:** Audio → Transcript + Drug Extraction

**Timeline:** 4-6 weeks

**Target Users:** Paramedics/EMTs (primary), Supervisors, Admin staff

---

## 2. Problem Statement

### 2.1 Current State Pain Points

| Problem | Impact | Data |
|---------|--------|------|
| **Documentation takes too long** | Paramedics spend 3-6 hours/shift on charting | 20-30 min per call × 10-15 calls/shift |
| **Poor data quality** | Medication errors, missing dosages | 70-80% accuracy on manual entry |
| **NEMSIS compliance burden** | Manual export, validation errors | 10-15 hours/month per agency |

### 2.2 Impact on Stakeholders

**Paramedics/EMTs:**
- Administrative burden reduces quality of life
- Recall errors lead to documentation gaps
- Extended shifts due to catch-up charting

**EMS Supervisors:**
- Cannot track medication usage trends (inconsistent data)
- Manual incident review is time-prohibitive
- Training gaps hidden by poor documentation

**Billing/Admin Staff:**
- 30-40% of reports have missing medication data
- Claim denials from incomplete documentation
- NEMSIS export requires 10-15 hours/month

### 2.3 Why Existing Solutions Fail

| Solution | Problem |
|----------|---------|
| Cloud transcription (AWS, Google) | Requires internet, HIPAA risk, no medical NER |
| Traditional ePCR (ImageTrend, ESO) | Still manual entry, doesn't reduce charting time |
| Generic AI (ChatGPT) | No HIPAA, no offline, no EMS training |

---

## 3. User Personas

### 3.1 Primary: Paramedic/EMT (Field Responder)

- **Name:** Marcus Chen
- **Age:** 34
- **Experience:** 8 years as paramedic

**Goals:**
- Reduce charting time from 25 min to <10 min per call
- Create accurate reports without relying on memory
- Have documentation ready immediately after transport

**Frustrations:**
- Forgets dosages/drug names by the time he sits down to document
- Gets called back to correct incomplete reports
- 1-2 hours of catch-up charting at end of shift

**Tech Comfort:** High (uses smartphone daily, wants intuitive UX)

**Success Metric:** Charting time reduced to 5-7 min per call

---

### 3.2 Secondary: EMS Supervisor

- **Name:** Dr. Sarah Okoye
- **Age:** 52
- **Experience:** 20 years paramedic, 5 years Medical Director

**Goals:**
- Audit incident reports for protocol compliance
- Track medication usage across the system
- Maintain state NEMSIS compliance

**Frustrations:**
- Manually reviewing 10,000+ incident reports/year
- Inconsistent drug naming makes trending impossible
- NEMSIS export is a 2-3 week monthly process

**Tech Comfort:** Medium (comfortable with EMR systems, not a programmer)

**Success Metric:** NEMSIS reporting time drops from 2-3 weeks to 1-2 days

---

### 3.3 Secondary: Billing/Admin Staff

- **Name:** James Patterson
- **Age:** 41
- **Experience:** 12 years healthcare billing

**Goals:**
- Reduce data entry and validation time
- Eliminate claim denials from missing data
- Automate NEMSIS export

**Frustrations:**
- 30-40% of reports have missing medication data
- Inconsistent abbreviations don't match billing codes
- Manual NEMSIS export takes 10-15 hours/month

**Tech Comfort:** Medium-High (daily Excel, billing software)

**Success Metric:** Claim denial rate drops from 8-12% to <2%

---

## 4. Jobs to Be Done

| Job | User | Desired Outcome |
|-----|------|-----------------|
| **Complete documentation faster** | Paramedic | Transform call into report in <10 min (vs 20-30 min) |
| **Ensure accurate data capture** | All | Auto-extract medications with >80% accuracy |
| **Meet compliance requirements** | Admin/Supervisor | Generate NEMSIS-compliant reports with minimal effort |
| **Preserve patient privacy** | All | Keep all PHI local, never transmitted to cloud |
| **Support clinical quality** | Supervisor | Reliable data for trend analysis without manual review |

---

## 5. User Stories

### 5.1 P0 - MVP (Must Have for Launch)

#### Paramedic Stories

**US-001: Quick Audio Recording**
> As a paramedic, I want to start recording audio with a single tap so that I can document without looking away from the patient.

**Acceptance Criteria:**
- One-tap start/stop
- Recording indicator visible
- Auto-timeout after 90 min
- Resume if accidentally stopped

---

**US-002: View Transcription**
> As an EMT, I want to see the transcript after recording so that I can verify accuracy.

**Acceptance Criteria:**
- Transcript appears within processing time (< 2x audio duration)
- Clear display of full text
- Timestamps aligned with audio

---

**US-003: Extract Medications Automatically with RxNorm Mapping**
> As a paramedic, I want the system to highlight medications and dosages AND automatically map them to RxNorm codes so that NEMSIS v3.5 compliance is automatic.

**Acceptance Criteria:**
- Drug names recognized with >80% accuracy
- Dosages extracted with route (IV, IM, sublingual)
- Medications highlighted in transcript
- Confidence indicator for uncertain extractions (<90% confidence flagged for review)
- **Each medication mapped to RxNorm code** (using RxNav API or local lookup)
- **Both drug name and RxNorm code stored in database**
- **Mapping to RxNorm/SNOMED CT codes** with 84-93% recall, 99.5-100% precision
- NEMSIS TAC Ingredient (IN) codes used for EMS medications

---

**US-004: Offline Operation**
> As an EMT, I want the system to work with no WiFi so that I can record in rural areas.

**Acceptance Criteria:**
- Full transcription works offline
- Drug extraction works offline
- No data loss if device powered off
- Sync queued for when WiFi available

---

**US-005: Export Incident Report**
> As a paramedic, I want to export the transcript and medications so that I can attach to my official report.

**Acceptance Criteria:**
- PDF export with transcript + extracted drugs
- Includes timestamp and duration
- One-tap share (email/message)

---

**US-005a: Verify and Sign-off on Extracted Data (Human-in-Loop)**
> As a paramedic, I want to review the extracted medications and sign off before finalizing the report so that I assume responsibility for accuracy.

**Acceptance Criteria:**
- Manual review queue for extracted medications (drugs, dosages, routes)
- Ability to **edit/correct/delete** entries with full audit trail
- **Digital signature required before finalization** (drawn signature preferred per legal guidance)
- Confidence indicators displayed for AI extractions (<90% confidence flagged for review)
- Audit log captures: user ID, timestamp, action (add/edit/delete), field changed, old value, new value
- All changes logged per **HIPAA 6-year retention requirement**
- Report cannot be exported until paramedic signs off
- Support for PIN, biometric, and drawn signature methods (drawn has strongest legal weight)

---

#### Compliance Stories

**US-006: Secure Data Storage**
> As a compliance officer, I want all data encrypted at rest so that we maintain HIPAA compliance.

**Acceptance Criteria:**
- All transcripts encrypted (SQLCipher)
- No data transmitted to cloud
- Encryption verified in audit

---

**US-007: Audit Logging**
> As an admin, I want all access logged so that we can audit who viewed what.

**Acceptance Criteria:**
- Log: user ID, timestamp, action, incident ID
- Immutable log entries
- Export log for compliance review

---

### 5.2 P1 - v1.1 (Post-MVP)

| ID | Story | User |
|----|-------|------|
| US-008 | Review and correct extracted medications before export | Paramedic |
| US-009 | NEMSIS v3.5 XML export for state reporting | Admin |
| US-010 | Supervisor dashboard to review all incidents | Supervisor |
| US-011 | Station sync for enhanced processing | Paramedic |
| US-012 | Role-based access control | Admin |

### 5.3 P2 - Future

| ID | Story | User |
|----|-------|------|
| US-013 | Real-time vitals integration from monitors | Paramedic |
| US-014 | HL7 v2 export for hospital EMR | Admin |
| US-015 | AI-assisted audit flagging | Supervisor |
| US-016 | Photo/video attachment | Paramedic |

---

## 6. Feature Requirements

### 6.1 MoSCoW Prioritization

#### MUST HAVE (MVP - 4-6 weeks)

| ID | Feature | Description | Effort |
|----|---------|-------------|--------|
| F-001 | Audio Recording | Capture audio, save as compressed file | 3-4 days |
| F-002 | Whisper Transcription | Convert audio to text locally | 5-7 days |
| F-003 | Drug Extraction (NER) | Extract medications, dosages, routes | 6-8 days |
| F-004 | Text Normalization | Pre-process transcript for better NER | 3-4 days |
| F-005 | Transcript Display UI | Show transcript with highlighted entities | 4-5 days |
| F-006 | PDF Export | Generate printable incident report | 2-3 days |
| F-007 | Encrypted Database | SQLite + SQLCipher for local storage | 3-4 days |
| F-008 | Basic Authentication | PIN/biometric login | 2-3 days |
| F-009 | Audit Logging | Log all data access | 2 days |
| F-003a | Model Optimization | INT8 quantization, ONNX runtime for Med7 on tablets | 3-4 days |
| F-005a | Glove-Mode Touch Optimization | 60x60px touch targets, high contrast, minimal keyboard | 2-3 days |
| F-009a | Human Review Queue | Manual review for <90% confidence extractions | 3-4 days |
| F-009b | Digital Signature Integration | Drawn/PIN/biometric signature capture | 2-3 days |

**Total MVP Effort:** ~40-52 days (1 developer)

---

#### SHOULD HAVE (v1.1)

| ID | Feature | Description |
|----|---------|-------------|
| F-010 | NEMSIS v3.5 Export | XML generation for state reporting |
| F-011 | Station Desktop Processing | Enhanced accuracy with larger models |
| F-012 | Medication Review UI | Edit/correct extractions before export |
| F-013 | Supervisor Dashboard | View all incidents with search/filter |
| F-014 | Data Sync | Queue incidents, sync when WiFi available |

---

#### COULD HAVE (Future)

| ID | Feature |
|----|---------|
| F-015 | Incident Summarization (LLM) |
| F-016 | HL7 v2 Integration |
| F-017 | Real-time Vitals Integration |
| F-018 | Multi-language Support |
| F-019 | John Snow Labs NER Upgrade |

---

#### WON'T HAVE (MVP Out of Scope)

| Feature | Rationale |
|---------|-----------|
| Cloud Sync | Adds HIPAA complexity, no time in MVP |
| Hospital Integration | Requires ePCR API contracts |
| Advanced Analytics | Time-consuming ML work |
| Voice Commands | Complex in noisy ambulance |

---

## 7. Out of Scope

### 7.1 Not in MVP

| Feature | When | Why |
|---------|------|-----|
| NEMSIS v3.5 Export | v1.1 | Requires domain expertise |
| ePCR Integration | v1.1+ | Requires vendor contracts |
| Incident Summarization | v1.1 | LLM quality needs tuning |
| Multi-device Sync | v2.0 | Complex conflict resolution |
| Hospital Handoff | v1.1+ | HL7/FHIR integration |

### 7.2 User Expectations Not Met in MVP

- **"Will it auto-file the report?"** - No, generates data for manual review
- **"Can admin control everything?"** - Local admin only, no cloud management
- **"Does it give call summaries?"** - MVP extracts drugs only, summary in v1.1
- **"Will older tablets work?"** - Requires 8GB+ RAM

---

## 8. Assumptions

### 8.1 Technical Assumptions

| Assumption | Risk if Wrong |
|------------|---------------|
| Whisper-small runs on 8GB tablet | May need smaller model |
| ScispaCy/Med7 F1 >80% on transcribed speech | May need text normalization |
| SQLCipher <10% performance overhead | May need optimization |
| 2-3x realtime latency acceptable | Users may expect faster |
| Med7 achieves 2x audio latency on mid-range tablet (INT8/ONNX) | May need DistilBERT alternative |
| RNNoise/DeepFilterNet effective on 102.5 dBA ambulance noise | May need custom noise profiles |
| Glove-mode touchscreen supported on target tablets | Verify Samsung Tab Active/Panasonic Toughbook |

### 8.2 User Behavior Assumptions

| Assumption | Risk if Wrong |
|------------|---------------|
| Paramedics will record immediately after calls | May need workflow training |
| Users will tolerate 85% accuracy with manual review | May need higher accuracy |
| Crews will sync at station before shift end | May lose data |

### 8.3 Business Assumptions

| Assumption | Risk if Wrong |
|------------|---------------|
| 20-30 agencies achievable in Year 1 | May need more sales effort |
| $50-150/seat pricing acceptable | May need lower price point |
| Fire Chiefs are primary buyers | May need different sales approach |

---

## 9. Success Metrics

### 9.1 Product Metrics

| Metric | MVP Target | V1.0 Target |
|--------|-----------|-------------|
| Transcription WER | < 15% | < 10% |
| Drug Extraction F1 | > 80% | > 88% |
| RxNorm Mapping Accuracy | > 90% | > 95% |
| End-to-End Latency | < 2x audio | < 1.5x audio |
| Offline Functionality | 100% | 100% |
| User Adoption (pilot) | N/A | > 70% |

### 9.2 Business Metrics

| Metric | Month 6 | Month 12 |
|--------|---------|----------|
| Paying Customers | 3-5 | 20-30 |
| ARR | $18-30K | $96-180K |
| NPS | > 30 | > 50 |
| Churn Rate | < 10% | < 5% |

### 9.3 Technical Metrics

| Metric | Target |
|--------|--------|
| Memory Usage | < 6GB on 8GB device |
| Battery Drain | < 20%/hour active use |
| Database Query P95 | < 500ms |
| Error Rate | < 5% |

---

## 10. Risks & Mitigations

### 10.1 Technical Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| **T1: Whisper accuracy <85% on EMS audio** | High | Critical | Test early with real samples; add noise reduction |
| **T2: NER misses critical medications** | Medium | Critical | Add text normalization; manual review queue |
| **T3: Edge device performance issues** | Medium | High | Use lightweight models; lazy-load |
| **T4: Encryption performance overhead** | Medium | Medium | Database-level only for MVP |

### 10.2 Business Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| **B1: Crew adoption resistance** | Medium | High | Early pilots with champions; emphasize time savings |
| **B2: Competition from ESO/ImageTrend** | Medium | High | Differentiate on offline-first, lower cost |
| **B3: Pricing rejection** | Medium | Medium | Validate with fire chiefs; offer tiers |

### 10.3 Regulatory Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| **R1: HIPAA violation** | Low | Critical | Encryption at rest; no cloud; audit logging |
| **R2: NEMSIS compliance gap** | Medium | High | Map fields early; test with state registries |

---

## 11. Dependencies

### 11.1 External Dependencies

| Dependency | Purpose | Risk |
|------------|---------|------|
| OpenAI Whisper (faster-whisper) | Speech-to-text | Low (open source) |
| ScispaCy + Med7 | Medical NER | Medium (niche maintenance) |
| Ollama + Phi-3/Llama | LLM summarization | Low |
| SQLCipher | Database encryption | Low |
| State NEMSIS registries | Compliance | High (external authority) |

### 11.2 Internal Dependencies

| Dependency | Owner | Timeline |
|------------|-------|----------|
| EMS audio samples (100+ min) | Product | Week 1 |
| Fire Chief advisor | Sales | Week 1 |
| HIPAA legal review | Legal | Week 2 |
| Target tablet hardware | Ops | Week 1 |

---

## 12. Release Criteria

### 12.1 MVP Launch Criteria (Must Pass)

**Functionality:**
- [ ] Audio upload works (MP3/WAV, < 30 min)
- [ ] Transcription completes (< 2x audio duration)
- [ ] Medications extracted and highlighted
- [ ] Offline mode works (no internet required)
- [ ] Database encrypted (SQLCipher)
- [ ] PDF export works
- [ ] Paramedic sign-off required before export
- [ ] RxNorm codes mapped to extracted medications

**Quality Gates:**
- [ ] WER < 15% on EMS test set
- [ ] Drug extraction F1 > 80%
- [ ] RxNorm mapping accuracy > 90%
- [ ] Latency < 2x audio on target tablet
- [ ] Memory < 6GB
- [ ] Zero P1 bugs
- [ ] Noise reduction tested on ambulance audio (WER with vs without)

**UI/Accessibility:**
- [ ] All touch targets verified at 60x60px minimum
- [ ] Glove-mode enabled on supported tablets
- [ ] Contrast ratio >= 4.5:1 verified
- [ ] Minimal keyboard input required

**Compliance:**
- [ ] HIPAA Risk Assessment completed
- [ ] BAA template drafted
- [ ] Audit logging working (incl. edit/delete tracking)
- [ ] No PHI in application logs
- [ ] Digital signature capture functional

**Documentation:**
- [ ] User guide (written + video)
- [ ] API documentation
- [ ] Installation guide

### 12.2 Go/No-Go Checkpoints

| Checkpoint | Day | Criteria |
|------------|-----|----------|
| Week 3 | 20 | Whisper + extraction pipeline working |
| Week 4 | 27 | F1 > 80%, latency OK |
| Week 5 | 34 | UI ready, docs complete, HIPAA audit passed |
| Week 6 | 42 | **LAUNCH** if all criteria met |

### 12.3 Rollout Strategy

| Phase | Timeline | Target | Success Criteria |
|-------|----------|--------|------------------|
| Closed Pilot | Weeks 7-9 | 2-3 agencies | >70% adoption, NPS >30 |
| Controlled Rollout | Weeks 10-16 | 5-8 agencies | >70% adoption, NPS >40 |
| General Availability | Month 4+ | Open | 15-25 agencies |

---

## 13. Timeline & Milestones

### 13.1 MVP Schedule (6 Weeks)

```
Week 1: Audio Pipeline + NOISE REDUCTION BASELINE
  - Set up faster-whisper with whisper-small model
  - Collect ambulance cab/siren noise samples (100+ min)
  - Test RNNoise and DeepFilterNet filters on EMS audio
  - Establish baseline WER with vs without noise preprocessing
  - Set sampling rate standard to 16kHz
  - CRITICAL: Do not proceed to UI until baseline validates audio pipeline

Week 2: Drug extraction (ScispaCy/Med7 + text normalization + RxNorm)
Week 3: Database + encryption (SQLite/SQLCipher)
Week 4: UI + API (Streamlit + FastAPI + Glove-mode touch targets)
Week 5: Documentation + HIPAA audit
Week 6: Final testing + pilot launch
```

### 13.2 Key Milestones

| Milestone | Date | Deliverable |
|-----------|------|-------------|
| M1: Transcription Baseline | Week 1 | Audio → transcript with WER measured; noise baseline established (RNNoise/DeepFilterNet tested) |
| M2: NER Pipeline | Week 2 | Drug extraction with F1 > 80%; RxNorm mapping working |
| M3: End-to-End | Week 3 | Full pipeline working |
| M4: UI Complete | Week 4 | Streamlit app functional |
| M5: Compliance Ready | Week 5 | HIPAA audit passed |
| M6: MVP Launch | Week 6 | Live with 2-3 pilot agencies |

### 13.3 Post-MVP Roadmap

| Phase | Timeline | Focus |
|-------|----------|-------|
| Pilot Refinement | Weeks 7-9 | Bug fixes, feedback collection |
| Controlled Rollout | Weeks 10-16 | 5-8 agencies, NPS tracking |
| V1.0 Development | Month 4-6 | NEMSIS export, ePCR integrations |
| General Availability | Month 6+ | Open sales, 15-25 agencies |

---

## Appendix A: Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│  FIELD (Rugged Tablet - 8-16GB RAM)                         │
│                                                             │
│  Audio → whisper-small → Text Normalization → ScispaCy/Med7 │
│                                                             │
│  Extracted Entities → SQLite (SQLCipher encrypted)          │
│                                                             │
│  Sync Queue → Waiting for WiFi                              │
└─────────────────────────────────────────────────────────────┘
                    ↓ (WiFi at station)
┌─────────────────────────────────────────────────────────────┐
│  STATION (Desktop - 32GB RAM + GPU)                         │
│                                                             │
│  whisper-large → Full Med7 NER → Llama-3.2 Summary          │
│                                                             │
│  NEMSIS Export → State Compliance                           │
└─────────────────────────────────────────────────────────────┘
```

---

## Appendix B: Edge-Optimized Models

| Component | Field (Tablet) | Station (Desktop) |
|-----------|----------------|-------------------|
| Speech-to-Text | whisper-small (500MB) | whisper-large-v3 (1.5GB) |
| NER | en_core_sci_sm | Med7 full |
| LLM | TinyLlama (1.1B) | Llama-3.2-8B |
| Performance | 2-3x realtime | 0.5x realtime |

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Jan 2026 | Product Team | Initial PRD |

---

**Next Steps:**
1. Review this PRD
2. Approve or request changes
3. Begin implementation per timeline