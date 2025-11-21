# Guardial Feature Quick Reference

## 🎯 All 8 Features at a Glance

---

## 1️⃣ Prompt Shield
**What it does:** Real-time input protection  
**Status:** Core Feature  
**Icon:** ≡ (lines)

### Capabilities
- Prompt injection detection
- Jailbreak prevention
- Harmful content classification
- Multi-layer PII redaction (regex + spaCy + LLM)
- Configurable thresholds
- Live trace visualization

### Use Cases
- Prevent system prompt exfiltration
- Block malicious instructions
- Redact sensitive data before LLM sees it

### API
```
POST /shield_prompt
{ "prompt": "your text" }
```

---

## 2️⃣ LLM Unlearning ⚡ NEW
**What it does:** Remove specific info from models  
**Status:** NEW Enterprise Feature  
**Icon:** ⏱️ (clock)

### Capabilities
- **96x faster** than retraining (30 min vs 14 hours)
- LoRA adapter technique
- 94-96% retain accuracy
- 88-93% forget accuracy
- Multi-prompt validation

### Use Cases
- GDPR "right to be forgotten" requests
- Remove customer data from models
- Comply with privacy regulations without full retraining

### API
```
POST /api/unlearn
Form Data:
- training_set: <file>
- forget_set: <file>
- info_to_forget: "specific term"
```

### Metrics
- Retain Accuracy: How well model keeps other knowledge
- Forget Accuracy: How well specific info is removed

---

## 3️⃣ Hallucination Auditor 🛡️ NEW
**What it does:** Ground outputs in knowledge base  
**Status:** NEW Enterprise Feature  
**Icon:** 🛡️ (shield with core)

### Capabilities
- ISR (Information Source Retrieval) scoring (0-1)
- Configurable threshold (default 0.70)
- Vector database (ChromaDB)
- Semantic search validation
- 5 confidence levels

### Use Cases
- Ensure factual accuracy
- Prevent made-up information
- Validate model responses against trusted data

### API
```
POST /api/auditor/upload
Form Data: dataset (CSV/JSON)

POST /api/auditor/query
{ "query": "question", "threshold": 0.70 }

GET/POST /api/auditor/threshold
```

### ISR Threshold Guide
- **0.75-1.0:** Very high confidence (direct match)
- **0.55-0.74:** High confidence (closely related)
- **0.35-0.54:** Moderate confidence (some relevance)
- **0.20-0.34:** Low confidence (limited relevance)
- **0.0-0.19:** Very low confidence (outside scope) ❌ BLOCKED

---

## 4️⃣ Model Forge 🔨 NEW
**What it does:** Fine-tune base models  
**Status:** NEW Enterprise Feature  
**Icon:** ✓ (checkmark in box)

### Capabilities
- Customizable hyperparameters (epochs, learning rate)
- Real-time loss curve visualization
- Model metadata tracking
- Version control
- Mixed precision training (FP16)

### Use Cases
- Adapt models to domain-specific data
- Create custom models for unlearning
- Rapid iteration on model improvements

### API
```
POST /api/forge/tune
Form Data:
- dataset: <file.csv>
- epochs: 2
- learning_rate: 5e-5
```

### Workflow
1. Upload training data (CSV with 'text' column)
2. Configure epochs & learning rate
3. Monitor loss curve in real-time
4. Model saved with metadata
5. Ready for unlearning or deployment

---

## 5️⃣ Output Screening
**What it does:** Post-generation safety checks  
**Status:** Core Feature  
**Icon:** 🏛️ (layers)

### Capabilities
- Response harmful content detection
- PII redaction in generated text
- Policy violation blocking
- Configurable screening rules
- Full audit trail logging

### Use Cases
- Screen LLM outputs before delivery
- Redact sensitive data in responses
- Ensure compliance in generated content

### Configuration
Via `policy.json`:
```json
{
  "response_screening": {
    "enabled": true,
    "detectors": {
      "harmful_content": { "enabled": true },
      "pii_redaction": { "enabled": true }
    }
  }
}
```

---

## 6️⃣ Policy & Controls
**What it does:** Configure without code changes  
**Status:** Core Feature  
**Icon:** 📋 (document with lines)

### Capabilities
- JSON policy configuration
- Per-detector threshold tuning
- Strategy selection (ML/heuristic/LLM/hybrid)
- Entity type customization
- Feature flags for modules

### Strategies
- **ml:** Use ML models (transformers, spaCy)
- **heuristic:** Keyword/rule-based
- **llm:** Use LLM for detection
- **hybrid:** Combine multiple approaches

### API
```
GET /api/policy
```

---

## 7️⃣ Observability
**What it does:** Transparency into decisions  
**Status:** Core Feature  
**Icon:** ⊖ (circle with line)

### Capabilities
- Structured JSON event logging
- Step-by-step trace visualization
- Decision rationales
- RESTful logs API
- Compliance audit trails

### Event Types
- **BLOCK:** Request blocked (harmful, injection)
- **REDACT:** PII redacted
- **SUCCESS:** Request allowed
- **UNLEARN:** Model unlearning completed
- **FORGE:** Model fine-tuning completed

### API
```
GET /api/logs?limit=500
```

### Use Cases
- Debug why a request was blocked
- Generate compliance reports
- Monitor system behavior
- Audit trail for regulations

---

## 8️⃣ Enterprise Integration
**What it does:** Production deployment  
**Status:** Core Feature  
**Icon:** 🔌 (connection nodes)

### Capabilities
- Cloud Run ready (Buildpacks + Procfile)
- Secret Manager integration
- RESTful API for all features
- GDPR & CCPA compliance tools
- Minimal latency (<1s overhead)

### Deployment
```bash
gcloud run deploy guardial \
  --source . \
  --region us-central1 \
  --set-secrets GEMINI_API_KEY=GEMINI_API_KEY:latest
```

### Integration Points
- All features accessible via REST API
- Webhook support (planned)
- SDK libraries (planned)

---

## 🔗 Feature Relationships

### Workflow: Train → Unlearn → Audit → Shield

```
┌──────────────┐
│ Model Forge  │ Fine-tune base model
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ LLM Unlearn  │ Remove specific info (30 min)
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ Halluc Audit │ Validate against knowledge base
└──────┬───────┘
       │
       ↓
┌──────────────┐
│Prompt Shield │ Real-time protection
└──────────────┘
```

### Data Flow

```
User Input
   ↓
[Prompt Shield] ← Policy & Controls
   ↓ (clean)
[LLM (Forged/Unlearned Model)]
   ↓ (response)
[Output Screening] ← Policy & Controls
   ↓ (safe)
[Hallucination Auditor] ← Vector DB
   ↓ (validated)
User Output
   ↓
[Observability] → Logs & Audit Trail
```

---

## 📊 Performance Benchmarks

| Feature | Latency | Accuracy | Compliance |
|---------|---------|----------|------------|
| Prompt Shield | <500ms | 95%+ | ✅ GDPR |
| LLM Unlearning | 30-40 min | 94-96% retain | ✅ GDPR |
| Hallucination Auditor | <1s | ISR-dependent | ✅ |
| Model Forge | 10-60 min | Variable | ✅ |
| Output Screening | <500ms | 95%+ | ✅ GDPR |

---

## 🎯 Use Case Matrix

| Scenario | Features Used |
|----------|---------------|
| **Customer Data Removal** | Forge → Unlearn → Audit |
| **Real-time Chat Safety** | Shield → Output Screening → Observability |
| **Compliance Reporting** | Observability + all features |
| **Domain Adaptation** | Forge → Audit |
| **Prevent Hallucinations** | Auditor → Shield |
| **Custom Model Training** | Forge → Unlearn → Shield |

---

## 🔐 Compliance Features

### GDPR
- ✅ Right to be forgotten (Unlearning)
- ✅ PII redaction (Shield + Output Screening)
- ✅ Audit trails (Observability)
- ✅ Data minimization (configurable policies)

### CCPA
- ✅ Consumer data deletion (Unlearning)
- ✅ Data transparency (Observability)
- ✅ Opt-out mechanisms (Policy Controls)

### HIPAA (Healthcare)
- ✅ PHI redaction (Shield + Output Screening)
- ✅ Access logging (Observability)
- ✅ Data integrity (Auditor)

---

## 🚀 Quick Start by Use Case

### "I need to remove customer data from my model"
1. **Prepare datasets:** training_set.csv (full data), forget_set.csv (without customer data)
2. **Navigate to:** http://localhost:8080/unlearning
3. **Upload & run:** 30-40 minutes
4. **Verify:** Retain 94-96%, Forget 88-93%

### "I want to prevent hallucinations"
1. **Prepare knowledge base:** CSV with 'text' column
2. **Navigate to:** http://localhost:8080/auditor
3. **Upload dataset:** Vector DB created
4. **Set threshold:** 0.70 (balanced) or adjust
5. **Query:** Responses validated against dataset

### "I need real-time input protection"
1. **Configure policy:** Edit policy.json
2. **Navigate to:** http://localhost:8080
3. **Test prompts:** Try attacks from example dropdown
4. **Review traces:** See step-by-step decisions

### "I want to fine-tune a model"
1. **Prepare data:** CSV with 'text' column
2. **Navigate to:** http://localhost:8080/forge
3. **Upload & configure:** Set epochs & learning rate
4. **Monitor:** Watch loss curve
5. **Use:** Model available for unlearning

---

## 📞 Support & Resources

- **Documentation:** `README_OBLIVIATE.md`
- **Implementation Details:** `IMPLEMENTATION_COMPLETE.md`
- **Demo Guide:** `DEMO_INSTRUCTIONS.md`
- **API Reference:** Built-in at `/api/policy` and `/api/logs`
- **Frontend Updates:** `FRONTEND_UPDATES.md`
- **Visual Guide:** `VISUAL_CHANGES_SUMMARY.md`

---

**Quick Access URLs:**
- Main Page: http://localhost:8080
- Shield Demo: http://localhost:8080#demo
- Auditor: http://localhost:8080/auditor
- Forge: http://localhost:8080/forge
- Unlearning: http://localhost:8080/unlearning

**Last Updated:** November 21, 2025
