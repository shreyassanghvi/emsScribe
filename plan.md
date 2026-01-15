# emsScribe - Implementation Plan

## Project Overview
HIPAA-compliant tool for emergency responders that:
- Transcribes audio recordings from ambulance transport
- Extracts drugs/medications used
- Generates incident summaries
- Maintains audit logs

---

## Chosen Architecture

```
Audio Input → faster-whisper → ScispaCy/Med7 → Ollama/Llama-3 → SQLite
   (local)      (local STT)      (drug NER)     (summary LLM)   (encrypted logs)
```

**All processing happens offline. No internet required.**

---

## Tech Stack

| Component | Tool | Purpose |
|-----------|------|---------|
| Speech-to-Text | `faster-whisper` (large-v3) | 4x faster than Whisper, local |
| Medical NER | `ScispaCy` + `Med7` | Drug, dosage, condition extraction |
| Summarization | `Ollama` + `Llama-3.2-8B` | Incident summary generation |
| Database | `SQLite` + `SQLCipher` | Encrypted audit logging |
| API | `FastAPI` | REST endpoints |
| UI | `Streamlit` (MVP) | Simple web interface |

---

## Hardware Requirements (EDGE-COMPATIBLE)

### Field Device (Rugged Tablet in Ambulance)
| Spec | Requirement | Examples |
|------|-------------|----------|
| RAM | 8-16GB | Samsung Galaxy Tab Active, Panasonic Toughbook |
| CPU | ARM64 or x86 | Snapdragon 8 Gen 2, Intel Core Ultra |
| NPU/GPU | Integrated (optional) | Qualcomm Hexagon, Intel Arc |
| Storage | 128GB+ SSD | For models + local DB |
| Battery | 8+ hours | Hot-swappable preferred |

### Edge-Optimized Model Stack
| Component | Full Model | Edge Model (USE THIS) | Size |
|-----------|------------|----------------------|------|
| Whisper | large-v3 (1.5GB) | **small** or **base** (150-500MB) | 10x smaller |
| LLM | Llama-3.2-8B | **Phi-3-mini (3.8B)** or **TinyLlama (1.1B)** | 4-8x smaller |
| NER | en_core_sci_lg | **en_core_sci_sm** (lighter) | 3x smaller |

### Deployment Tiers
| Tier | Device | Models | Performance |
|------|--------|--------|-------------|
| Field (tablet) | 8GB RAM, ARM | whisper-small + TinyLlama + sci_sm | ~2-3x realtime |
| Field (tablet+) | 16GB RAM, NPU | whisper-base + Phi-3-mini + Med7 | ~1.5x realtime |
| Station (desktop) | 32GB + GPU | whisper-large + Llama-3.2 + full NER | ~0.5x realtime |

### Architecture: Edge + Station Sync
```
┌─────────────────────────────────────┐
│  AMBULANCE (Rugged Tablet)          │
│  - whisper-small (lightweight)      │
│  - Basic NER extraction             │
│  - TinyLlama quick summary          │
│  - Queue for station processing     │
└─────────────────────────────────────┘
              ↓ (WiFi sync at station)
┌─────────────────────────────────────┐
│  STATION (Desktop/Server)           │
│  - whisper-large (high accuracy)    │
│  - Full Med7 NER                    │
│  - Llama-3.2 detailed summary       │
│  - NEMSIS export & compliance       │
└─────────────────────────────────────┘
```

**Strategy:** Quick transcription in field, enhanced processing at station.

---

## Implementation Phases

### Phase 1: Audio Transcription Pipeline
**Goal:** Convert EMS audio to text

- [ ] Set up Python environment with `faster-whisper`
- [ ] Download `large-v3` Whisper model
- [ ] Create audio input handler (file upload + live mic)
- [ ] Build transcript output with timestamps
- [ ] Test with sample EMS audio files

**Deliverable:** CLI that converts audio → transcript text

---

### Phase 2: Medical Entity Extraction
**Goal:** Extract drugs, dosages, conditions from transcript

- [ ] Install ScispaCy + `en_core_sci_lg` model
- [ ] Install Med7 for medication-specific NER
- [ ] Build extraction pipeline for:
  - Drug names (e.g., "epinephrine", "naloxone")
  - Dosages (e.g., "0.3mg", "4mg")
  - Routes (e.g., "IV", "IM", "sublingual")
  - Frequencies (e.g., "every 5 minutes")
  - Conditions (e.g., "cardiac arrest", "overdose")
- [ ] Output structured JSON

**Deliverable:** Module that extracts medical entities from text

---

### Phase 3: Incident Summarization
**Goal:** Generate structured EMS incident summary

- [ ] Install Ollama
- [ ] Download Llama-3.2-8B or Phi-3-medium model
- [ ] Design prompt template for EMS summaries:
  - Patient presentation
  - Interventions performed
  - Medications administered
  - Vitals/observations
  - Transport notes
- [ ] Build summary generation module

**Deliverable:** LLM-based summarizer with EMS-specific output

---

### Phase 4: Logging & HIPAA Compliance
**Goal:** Secure, auditable data storage

- [ ] Set up SQLite with SQLCipher encryption
- [ ] Design schema:
  - `incidents` (id, timestamp, audio_hash, transcript, summary)
  - `entities` (id, incident_id, type, value, confidence)
  - `audit_log` (id, timestamp, action, user, details)
- [ ] Implement immutable audit trail
- [ ] Add user authentication (optional for MVP)
- [ ] Document data retention policy

**Deliverable:** Encrypted database with audit logging

---

### Phase 5: API & User Interface
**Goal:** Usable application for EMS crews

- [ ] Build FastAPI backend with endpoints:
  - `POST /transcribe` - Upload audio
  - `GET /incidents` - List incidents
  - `GET /incidents/{id}` - Get incident details
  - `GET /incidents/{id}/export` - Export to PDF
- [ ] Build Streamlit MVP interface:
  - Audio upload/record
  - Real-time transcription display
  - Entity highlighting
  - Summary review/edit
  - Export functionality

**Deliverable:** Working web application

---

### Phase 6: Testing & Validation
**Goal:** Validate accuracy on real EMS data

- [ ] Collect sample EMS audio (anonymized)
- [ ] Measure transcription accuracy (WER)
- [ ] Measure entity extraction accuracy (F1 score)
- [ ] Document accuracy benchmarks
- [ ] Identify improvement areas

**Deliverable:** Accuracy report with recommendations

---

## Future Upgrade: John Snow Labs

When MVP accuracy is insufficient, upgrade NER to John Snow Labs:

| Current (MVP) | Future (Production) |
|---------------|---------------------|
| ScispaCy + Med7 | Spark NLP Healthcare |
| 80-85% accuracy | 92-94% accuracy |
| Free | Enterprise license |

### Migration Triggers:
- Drug extraction accuracy < 85%
- Missing critical medications in testing
- Production deployment requirement

### JSL Benefits:
- +8.9% accuracy over AWS Comprehend Medical
- 2,400+ pre-trained medical models
- Built-in PHI de-identification
- On-premise deployment (HIPAA compliant)

---

## Project Structure

```
emsScribe/
├── CLAUDE.md              # Project context for Claude
├── plan.md                # This file
├── requirements.txt       # Python dependencies
├── src/
│   ├── __init__.py
│   ├── transcribe.py      # Whisper transcription
│   ├── extract.py         # Medical NER
│   ├── summarize.py       # LLM summarization
│   ├── database.py        # SQLite + encryption
│   └── api.py             # FastAPI endpoints
├── ui/
│   └── app.py             # Streamlit interface
├── tests/
│   └── test_pipeline.py   # Unit tests
└── data/
    └── samples/           # Test audio files
```

---

## Dependencies (REVISED)

```
# Core - Audio Processing (MISSING FROM ORIGINAL)
librosa>=0.10.0
soundfile>=0.12.0
pydub>=0.25.0

# Core - Speech to Text
faster-whisper>=1.0.0

# Core - NLP (VERSION PINNED for compatibility)
spacy>=3.4.0,<3.7.0          # ScispaCy requires 3.4-3.6
scispacy>=0.5.1

# Core - LLM
ollama>=0.1.30               # Earlier versions lack streaming support

# Medical NER models (download separately)
# python -m spacy download en_core_sci_lg
# pip install https://huggingface.co/kormilitzin/en_core_med7_lg/resolve/main/en_core_med7_lg-any-py3-none-any.whl

# Database & Encryption
sqlalchemy>=2.0.0
sqlcipher3>=5.4.0            # Updated from 0.5.0
cryptography>=41.0.0         # Field-level PHI encryption

# API & UI
fastapi>=0.100.0
uvicorn>=0.20.0
streamlit>=1.30.0
aiofiles>=23.0.0             # Async file operations

# Logging & Compliance
structlog>=23.0.0            # HIPAA audit trail logging
python-dotenv>=1.0.0         # Secure key management

# Export & Reporting
reportlab>=4.0.0             # PDF generation
lxml>=4.9.0                  # NEMSIS XML generation

# Utilities
pydantic>=2.0.0
python-multipart>=0.0.6
huggingface-hub>=0.17.0      # Model downloads
```

---

## HIPAA Compliance Checklist

- [ ] All PHI processed locally (no cloud transmission)
- [ ] Database encryption at rest (SQLCipher)
- [ ] Audit logging for all data access
- [ ] User authentication implemented
- [ ] Data retention policy documented
- [ ] Incident response plan defined
- [ ] No PHI in application logs

---

## Success Criteria

| Metric | Target |
|--------|--------|
| Transcription WER | < 15% |
| Drug extraction F1 | > 80% |
| End-to-end latency | < 2x audio duration |
| Offline operation | 100% functional |

---

## CRITICAL FIX: Text Normalization Layer

**Problem:** ScispaCy/Med7 is designed for clinical notes, NOT transcribed speech. Raw Whisper output has:
- Disfluencies ("um", "uh")
- Poor punctuation
- Abbreviations
- Non-standard drug names

**Solution:** Add normalization step between Whisper and NER:

```
Audio → Whisper → TEXT NORMALIZATION → ScispaCy/Med7 → Ollama → SQLite
                        ↑
                   NEW STEP REQUIRED
```

**Normalization tasks:**
- Fix common medical abbreviations (epi → epinephrine)
- Remove disfluencies
- Correct capitalization
- Standardize drug names
- Add sentence boundaries

---

## INTEROPERABILITY REQUIREMENTS (Critical for Sales)

### NEMSIS v3.5 Export (MANDATORY)
- 41+ states require NEMSIS compliance
- Without this, agencies CANNOT legally use emsScribe
- Must export: patient demographics, vitals, medications, disposition

### HL7 v2 Messages (Required for Hospitals)
- 90% of US hospitals use HL7 v2
- Need ORU/ORR observation messages for hospital EMR import

### ePCR System Integrations (High Priority)
- ImageTrend Elite API
- ESO EHR REST API
- ZOLL emsCharts HL7 v2

### New Files Needed:
```
src/integrations/
├── nemsis_exporter.py      # NEMSIS v3.5 XML generation
├── hl7v2_generator.py      # HL7 v2 message creation
├── epcr_systems.py         # ImageTrend/ESO/ZOLL connectors
├── hospital_emr.py         # Hospital handoff
└── validation.py           # Schema validation
```

### New API Endpoints:
```
GET /incidents/{id}/export?format=pdf         # Current
GET /incidents/{id}/export?format=nemsis_xml  # State reporting (NEW)
GET /incidents/{id}/export?format=hl7v2       # Hospital EMR (NEW)
GET /incidents/{id}/export?format=fhir_json   # Modern systems (NEW)
```

---

## BUSINESS MODEL & MONETIZATION

### Pricing Tiers

| Tier | Target | Price | Features |
|------|--------|-------|----------|
| Starter | Small/rural EMS | $50-75/seat/month | Core transcription, local processing |
| Professional | Mid-size fire/EMS | $100-150/seat + $500 platform | NEMSIS export, mobile sync |
| Enterprise | Hospital systems | $50-150K/year | Unlimited seats, on-premise, API |

### Competitive Advantages (Your Edge)
1. **Offline-first** - Works without internet (competitors require cloud)
2. **Local processing** - PHI never leaves device (true HIPAA)
3. **30-50% cheaper** - Lower infrastructure costs than cloud competitors
4. **Time savings** - Reduce charting from 20 min to 5 min per call

### Key Buyers
- Fire Chiefs (40%) - Budget authority, care about crew morale
- EMS Directors (35%) - Clinical authority, care about accuracy
- Ambulance Owners (15%) - Fast decisions, care about cost

### Critical Certifications
1. **NEMSIS v3.5** - Non-negotiable (90% of market needs this)
2. **State certifications** - CA, TX, NY have specific requirements
3. **HIPAA BAA** - Required for hospital customers

### Grant Opportunities
- **HRSA Rural Health** - $50K-500K for rural EMS tech
- **State EMS grants** - $5K-100K per agency
- **Target:** $200-500K in Year 1-2 through grants

### Revenue Projections
| Year | Customers | ARR |
|------|-----------|-----|
| 1 | 20-30 agencies | $50-100K |
| 2 | 100-150 agencies | $300-500K |
| 3 | 300-500 agencies | $1-2M |

---

## FUTURE UPGRADE: John Snow Labs

When MVP accuracy is insufficient, upgrade NER:

| Component | Current | Upgrade |
|-----------|---------|---------|
| NER | ScispaCy (80-85%) | Spark NLP Healthcare (92-94%) |
| Cost | Free | Enterprise license |
| Benefit | +8.9% accuracy over AWS Comprehend |

### Migration Triggers:
- Drug extraction accuracy < 85%
- Enterprise customer requires higher accuracy
- Production deployment at scale
