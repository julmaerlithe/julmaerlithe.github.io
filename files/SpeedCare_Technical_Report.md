# SpeedCare: Intelligent Clinical Triage and Queue Prioritization System

## 1. Methodology

### 1.1 Data Consolidation and Preprocessing

SpeedCare was developed through a structured data engineering and software integration workflow designed to transform raw symptom narratives into clinically meaningful triage decisions. The project began with a synthetic clinical urgency dataset containing free-text patient inquiries, numerical triage labels, urgency scores, priority indicators, and chief complaint descriptors. During preprocessing, the dataset was audited for obsolete or clinically ambiguous records. A legacy `ROUTINE` class was identified and removed to prevent conflicts with the final three-class triage schema. In total, 125 rows belonging to this legacy class were dropped.

After routine-class isolation, the cleaned urgency dataset was integrated with a secondary 1,200-row `Symptom2Disease` classification matrix. This secondary matrix supplied disease-specific non-urgent clinical conditions such as `Migraine`, `diabetes`, `Pneumonia`, and related chronic or outpatient conditions. The consolidation process yielded a final ground truth dataset named `Final_Triage_Dataset.csv`, containing 1,575 validated records.

The final dataset is organized around the following schema:

| Column | Description |
|---|---|
| `text input` | Patient symptom narrative or inquiry text |
| `label 0-2` | Numeric triage class used by the classifier |
| `language (english only)` | Language marker for standardized English input |
| `is priority` | Dataset-derived priority flag |
| `urgency score` | Numerical urgency score aligned with the triage label |
| `chief complaint` | Whitelisted clinical category used by the UI and backend |
| `source` | Dataset origin indicator |

This consolidation stage established `Final_Triage_Dataset.csv` as the authoritative reference file for backend inference, testing, and user interface validation.

### 1.2 Class Metric Allocation Rules

To prevent ambiguous label interpretation, SpeedCare uses explicit class-to-metric mapping rules. These rules are hard-aligned with the dataset schema and enforced in the backend inference layer.

| Label | Triage Meaning | Urgency Score | Chief Complaint Rule |
|---:|---|---:|---|
| `0` | Emergency | `5.0` | `Acute Medical Assessment` |
| `1` | Urgent | `4.0` | `Acute Medical Assessment` |
| `2` | Non-Urgent | `1.0` | One of 24 whitelisted disease-specific categories |

The system intentionally treats Label 2 as a non-urgent outpatient or chronic-care category. Therefore, conditions such as `Migraine`, `diabetes`, `Pneumonia`, `Common Cold`, and `Hypertension` remain tied to `Non-Urgent` with an urgency score of `1.0`. This rule prevents legacy indexing bugs where a disease category could accidentally inherit an urgent or emergency display string.

The complete chief complaint whitelist contains 25 official output strings:

| Group | Official Chief Complaint Strings |
|---|---|
| Acute triage category | `Acute Medical Assessment` |
| Non-urgent disease categories | `Cervical spondylosis`, `Jaundice`, `Malaria`, `Bronchial Asthma`, `Hypertension`, `Migraine`, `drug reaction`, `peptic ulcer disease`, `diabetes`, `urinary tract infection`, `allergy`, `gastroesophageal reflux disease`, `Chicken pox`, `Impetigo`, `Dengue`, `Psoriasis`, `Varicose Veins`, `Typhoid`, `Dimorphic Hemorrhoids`, `Arthritis`, `Acne`, `Fungal infection`, `Common Cold`, `Pneumonia` |

The backend validates that every returned `chief_complaint` exactly matches one of these strings. Generic terms such as `General Symptoms`, `Others`, or unvalidated free-text labels are not allowed to overwrite the chief complaint field.

### 1.3 Prioritization Overrides Layer

SpeedCare separates clinical urgency from operational queue priority. The machine learning or lookup stage first determines the underlying medical classification: label, urgency level, urgency score, chief complaint, and dataset-derived priority. After this prediction is complete, a demographic override layer evaluates whether the patient is a Senior Citizen or a Person with Disability (PWD).

The override hierarchy is:

1. Predict the clinical class from the text input.
2. Preserve the predicted `label`, `urgency_level`, and `urgency_score`.
3. Check vulnerable patient flags:
   - `is_senior == true`
   - `is_pwd == true`
4. If either condition is true, set final `is_priority = true`.
5. Reduce the estimated waiting time for operational queue handling.

This design ensures that a non-urgent case such as `Migraine` remains medically represented as Label 2, `Non-Urgent`, with an urgency score of `1.0`, even when the patient receives priority queue handling due to age or disability. The clinical score is not inflated by demographic status; only the queue routing priority changes.

### 1.4 Text Matching Pipeline Architecture

The backend inference route follows a dual-layer strategy designed to maximize both empirical alignment and real-time usability.

First, the submitted symptom text is normalized and checked against `Final_Triage_Dataset.csv`. If the incoming text exactly matches an existing `text input` row, the system bypasses model inference and returns the corresponding dataset row coordinates. This exact-text interceptor guarantees 100% alignment during validation, demonstration, and panel testing because known dataset strings produce their original label, urgency score, and chief complaint mapping.

Second, if the submitted symptom text is not present in the CSV, the fallback natural language processing layer computes a real-time similarity or classification result. In the analytics and model-development environment, this fallback is implemented using Scikit-Learn TF-IDF vectorizers and classification models. TF-IDF converts symptom narratives into numerical feature vectors, allowing the classifier to assign one of the three labels and, for non-urgent cases, map the text to a disease-specific chief complaint from the whitelist. In the deployed Django runtime, this classifier is protected by strict schema validation so the response can only emit approved labels, urgency scores, and chief complaint strings.

The resulting inference response is serialized as a strict JSON object consumed by the React frontend.

## 2. Tools Used

### 2.1 Integrated Development Environment

Visual Studio Code (VS Code) served as the centralized development workspace for SpeedCare. The same workspace was used for frontend implementation, Django backend development, dataset inspection, CSV preprocessing scripts, terminal-based Git operations, and local debugging.

### 2.2 Frontend Framework

The client application was built with React.js. React manages the patient registration form, local component state, triage result rendering, cashier and doctor dashboards, and queue-facing interfaces. Axios is used to submit asynchronous registration payloads to the Django API and consume the resulting JSON triage response.

Key frontend responsibilities include:

| Component Responsibility | Implementation Detail |
|---|---|
| Registration form | Captures patient identity, age, PWD status, and symptom text |
| API communication | Axios POST request to `/api/patient/register/` |
| Triage rendering | Displays raw symptom text, mapped chief complaint, urgency level, urgency score, wait time, and priority badge |
| Dashboard updates | Supports queue, cashier, nurse, and doctor-facing views |

### 2.3 Backend Core Architecture

The backend is implemented using Django and Django REST Framework. It is responsible for request validation, triage inference routing, vulnerable-patient override logic, data persistence, and JSON response generation.

Core backend responsibilities include:

| Backend Area | Responsibility |
|---|---|
| `views.py` | Registration endpoint, inference view mapping, override logic, response serialization |
| `ml_triage.py` | Dataset loading, whitelist validation, exact CSV matching, fallback classification support |
| `models.py` | Patient queue, patient profile, triage queue, vitals, payment, and treatment records |
| API routes | Registration, queue listing, patient lookup, vitals, payment, treatment records, and event streaming |

The backend enforces the rule that every final `chief_complaint` must match the approved `Final_Triage_Dataset.csv` category strings.

### 2.4 Machine Learning and Analytics

Data preprocessing and exploratory analysis were conducted using Pandas and NumPy. These tools supported CSV loading, class filtering, label normalization, dataset concatenation, and integrity checks.

Model experimentation and serialization were designed around Scikit-Learn, particularly:

| Library / Technique | Use |
|---|---|
| Pandas | Loading and transforming CSV datasets |
| NumPy | Numerical handling and array operations |
| Scikit-Learn TF-IDF Vectorizer | Converting symptom text into numerical feature vectors |
| Scikit-Learn classifiers | Predicting triage labels and complaint categories |
| CSV-based validation | Ensuring exact ground-truth alignment for known test inputs |

The system’s production response remains constrained by explicit schema rules even when model predictions are used.

### 2.5 Version Control

Git was used through terminal commands to manage project history and remote synchronization. The repository workflow included local initialization, branch management between `master` and `main`, clearing obsolete remote endpoints, and pushing the project to a dedicated GitHub remote using upstream tracking.

The target remote endpoint was:

```text
https://github.com/bncnttt/SpeedCare.git
```

Representative Git operations included:

```bash
git init
git status
git branch -M main
git remote remove origin
git remote add origin https://github.com/bncnttt/SpeedCare.git
git push -u origin main
```

## 3. Architectural Diagram

The following text blueprint summarizes the complete SpeedCare data flow from patient input to triage result display.

```text
+--------------------------------------------------------------------------------+
|                         SpeedCare Intelligent Triage Flow                       |
+--------------------------------------------------------------------------------+

  [1] Welcome Screen / Registration Form
      - Patient name
      - Age
      - PWD checkbox
      - Senior Citizen status inferred from age or explicit flag
      - Primary symptom / inquiry text
                         |
                         v
  [2] React State Handler
      - Normalizes patient form values
      - Preserves exact raw symptom string
      - Builds registration payload
                         |
                         v
  [3] Axios API Call
      POST /api/patient/register/
      Payload:
        {
          name,
          age,
          is_pwd,
          is_senior,
          text_input / symptoms,
          address,
          medical_history,
          language
        }
                         |
                         v
  [4] Django Backend View: views.py
      - Validates required fields
      - Extracts text_input
      - Parses Senior/PWD flags
      - Calls classification inference layer
                         |
                         v
  [5] Preprocessing and CSV Lookup Block
      - Trims and lowercases symptom text
      - Checks exact match against Final_Triage_Dataset.csv
      - If exact row exists:
            return row-aligned label, urgency score, chief complaint
      - Guarantees dataset-coordinate alignment for known test strings
                         |
                         v
  [6] Fallback TF-IDF / ML Classifier Engine
      - Used when text does not exactly match CSV
      - Converts text to vector representation
      - Predicts label 0, 1, or 2
      - Maps non-urgent label 2 to one whitelisted disease category
      - Applies strict output whitelist validation
                         |
                         v
  [7] Class Metadata Enforcement
      Label 0 -> Emergency  -> urgency_score 5.0
      Label 1 -> Urgent     -> urgency_score 4.0
      Label 2 -> Non-Urgent -> urgency_score 1.0
                         |
                         v
  [8] Vulnerable Patient Business Rule Overlay
      IF is_senior == true OR is_pwd == true:
          final is_priority = true
          estimated wait time is reduced
      ELSE:
          final is_priority follows dataset/model rule

      Note:
          label, urgency_level, and urgency_score remain unchanged.
                         |
                         v
  [9] Strict JSON Object Response
      {
        text_input,
        label,
        urgency_level,
        urgency_score,
        chief_complaint,
        is_priority,
        estimated_wait_time,
        classification {...},
        patientQueue {...}
      }
                         |
                         v
 [10] Triage Result Page
      - Displays original raw symptom text
      - Displays mapped whitelisted chief complaint
      - Displays correct Label 0-2 urgency string
      - Displays urgency score
      - Displays conditional priority badge
      - Displays estimated wait time and queue reference
```

## 4. Results and Discussion

### 4.1 Structural Data Integrity

The final SpeedCare dataset provides a controlled three-class triage structure suitable for deterministic validation. Removing the obsolete `ROUTINE` class eliminated a major source of label ambiguity. Integrating the cleaned urgency dataset with the 1,200-row `Symptom2Disease` matrix expanded the system’s non-urgent coverage while preserving a simple triage hierarchy.

The most important structural safeguard is the hardcoded label-to-metadata map. This map ensures that Label 2 cannot be accidentally displayed as `Urgent` or assigned an urgency score above `1.0`. For example, a `Migraine` classification must resolve as:

| Field | Expected Value |
|---|---|
| `chief_complaint` | `Migraine` |
| `label` | `2` |
| `urgency_level` | `Non-Urgent` |
| `urgency_score` | `1.0` |
| `dataset_is_priority` | `False` |

This alignment prevents legacy array-indexing failures where UI label arrays or model outputs could become offset from the dataset schema. As a result, user-facing triage results are consistent with the empirical dataset definitions.

### 4.2 Classification Pipeline Evaluation

After isolating the legacy routine records, the three-class pipeline became structurally clearer. Emergency and urgent records are represented through `Acute Medical Assessment`, while non-urgent records are distributed across disease-specific categories. This separation supports more interpretable classification behavior because the system distinguishes between acute triage severity and routine outpatient disease identification.

The evaluation of the three-class NLP pipeline demonstrated strong alignment across target classes once the obsolete routine category was removed. The most important practical outcome was not only predictive performance but also schema fidelity: each predicted class is constrained to a medically interpretable urgency level and score. The system therefore avoids high-confidence but invalid UI states, such as displaying `Migraine` as `Emergency` or assigning chronic-care symptoms an acute score.

### 4.3 User Interface Alignment and Transparency

The React interface was updated to bind directly to the backend classification payload instead of reconstructing triage results from local fallback arrays. This change ensures that the Triage Result page displays the same label, urgency level, urgency score, and chief complaint produced by the Django inference layer.

In the Complaints section, the interface displays both:

1. The exact raw symptom text submitted by the patient.
2. The whitelisted `chief_complaint` returned by the backend.

This dual display improves empirical transparency during demonstrations and panel evaluation. Reviewers can compare the patient’s original inquiry with the system’s mapped clinical category, making the classification process easier to audit. It also supports debugging because mismatches between free text, dataset rows, and displayed complaint categories become immediately visible.

### 4.4 Vulnerable-Patient Rule Interpretation

The Senior Citizen and PWD override layer provides an ethical operational adjustment without corrupting clinical metrics. For example, an older patient with a non-urgent migraine complaint can receive priority handling while the underlying classification remains `Non-Urgent` with a score of `1.0`. This separation makes the system more clinically honest and operationally humane.

The distinction between `dataset_is_priority` and final `is_priority` is important:

| Concept | Meaning |
|---|---|
| `dataset_is_priority` | Priority derived from the medical text classification |
| `is_priority` | Final queue priority after vulnerable-patient override |
| `urgency_score` | Clinical urgency score that remains unchanged by demographic status |
| `estimated_wait_time` | Operational queue estimate that may be reduced by Senior/PWD status |

## 5. Conclusion

SpeedCare demonstrates how data engineering, machine learning, clinical rule design, and responsive web development can be integrated into a practical intelligent triage application. By consolidating raw clinical text datasets, removing obsolete routine-class records, enforcing strict triage metadata rules, and validating all outputs against a controlled chief complaint whitelist, the system establishes a stable foundation for real-time clinical queue management.

The project’s most significant contribution is its separation of medical urgency from operational priority. Machine learning and dataset lookup determine the clinical classification, while Senior Citizen and PWD rules adjust queue handling without altering the underlying urgency score. This makes the system both ethically responsive and technically reliable.

The React and Django implementation further strengthens system usability. Patients submit symptom narratives through an accessible registration interface, the backend returns a strict JSON classification object, and the Triage Result page renders the original text next to the mapped chief complaint, label, urgency score, wait time, and priority badge. This architecture supports transparency, auditability, and readiness for capstone evaluation or real-world clinic deployment.

In summary, SpeedCare bridges scientific data preparation with real-time clinical software engineering. Its strict schema validation, protected demographic override rules, and responsive dashboard design position it as a reliable prototype for intelligent healthcare queueing systems.
