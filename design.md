# System Design Document
## AI-Powered Rural Healthcare Access Platform

**Version:** 1.0  
**Date:** February 6, 2026  
**Status:** Final

---

## Executive Summary

This document presents the comprehensive system design for an AI-powered healthcare platform targeting rural and semi-urban India. The platform provides AI-based disease triage, treatment guidance, doctor discovery, emergency ambulance booking, and government health scheme integration through a multilingual, voice-first, low-bandwidth interface with offline-first capabilities.

The design prioritizes accessibility, reliability, and compliance with Indian healthcare regulations while maintaining high performance under constrained network conditions.

---

## 1. Design Goals and Principles

### 1.1 Core Design Goals

**Accessibility First**
- Design for users with limited literacy and technical skills
- Support voice-first interactions with minimal text dependency
- Ensure functionality on low-end devices (1GB RAM, Android 6.0+)
- Operate effectively on 2G networks (64 kbps minimum)

**Reliability and Trust**
- Maintain 99.5% uptime for core services, 99.9% for emergency features
- Provide accurate AI predictions with transparent confidence scoring
- Implement graceful degradation when services are unavailable
- Ensure data consistency across offline-online transitions

**Privacy and Security**
- Encrypt all patient data at rest and in transit
- Implement role-based access control with audit logging
- Comply with Digital Personal Data Protection Act (DPDP) and ABDM standards
- Provide user control over data sharing and retention

**Performance Under Constraints**
- Complete AI disease detection within 5 seconds
- Load pages within 3 seconds on 2G networks
- Minimize data usage to under 5 MB per session
- Support 1 million concurrent users with horizontal scalability

**Offline-First Architecture**
- Cache essential medical information locally
- Enable symptom input and analysis offline
- Sync data automatically when connectivity is restored
- Provide SMS fallback for critical features


### 1.2 Design Principles

**Progressive Enhancement**
- Core functionality works on basic devices and networks
- Enhanced features activate on capable devices
- No feature should block basic access

**Human-in-the-Loop**
- AI provides assistance, not diagnosis
- Clear disclaimers about AI limitations
- Always recommend professional consultation for serious conditions
- Healthcare workers can override AI recommendations

**Government-First Approach**
- Prioritize government healthcare facilities in recommendations
- Integrate deeply with public health schemes
- Support government ambulance services (108/102) as primary option
- Align with ABDM infrastructure and standards

**Fail-Safe Emergency Handling**
- Emergency detection has redundant rule-based and ML-based systems
- Emergency features have dedicated high-availability infrastructure
- SMS and voice call fallbacks for ambulance booking
- Automatic notification to emergency contacts

**Cultural and Linguistic Sensitivity**
- Medical terminology adapted to local understanding
- Content reviewed by regional healthcare experts
- UI respects cultural norms and sensitivities
- Support for 10+ regional languages with accurate translations

---

## 2. High-Level System Architecture

### 2.1 Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Mobile     │  │     Web      │  │  SMS/USSD    │          │
│  │   (Android   │  │  (Progressive│  │   Gateway    │          │
│  │   /iOS)      │  │     Web)     │  │              │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    │   API GATEWAY     │
                    │  (Load Balancer)  │
                    └─────────┬─────────┘
                              │
┌─────────────────────────────┴─────────────────────────────────┐
│                    BACKEND SERVICES LAYER                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │   User   │  │ Symptom  │  │  Doctor  │  │Emergency │     │
│  │ Service  │  │ Service  │  │ Service  │  │ Service  │     │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │Treatment │  │  Scheme  │  │Notification│ │Analytics │     │
│  │ Service  │  │ Service  │  │  Service   │ │ Service  │     │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
└───────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────┴─────────────────────────────────┐
│                  AI & DECISION ENGINE LAYER                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │   Disease    │  │  Emergency   │  │ Explainability│        │
│  │   Triage     │  │    Rule      │  │    Engine     │        │
│  │   ML Model   │  │   Engine     │  │               │        │
│  └──────────────┘  └──────────────┘  └──────────────┘        │
│  ┌──────────────┐  ┌──────────────┐                           │
│  │  Treatment   │  │  Confidence  │                           │
│  │ Recommendation│  │   Scoring    │                           │
│  │    Model     │  │   Module     │                           │
│  └──────────────┘  └──────────────┘                           │
└───────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────┴─────────────────────────────────┐
│                  INTEGRATION LAYER                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │    ABDM      │  │  Government  │  │  Ambulance   │        │
│  │  Integration │  │   Scheme     │  │   Service    │        │
│  │              │  │     APIs     │  │  Integration │        │
│  └──────────────┘  └──────────────┘  └──────────────┘        │
│  ┌──────────────┐  ┌──────────────┐                           │
│  │     SMS      │  │   Payment    │                           │
│  │   Gateway    │  │   Gateway    │                           │
│  └──────────────┘  └──────────────┘                           │
└───────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────┴─────────────────────────────────┐
│                      DATA LAYER                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │  PostgreSQL  │  │    Redis     │  │  MongoDB     │        │
│  │  (Primary)   │  │   (Cache)    │  │  (Medical    │        │
│  │              │  │              │  │   Content)   │        │
│  └──────────────┘  └──────────────┘  └──────────────┘        │
│  ┌──────────────┐  ┌──────────────┐                           │
│  │  S3/Object   │  │  Elasticsearch│                           │
│  │   Storage    │  │   (Search)   │                           │
│  └──────────────┘  └──────────────┘                           │
└───────────────────────────────────────────────────────────────┘
```

### 2.2 Key Architectural Characteristics

**Microservices Architecture**
- Independent services with clear boundaries
- Each service owns its data and business logic
- Services communicate via REST APIs and message queues
- Enables independent scaling and deployment

**Event-Driven Communication**
- Asynchronous processing for non-critical operations
- Message queue (RabbitMQ/Kafka) for service communication
- Event sourcing for audit trails and data consistency

**Multi-Region Deployment**
- Services deployed across multiple Indian regions
- Data residency compliance (data stays in India)
- Regional failover for high availability
- CDN for static content delivery

**Hybrid Cloud Strategy**
- Core services on public cloud (AWS/Azure/GCP)
- Sensitive data in government cloud (MeghRaj) where required
- Edge caching for offline-first capabilities

---

## 3. Component-wise Architecture

### 3.1 Client Layer

#### 3.1.1 Mobile Application (Android/iOS)

**Technology Stack**
- Framework: React Native (cross-platform development)
- State Management: Redux with Redux Persist (offline support)
- Local Database: SQLite (offline data storage)
- Voice: React Native Voice (speech-to-text)
- Maps: Google Maps SDK (doctor/hospital location)

**Key Components**

**Offline Data Manager**
- Caches medical content, symptom questionnaires, and user data
- Implements conflict resolution for offline edits
- Queues API requests when offline
- Syncs automatically when connectivity is restored

**Voice Interface Module**
- Supports speech-to-text in 10+ regional languages
- Text-to-speech for reading recommendations
- Noise cancellation for rural environments
- Fallback to text input if voice fails

**Location Services**
- GPS-based location for emergency services
- Cached location for offline doctor search
- Privacy controls for location sharing

**Push Notification Handler**
- Receives treatment reminders and follow-ups
- Emergency alerts and scheme notifications
- Works with Firebase Cloud Messaging (FCM)

**Biometric Authentication**
- Fingerprint/Face ID for quick access
- PIN fallback for devices without biometrics
- Secure storage of authentication tokens


#### 3.1.2 Progressive Web Application (PWA)

**Technology Stack**
- Framework: React.js with Next.js (server-side rendering)
- PWA Features: Service Workers, Web App Manifest
- Offline Storage: IndexedDB
- Responsive Design: Tailwind CSS

**Key Features**
- Installable on home screen (app-like experience)
- Works offline with cached content
- Responsive design for tablets and desktops
- Accessible via any modern browser
- Automatic updates without app store dependency

**Optimization Strategies**
- Code splitting for faster initial load
- Lazy loading of non-critical components
- Image optimization with WebP format
- Minified and compressed assets
- Progressive image loading

#### 3.1.3 SMS/USSD Gateway Interface

**Purpose**
- Provides access for feature phone users
- Fallback channel when data connectivity fails
- Critical for emergency services in remote areas

**Capabilities**
- Symptom reporting via structured SMS commands
- Receive disease triage results via SMS
- Ambulance booking through SMS
- Doctor information delivery
- Scheme enrollment status updates

**SMS Command Structure**
```
SYMPTOM <fever> <cough> <duration>
EMERGENCY <location>
DOCTOR <specialty> <pincode>
SCHEME <check/enroll>
```

**USSD Menu Flow**
```
*123# → Main Menu
1. Check Symptoms
2. Find Doctor
3. Emergency
4. Health Schemes
5. My Profile
```

### 3.2 Backend Services Layer

#### 3.2.1 User Service

**Responsibilities**
- User registration and authentication
- Profile management (patient and family members)
- Medical history storage and retrieval
- Privacy preference management
- Session management

**Key APIs**
- POST /api/v1/users/register
- POST /api/v1/users/login
- GET /api/v1/users/profile
- PUT /api/v1/users/profile
- GET /api/v1/users/medical-history
- POST /api/v1/users/family-member

**Data Model**
- User: id, phone, name, age, gender, language_preference, location
- MedicalHistory: user_id, condition, diagnosis_date, medications, allergies
- FamilyMember: user_id, name, age, gender, relationship
- Consent: user_id, consent_type, granted_at, expires_at

**Security Features**
- JWT-based authentication with refresh tokens
- OTP verification via SMS
- Rate limiting on authentication endpoints
- Password hashing with bcrypt (if password-based auth added)
- Session timeout after 30 days of inactivity

#### 3.2.2 Symptom Service

**Responsibilities**
- Accept symptom input (text, voice, structured)
- Coordinate with AI engine for disease triage
- Store symptom check history
- Provide symptom questionnaires
- Track symptom progression over time

**Key APIs**
- POST /api/v1/symptoms/check
- GET /api/v1/symptoms/questionnaire
- GET /api/v1/symptoms/history
- POST /api/v1/symptoms/follow-up
- GET /api/v1/symptoms/trending (for analytics)

**Processing Flow**
1. Receive symptom input from client
2. Validate and normalize symptom data
3. Enrich with user medical history
4. Send to AI engine for analysis
5. Apply emergency detection rules
6. Format and return results with confidence scores
7. Store symptom check record

**Caching Strategy**
- Cache common symptom questionnaires (Redis, 24-hour TTL)
- Cache user's recent symptom checks (Redis, 1-hour TTL)
- Invalidate cache on new symptom submission

#### 3.2.3 Doctor Service

**Responsibilities**
- Doctor and hospital directory management
- Search and filter doctors by specialty, location
- Availability and appointment management
- Integration with government hospital databases
- Doctor ratings and reviews

**Key APIs**
- GET /api/v1/doctors/search
- GET /api/v1/doctors/{id}
- POST /api/v1/doctors/appointment
- GET /api/v1/doctors/availability
- POST /api/v1/doctors/review

**Search Algorithm**
- Primary: Government hospitals within 50 km
- Secondary: Private clinics with government scheme acceptance
- Tertiary: Other private practitioners
- Ranking factors: distance, availability, ratings, consultation fees

**Data Sources**
- Government hospital database (via ABDM)
- Private doctor registrations
- Medical council registrations
- User-contributed information (verified)

#### 3.2.4 Emergency Service

**Responsibilities**
- Emergency condition detection
- Ambulance booking and dispatch
- Emergency contact notification
- First aid guidance delivery
- Emergency case tracking

**Key APIs**
- POST /api/v1/emergency/detect
- POST /api/v1/emergency/ambulance/book
- POST /api/v1/emergency/contacts/notify
- GET /api/v1/emergency/first-aid
- GET /api/v1/emergency/status/{booking_id}

**High Availability Design**
- Dedicated infrastructure with 99.9% SLA
- Multi-region deployment with automatic failover
- Redundant database with synchronous replication
- Direct integration with ambulance dispatch systems
- SMS and voice call fallbacks

**Emergency Detection Logic**
- Rule-based system for critical symptoms (chest pain, severe bleeding, etc.)
- ML model for pattern-based detection
- Dual validation (both systems must agree for non-obvious cases)
- False negative minimization (err on side of caution)

#### 3.2.5 Treatment Service

**Responsibilities**
- Provide treatment recommendations based on diagnosis
- Recovery timeline estimation
- Medication information (generic names, dosage)
- Precautions and care instructions
- Follow-up scheduling

**Key APIs**
- GET /api/v1/treatment/recommendations
- GET /api/v1/treatment/recovery-timeline
- POST /api/v1/treatment/follow-up/schedule
- GET /api/v1/treatment/medications/{condition}
- GET /api/v1/treatment/precautions

**Content Management**
- Medical content stored in MongoDB (flexible schema)
- Content versioning for updates
- Regional variations in treatment approaches
- Evidence-based recommendations from medical literature
- Regular review by medical advisory board

**Personalization**
- Consider patient age, gender, medical history
- Account for chronic conditions and allergies
- Adjust for regional disease prevalence
- Factor in available healthcare resources

#### 3.2.6 Scheme Service

**Responsibilities**
- Government health scheme discovery
- Eligibility checking
- Enrollment assistance
- Benefit information
- Scheme utilization tracking

**Key APIs**
- GET /api/v1/schemes/eligible
- GET /api/v1/schemes/{scheme_id}
- POST /api/v1/schemes/enroll
- GET /api/v1/schemes/benefits
- GET /api/v1/schemes/hospitals

**Integrated Schemes**
- Ayushman Bharat (PM-JAY)
- State-specific health schemes
- District-level programs
- Special schemes (maternal health, child health, etc.)

**Enrollment Process**
- Check eligibility based on user profile
- Provide document checklist
- Guide through enrollment steps
- Track application status
- Notify on approval/rejection

#### 3.2.7 Notification Service

**Responsibilities**
- Push notifications to mobile apps
- SMS notifications
- Email notifications (where applicable)
- In-app notifications
- Notification preferences management

**Key APIs**
- POST /api/v1/notifications/send
- GET /api/v1/notifications/history
- PUT /api/v1/notifications/preferences
- POST /api/v1/notifications/mark-read

**Notification Types**
- Treatment reminders
- Follow-up check-ins
- Scheme updates
- Emergency alerts
- System announcements

**Delivery Channels**
- Push: Firebase Cloud Messaging (FCM)
- SMS: Twilio/MSG91 integration
- Email: SendGrid/AWS SES
- In-app: WebSocket connections

**Scheduling**
- Immediate: Emergency alerts
- Scheduled: Treatment reminders, follow-ups
- Batched: Non-urgent notifications
- Time-zone aware delivery

#### 3.2.8 Analytics Service

**Responsibilities**
- User behavior analytics
- Health trend analysis
- System performance monitoring
- Business intelligence reporting
- Predictive analytics

**Key Metrics Tracked**
- User engagement (DAU, MAU, retention)
- Symptom check patterns
- Emergency response times
- Doctor booking conversion rates
- Scheme enrollment rates
- AI model accuracy metrics

**Data Pipeline**
- Real-time: Apache Kafka for event streaming
- Batch: Apache Spark for large-scale processing
- Storage: Data warehouse (Amazon Redshift/Google BigQuery)
- Visualization: Tableau/Metabase dashboards

**Privacy Considerations**
- All analytics data anonymized
- Aggregated reporting only
- No individual patient tracking without consent
- Compliance with data protection regulations


### 3.3 AI & Decision Engine Layer

#### 3.3.1 Disease Triage ML Model

**Model Architecture**
- Primary: Gradient Boosting (XGBoost/LightGBM) for structured symptom data
- Secondary: Deep Neural Network for complex pattern recognition
- Ensemble: Weighted combination of multiple models for robustness

**Input Features**
- Symptom presence/absence (binary features)
- Symptom severity (1-10 scale)
- Symptom duration (hours/days)
- Patient demographics (age, gender)
- Medical history (chronic conditions, past diagnoses)
- Geographic location (regional disease prevalence)
- Seasonal factors (time of year)

**Output**
- Top 3-5 probable conditions with confidence scores
- Risk category (Emergency/Urgent/Moderate/Minor)
- Recommended next steps
- Confidence level (High/Medium/Low)

**Training Data**
- Indian population health records (anonymized)
- Government hospital diagnostic data
- Validated symptom-disease mappings
- Regional disease prevalence statistics
- Continuous learning from platform usage (with consent)

**Model Performance Targets**
- Overall accuracy: >85%
- Emergency detection recall: >98% (minimize false negatives)
- Emergency detection precision: >80%
- Inference time: <2 seconds

**Model Updates**
- Quarterly retraining with new data
- A/B testing for model improvements
- Gradual rollout of new versions
- Rollback capability if performance degrades

#### 3.3.2 Emergency Rule Engine

**Purpose**
- Provides deterministic emergency detection
- Acts as safety net for ML model
- Ensures zero false negatives for critical conditions

**Rule Categories**

**Critical Symptoms (Immediate Emergency)**
- Chest pain with radiation to arm/jaw
- Difficulty breathing/severe shortness of breath
- Severe bleeding that won't stop
- Loss of consciousness
- Severe head injury
- Suspected stroke (FAST symptoms)
- Severe allergic reaction
- Poisoning/overdose

**Urgent Symptoms (Seek Care Within Hours)**
- High fever (>103°F) with confusion
- Severe abdominal pain
- Persistent vomiting/diarrhea with dehydration
- Severe burns
- Suspected fractures

**Rule Evaluation**
- Rules evaluated before ML model
- Any critical symptom triggers immediate emergency flow
- ML model provides additional context
- Both systems must agree for borderline cases

**Rule Management**
- Rules defined by medical advisory board
- Version controlled and auditable
- Regular review and updates
- Regional customization where needed

#### 3.3.3 Treatment Recommendation Model

**Approach**
- Knowledge-based system (not pure ML)
- Decision trees based on medical guidelines
- Personalized based on patient profile
- Evidence-based recommendations

**Recommendation Types**
- Home care instructions
- Over-the-counter medications
- Lifestyle modifications
- When to seek professional care
- Red flags to watch for

**Personalization Factors**
- Patient age and weight (for dosage)
- Existing medical conditions
- Known allergies
- Pregnancy/breastfeeding status
- Current medications (interaction checking)

**Content Sources**
- WHO treatment guidelines
- Indian medical association protocols
- Government health ministry guidelines
- Peer-reviewed medical literature
- Regional medical practices

#### 3.3.4 Explainability Engine

**Purpose**
- Provide transparent reasoning for AI decisions
- Build user trust in recommendations
- Enable medical professional review
- Support regulatory compliance

**Explanation Types**

**Feature Importance**
- Which symptoms most influenced the diagnosis
- Relative weight of each factor
- Visualization of decision path

**Similar Cases**
- Show anonymized similar cases
- Outcomes of similar symptom patterns
- Statistical prevalence information

**Confidence Breakdown**
- Why confidence is high/medium/low
- What additional information would improve confidence
- Uncertainty quantification

**Plain Language Explanations**
- "Based on your fever and cough, this could be flu"
- "Your symptoms match 85% of flu cases in our database"
- "We recommend seeing a doctor because of your age and existing conditions"

**Technical Implementation**
- SHAP (SHapley Additive exPlanations) for model interpretation
- LIME (Local Interpretable Model-agnostic Explanations) for local explanations
- Attention mechanisms in neural networks
- Rule tracing for rule-based components

#### 3.3.5 Confidence Scoring Module

**Scoring Factors**
- Model prediction probability
- Agreement between multiple models
- Completeness of symptom information
- Similarity to training data
- Historical accuracy for similar cases

**Confidence Levels**
- High (>80%): Strong recommendation, clear next steps
- Medium (60-80%): Multiple possibilities, suggest professional consultation
- Low (<60%): Insufficient information, recommend in-person evaluation

**User Communication**
- Visual indicators (color-coded)
- Plain language explanations
- Suggestions to improve confidence (answer more questions)
- Always recommend professional care for low confidence

**Calibration**
- Regular calibration against actual outcomes
- Adjustment of confidence thresholds
- Monitoring of confidence vs. accuracy correlation

### 3.4 Emergency Handling Module

#### 3.4.1 Emergency Detection Pipeline

**Multi-Stage Detection**

**Stage 1: Keyword Detection**
- Scan input for emergency keywords
- Immediate flagging of critical terms
- Language-agnostic (works across all supported languages)

**Stage 2: Rule-Based Evaluation**
- Apply emergency rule engine
- Check symptom combinations
- Evaluate severity indicators

**Stage 3: ML-Based Assessment**
- Run emergency classification model
- Consider patient context
- Generate risk score

**Stage 4: Consensus Decision**
- Combine results from all stages
- Apply conservative threshold (favor false positives)
- Trigger emergency flow if any stage indicates emergency

**Performance Monitoring**
- Track false positive/negative rates
- Continuous improvement of detection logic
- Regular audits by medical professionals

#### 3.4.2 Ambulance Booking System

**Integration Architecture**

**Government Ambulance Services**
- Primary: 108 (Emergency) and 102 (Non-emergency)
- Direct API integration where available
- Phone call fallback via automated dialer
- SMS-based booking for areas without API

**Private Ambulance Networks**
- Secondary option if government unavailable
- Multiple provider integrations
- Real-time availability checking
- Transparent pricing

**Booking Flow**
1. Detect emergency condition
2. Confirm patient location (GPS or manual)
3. Check government ambulance availability
4. If unavailable, check private providers
5. Send booking request with patient details
6. Receive confirmation and ETA
7. Notify patient and emergency contacts
8. Track ambulance location (if supported)
9. Provide first aid instructions while waiting

**Fallback Mechanisms**
- If API fails: Automated phone call
- If phone fails: SMS with emergency number
- If all fail: Display emergency numbers for manual calling

**Data Shared with Ambulance**
- Patient location (GPS coordinates + address)
- Emergency condition summary
- Patient age and gender
- Known allergies and medical conditions
- Emergency contact information

#### 3.4.3 First Aid Guidance

**Content Delivery**
- Step-by-step instructions
- Voice-guided procedures
- Visual diagrams (when bandwidth allows)
- Video demonstrations (cached offline)

**Condition-Specific Guidance**
- CPR instructions
- Choking response
- Bleeding control
- Burn treatment
- Fracture immobilization
- Stroke response (FAST)

**Safety Features**
- Clear warnings about what NOT to do
- Emphasis on waiting for professional help
- Continuous reassurance and support
- Timer for time-sensitive procedures (CPR cycles)

### 3.5 Government Integration Layer

#### 3.5.1 ABDM (Ayushman Bharat Digital Mission) Integration

**Health ID Integration**
- Create/link ABDM Health ID
- Fetch patient health records
- Share health records with consent
- Maintain interoperability standards

**Health Data Exchange**
- HL7 FHIR standard compliance
- Secure data exchange protocols
- Consent management framework
- Audit trail for all data access

**Integration Points**
- Health ID creation API
- Health record fetch API
- Consent management API
- Healthcare provider registry

#### 3.5.2 Government Scheme APIs

**Ayushman Bharat (PM-JAY)**
- Eligibility verification API
- Beneficiary details fetch
- Empaneled hospital list
- Claim status tracking

**State Health Schemes**
- State-specific API integrations
- Unified interface for multiple schemes
- Fallback to manual verification if API unavailable

**Data Synchronization**
- Daily sync of scheme databases
- Real-time eligibility checks where possible
- Cached data for offline access

#### 3.5.3 Government Hospital Database

**Integration Approach**
- API integration where available
- Web scraping for public data (as fallback)
- Manual data entry and verification
- Crowdsourced updates with verification

**Data Maintained**
- Hospital name and location
- Available specialties
- OPD timings
- Contact information
- Bed availability (where available)
- Scheme acceptance status


---

## 4. Data Flow Diagrams

### 4.1 Normal Symptom Check Flow

```
User → Mobile App → Symptom Service → AI Engine → Treatment Service → User

Detailed Steps:
1. User opens app and selects "Check Symptoms"
2. App presents symptom input interface (voice/text/questionnaire)
3. User describes symptoms (e.g., "fever and headache for 2 days")
4. App sends request to Symptom Service with:
   - Symptom data
   - User ID and medical history
   - Location and timestamp
5. Symptom Service validates and enriches data:
   - Normalizes symptom descriptions
   - Fetches user medical history from User Service
   - Adds contextual information (season, regional prevalence)
6. Symptom Service sends enriched data to AI Engine
7. AI Engine processes:
   - Emergency Rule Engine checks for critical symptoms
   - ML Model generates disease predictions
   - Confidence Scoring Module calculates confidence levels
   - Explainability Engine generates reasoning
8. AI Engine returns results:
   - Top 3-5 probable conditions with confidence scores
   - Risk category (Emergency/Urgent/Moderate/Minor)
   - Explanation of reasoning
9. If NOT emergency:
   - Symptom Service requests treatment recommendations from Treatment Service
   - Treatment Service returns:
     * Home care instructions
     * Medication suggestions
     * Recovery timeline
     * When to seek professional care
   - Symptom Service requests doctor recommendations from Doctor Service
   - Doctor Service returns nearby doctors/hospitals
10. Symptom Service stores symptom check record
11. Symptom Service returns complete response to app
12. App displays results in user's language:
    - Disease possibilities with confidence
    - Treatment recommendations
    - Doctor recommendations
    - Follow-up scheduling option
13. Notification Service schedules follow-up reminders
14. Analytics Service logs interaction for improvement
```

### 4.2 Emergency Flow

```
User → Mobile App → Emergency Service → Ambulance + Contacts → First Aid

Detailed Steps:
1. User reports emergency symptoms OR AI detects emergency
2. App immediately switches to emergency mode:
   - Red alert UI
   - Large "Call Ambulance" button
   - Emergency disclaimer
3. If AI-detected:
   - Emergency Rule Engine flags critical symptoms
   - ML Model confirms high-risk condition
   - Both systems agree → Emergency flow triggered
4. App prompts: "This appears to be an emergency. Book ambulance?"
5. User confirms (or auto-confirm after 5 seconds)
6. App sends emergency request to Emergency Service:
   - Patient location (GPS + address)
   - Emergency condition summary
   - Patient vital information
   - Emergency contact list
7. Emergency Service processes:
   - Validates and prioritizes request
   - Logs emergency case with unique ID
8. Emergency Service attempts ambulance booking:
   - Primary: Government ambulance (108/102) API
   - If unavailable: Private ambulance network
   - If API fails: Automated phone call
   - If all fail: Display emergency numbers
9. Ambulance booking confirmed:
   - Emergency Service receives confirmation and ETA
   - Booking ID and ambulance details stored
10. Emergency Service triggers parallel actions:
    - Notify emergency contacts via SMS and push notification
    - Send patient location and condition to contacts
    - Fetch first aid instructions for condition
11. Emergency Service returns to app:
    - Ambulance confirmation and ETA
    - First aid instructions
    - Nearest hospital information
12. App displays:
    - "Ambulance booked, arriving in X minutes"
    - Step-by-step first aid guidance (voice + text)
    - Emergency contact notification status
    - Live ambulance tracking (if available)
13. App provides continuous support:
    - Voice-guided first aid
    - Reassurance messages
    - Option to call emergency contacts
    - Timer for time-sensitive procedures
14. Emergency Service monitors:
    - Ambulance arrival status
    - Updates to emergency contacts
    - Logs all actions for audit
15. Post-emergency:
    - Emergency Service logs outcome
    - Analytics Service tracks response time
    - Follow-up notification scheduled
```

### 4.3 Offline-to-Online Sync Flow

```
Offline Actions → Local Storage → Connectivity Restored → Sync Service → Backend

Detailed Steps:

OFFLINE PHASE:
1. User loses internet connectivity
2. App detects offline state:
   - Displays offline indicator
   - Switches to offline mode
   - Loads cached data
3. User performs actions offline:
   - Checks symptoms using cached questionnaires
   - Views cached medical content
   - Updates profile information
   - Schedules follow-ups
4. App stores actions locally:
   - SQLite database for structured data
   - IndexedDB for web app
   - Action queue with timestamps
   - Conflict resolution metadata
5. Offline AI processing:
   - Basic symptom analysis using cached ML model
   - Rule-based emergency detection
   - Limited disease triage (common conditions only)
   - Results marked as "offline analysis"
6. App displays offline results:
   - Clear "Offline Mode" indicator
   - Limited confidence in predictions
   - Recommendation to sync when online
   - Emergency features still available (SMS fallback)

ONLINE PHASE:
7. Connectivity restored:
   - App detects online state
   - Displays "Syncing..." indicator
   - Initiates sync process
8. Sync Service processes queue:
   - Retrieves all pending actions from local storage
   - Orders by timestamp
   - Groups related actions
9. Conflict detection:
   - Checks if server data changed during offline period
   - Identifies conflicts (e.g., profile updated on another device)
   - Applies conflict resolution rules:
     * Last-write-wins for profile updates
     * Merge for non-conflicting changes
     * User prompt for critical conflicts
10. Data upload:
    - Sends offline symptom checks to Symptom Service
    - Uploads profile changes to User Service
    - Submits scheduled follow-ups to Treatment Service
    - Includes offline timestamp and device ID
11. Server processing:
    - Backend services validate offline data
    - Re-run AI analysis with full models
    - Update confidence scores
    - Generate improved recommendations
12. Data download:
    - Fetch latest medical content updates
    - Download new symptom questionnaires
    - Sync scheme information
    - Update doctor database
    - Refresh ML model if new version available
13. Cache update:
    - Replace offline results with online analysis
    - Update local database with server data
    - Refresh cached content
    - Store sync timestamp
14. Notification to user:
    - "Sync complete"
    - Highlight updated information
    - Show improved recommendations if available
    - Prompt to review changes
15. Analytics logging:
    - Track offline usage patterns
    - Monitor sync success rate
    - Identify common offline scenarios
    - Improve offline capabilities

CONFLICT RESOLUTION EXAMPLES:
- Profile updated offline and online:
  * Merge non-conflicting fields
  * Last-write-wins for same field
  * Notify user of changes
- Symptom check done offline, then online:
  * Keep both records
  * Mark offline version
  * Show improved online analysis
- Emergency detected offline:
  * Immediate SMS fallback used
  * Sync emergency record when online
  * Update emergency contacts with full details
```

---

## 5. AI/ML Architecture

### 5.1 Model Types and Responsibilities

#### 5.1.1 Disease Triage Model

**Model Type:** Ensemble (XGBoost + Neural Network)

**Responsibilities**
- Predict probable diseases from symptoms
- Generate confidence scores
- Categorize risk level
- Provide differential diagnosis

**Architecture**

**XGBoost Component**
- Input: 200+ symptom features (binary + severity)
- Output: Probability distribution over 500+ diseases
- Advantages: Fast inference, interpretable, handles missing data
- Training: Supervised learning on labeled symptom-disease pairs

**Neural Network Component**
- Architecture: Multi-layer perceptron (3 hidden layers)
- Input: Symptom embeddings + patient context
- Output: Disease probabilities
- Advantages: Captures complex patterns, better for rare diseases
- Training: Supervised learning with class balancing

**Ensemble Strategy**
- Weighted average of predictions (70% XGBoost, 30% NN)
- Weights tuned on validation set
- Fallback to XGBoost if NN fails

**Feature Engineering**
- Symptom presence (binary)
- Symptom severity (1-10 scale, normalized)
- Symptom duration (log-transformed hours)
- Symptom combinations (interaction features)
- Patient age group (categorical)
- Gender (binary)
- Chronic conditions (multi-hot encoding)
- Geographic region (categorical)
- Season (categorical)
- Time of day (cyclical encoding)

**Output Format**
```json
{
  "predictions": [
    {
      "disease": "Influenza",
      "confidence": 0.78,
      "icd_code": "J11.1",
      "prevalence": "common"
    },
    {
      "disease": "Common Cold",
      "confidence": 0.65,
      "icd_code": "J00",
      "prevalence": "very_common"
    }
  ],
  "risk_category": "moderate",
  "emergency_score": 0.12,
  "model_version": "2.3.1"
}
```

#### 5.1.2 Emergency Classification Model

**Model Type:** Binary Classifier (Random Forest + Rule Engine)

**Responsibilities**
- Detect emergency conditions
- Minimize false negatives
- Fast inference (<1 second)
- Explainable decisions

**Architecture**

**Random Forest Component**
- 100 decision trees
- Max depth: 10
- Input: Symptom features + vital signs
- Output: Emergency probability (0-1)
- Threshold: 0.3 (conservative, favors sensitivity)

**Rule Engine Component**
- 50+ expert-defined rules
- Covers all critical conditions
- Deterministic and auditable
- Takes precedence over ML model

**Decision Logic**
```
IF rule_engine_flags_emergency:
    RETURN emergency = True
ELIF ml_model_score > 0.3:
    RETURN emergency = True
ELSE:
    RETURN emergency = False
```

**Performance Metrics**
- Recall (Sensitivity): >98% (critical)
- Precision: >80% (acceptable false positives)
- F1 Score: >88%
- Inference time: <500ms

#### 5.1.3 Treatment Recommendation System

**Model Type:** Knowledge-Based System (Decision Trees + Rules)

**Responsibilities**
- Suggest appropriate treatments
- Personalize recommendations
- Check contraindications
- Provide evidence-based guidance

**Architecture**

**Decision Tree Structure**
```
Disease → Severity → Patient Profile → Treatment Options

Example:
Influenza → Mild → Adult → Rest + Fluids + Paracetamol
Influenza → Severe → Elderly → Doctor Consultation + Antiviral
```

**Personalization Factors**
- Age and weight (dosage calculation)
- Pregnancy/breastfeeding status
- Chronic conditions (diabetes, hypertension, etc.)
- Known allergies
- Current medications (interaction checking)
- Access to healthcare (urban vs. rural)

**Contraindication Checking**
- Drug-drug interactions
- Drug-disease interactions
- Age-based restrictions
- Pregnancy category checking

**Evidence Levels**
- Level A: Strong evidence (RCTs, meta-analyses)
- Level B: Moderate evidence (cohort studies)
- Level C: Expert opinion
- All recommendations tagged with evidence level


#### 5.1.4 Symptom Extraction Model (NLP)

**Model Type:** Named Entity Recognition (BERT-based)

**Responsibilities**
- Extract symptoms from free-text input
- Handle multilingual input
- Normalize symptom descriptions
- Map to standard medical terminology

**Architecture**
- Base model: mBERT (multilingual BERT)
- Fine-tuned on medical symptom corpus
- Entity types: Symptom, Duration, Severity, Body Part
- Output: Structured symptom representation

**Example**
```
Input: "मुझे 3 दिन से बुखार और सिर दर्द है" (Hindi)
Output: {
  "symptoms": ["fever", "headache"],
  "duration": "3 days",
  "severity": "moderate",
  "language": "hi"
}
```

**Multilingual Support**
- Trained on parallel corpus (10+ languages)
- Language detection built-in
- Translation to English for processing
- Results translated back to user language

### 5.2 Emergency Rule Engine

**Rule Categories**

**Cardiovascular Emergencies**
- Chest pain + radiation to arm/jaw → Possible heart attack
- Severe chest pain + shortness of breath → Cardiac emergency
- Irregular heartbeat + dizziness → Arrhythmia

**Respiratory Emergencies**
- Severe difficulty breathing → Respiratory distress
- Choking + inability to speak → Airway obstruction
- Blue lips/fingernails → Hypoxia

**Neurological Emergencies**
- Sudden severe headache → Possible stroke/aneurysm
- Face drooping + arm weakness + speech difficulty → Stroke (FAST)
- Loss of consciousness → Medical emergency
- Seizure lasting >5 minutes → Status epilepticus

**Trauma Emergencies**
- Severe bleeding that won't stop → Hemorrhage
- Suspected spinal injury → Immobilization needed
- Severe burns (>10% body surface) → Burn emergency
- Head injury + confusion/vomiting → Possible concussion/TBI

**Other Critical Conditions**
- Severe allergic reaction + difficulty breathing → Anaphylaxis
- Suspected poisoning → Toxicology emergency
- Severe abdominal pain + rigid abdomen → Surgical emergency
- High fever + stiff neck + confusion → Possible meningitis

**Rule Evaluation Engine**
- Rules stored in JSON format
- Version controlled
- Evaluated in priority order
- Multiple rules can trigger simultaneously
- Explanation generated for triggered rules

**Rule Update Process**
1. Medical advisory board proposes rule changes
2. Rules tested on historical data
3. Staged rollout (10% → 50% → 100%)
4. Monitoring for false positives/negatives
5. Rollback if issues detected

### 5.3 Explainability and Confidence Scoring

#### 5.3.1 Explainability Techniques

**SHAP (SHapley Additive exPlanations)**
- Calculates contribution of each feature
- Generates feature importance plots
- Provides local explanations for individual predictions

**Example Output**
```
Your diagnosis of Influenza is based on:
- Fever (high importance: +0.35)
- Body ache (medium importance: +0.22)
- Cough (medium importance: +0.18)
- Fatigue (low importance: +0.08)
- Season (winter increases likelihood: +0.05)
```

**LIME (Local Interpretable Model-agnostic Explanations)**
- Creates simple interpretable model locally
- Shows which symptoms most influenced decision
- Provides "what-if" scenarios

**Example**
```
If you didn't have fever, the diagnosis would change to:
- Common Cold (confidence: 0.72)
- Allergic Rhinitis (confidence: 0.58)
```

**Attention Visualization (for Neural Networks)**
- Shows which symptoms the model focused on
- Highlights important symptom combinations
- Visual heatmap of attention weights

**Plain Language Explanations**
- Template-based generation
- Translated to user's language
- Avoids medical jargon
- Includes analogies and examples

**Example**
```
"Based on your symptoms, you likely have the flu. This is common 
during winter months. Your fever and body ache are the strongest 
indicators. About 8 out of 10 people with these symptoms have flu."
```

#### 5.3.2 Confidence Scoring Methodology

**Factors Contributing to Confidence**

**Model Agreement (40% weight)**
- Agreement between XGBoost and Neural Network
- Higher agreement → Higher confidence
- Formula: 1 - |prob_xgb - prob_nn|

**Prediction Probability (30% weight)**
- Raw probability from ensemble model
- Higher probability → Higher confidence
- Normalized to 0-1 scale

**Information Completeness (15% weight)**
- Percentage of relevant questions answered
- More information → Higher confidence
- Penalty for missing critical symptoms

**Training Data Similarity (10% weight)**
- Distance to nearest training examples
- Closer to training data → Higher confidence
- Uses k-NN in feature space

**Historical Accuracy (5% weight)**
- Model's past performance on similar cases
- Tracked per disease category
- Updated continuously

**Confidence Calculation**
```
confidence = 0.4 * model_agreement +
             0.3 * prediction_probability +
             0.15 * information_completeness +
             0.1 * training_similarity +
             0.05 * historical_accuracy
```

**Confidence Levels**
- High (0.8-1.0): Strong recommendation, clear next steps
- Medium (0.6-0.8): Multiple possibilities, suggest consultation
- Low (0.0-0.6): Insufficient information, recommend in-person evaluation

**User Communication**
```
High Confidence:
"We're quite confident this is Influenza. Follow the treatment 
recommendations below."

Medium Confidence:
"This could be Influenza or a similar viral infection. We recommend 
consulting a doctor to confirm."

Low Confidence:
"We don't have enough information to make a reliable assessment. 
Please consult a doctor for proper diagnosis."
```

### 5.4 Human-in-the-Loop Considerations

#### 5.4.1 Medical Professional Review

**Review Triggers**
- Low confidence predictions (<0.6)
- Conflicting model outputs
- Rare diseases (prevalence <1%)
- User-reported incorrect predictions
- Random sampling (5% of all predictions)

**Review Process**
1. Prediction flagged for review
2. Assigned to medical professional
3. Professional reviews symptoms and prediction
4. Provides feedback: Correct/Incorrect/Uncertain
5. Suggests correct diagnosis if incorrect
6. Feedback used to retrain models

**Review Dashboard**
- Queue of cases needing review
- Priority based on urgency and confidence
- Historical context for each case
- Ability to contact patient if needed
- Feedback submission interface

#### 5.4.2 Continuous Learning Pipeline

**Feedback Collection**
- User feedback on prediction accuracy
- Doctor diagnosis after consultation
- Treatment outcome tracking
- Follow-up symptom progression

**Model Retraining**
- Quarterly retraining schedule
- Incorporates all validated feedback
- A/B testing of new models
- Gradual rollout (10% → 50% → 100%)
- Performance monitoring and rollback capability

**Quality Assurance**
- Medical advisory board reviews model changes
- Validation on held-out test set
- Fairness and bias testing
- Regulatory compliance verification

#### 5.4.3 Override Mechanisms

**Healthcare Worker Override**
- ASHA/ANM workers can override AI recommendations
- Override requires justification
- Logged for audit and learning
- Escalation to doctor if needed

**User Override**
- Users can reject AI recommendations
- Prompted to provide reason
- Alternative options presented
- Feedback used to improve models

**Emergency Override**
- Users can always call emergency services directly
- "Call 108" button always visible
- No AI gatekeeping for emergencies
- Direct access to emergency contacts

---

## 6. Backend Design

### 6.1 API Architecture

#### 6.1.1 API Design Principles

**RESTful Design**
- Resource-based URLs
- Standard HTTP methods (GET, POST, PUT, DELETE)
- Stateless communication
- HATEOAS for discoverability

**Versioning**
- URL-based versioning: /api/v1/, /api/v2/
- Backward compatibility maintained for 2 versions
- Deprecation notices 6 months in advance

**Response Format**
```json
{
  "status": "success",
  "data": { ... },
  "meta": {
    "timestamp": "2026-02-06T10:30:00Z",
    "version": "1.0",
    "request_id": "uuid"
  }
}
```

**Error Format**
```json
{
  "status": "error",
  "error": {
    "code": "INVALID_SYMPTOM",
    "message": "Symptom 'xyz' not recognized",
    "details": { ... }
  },
  "meta": {
    "timestamp": "2026-02-06T10:30:00Z",
    "request_id": "uuid"
  }
}
```

#### 6.1.2 API Gateway

**Responsibilities**
- Request routing to appropriate services
- Load balancing across service instances
- Rate limiting and throttling
- Authentication and authorization
- Request/response transformation
- Caching of common responses
- API analytics and monitoring

**Technology**
- Kong or AWS API Gateway
- Redis for rate limiting
- JWT validation
- Request logging to ELK stack

**Rate Limiting**
- Anonymous users: 10 requests/minute
- Authenticated users: 100 requests/minute
- Emergency endpoints: No rate limiting
- Burst allowance: 2x normal rate for 10 seconds

**Caching Strategy**
- GET requests cached for 5 minutes (default)
- User-specific data: No caching
- Static content: 24-hour cache
- Cache invalidation on data updates

#### 6.1.3 Key API Endpoints

**User Management**
```
POST   /api/v1/users/register
POST   /api/v1/users/login
POST   /api/v1/users/logout
GET    /api/v1/users/profile
PUT    /api/v1/users/profile
POST   /api/v1/users/family-member
GET    /api/v1/users/medical-history
```

**Symptom Checking**
```
POST   /api/v1/symptoms/check
GET    /api/v1/symptoms/questionnaire
GET    /api/v1/symptoms/history
POST   /api/v1/symptoms/follow-up
```

**Doctor Discovery**
```
GET    /api/v1/doctors/search?specialty=cardiology&lat=28.6&lon=77.2&radius=50
GET    /api/v1/doctors/{id}
POST   /api/v1/doctors/appointment
GET    /api/v1/doctors/availability/{id}
```

**Emergency Services**
```
POST   /api/v1/emergency/detect
POST   /api/v1/emergency/ambulance/book
GET    /api/v1/emergency/status/{booking_id}
POST   /api/v1/emergency/contacts/notify
```

**Treatment Recommendations**
```
GET    /api/v1/treatment/recommendations?condition=influenza&age=45
GET    /api/v1/treatment/recovery-timeline/{condition}
POST   /api/v1/treatment/follow-up/schedule
```

**Government Schemes**
```
GET    /api/v1/schemes/eligible
GET    /api/v1/schemes/{scheme_id}
POST   /api/v1/schemes/enroll
GET    /api/v1/schemes/benefits
```

### 6.2 Service Communication

#### 6.2.1 Synchronous Communication (REST)

**Use Cases**
- User-facing requests requiring immediate response
- Simple request-response patterns
- Low-latency requirements

**Implementation**
- HTTP/REST with JSON payloads
- Service discovery via Consul/Eureka
- Circuit breaker pattern (Hystrix/Resilience4j)
- Retry logic with exponential backoff
- Timeout configuration (5 seconds default)

**Circuit Breaker**
- Opens after 5 consecutive failures
- Half-open state after 30 seconds
- Closes after 3 successful requests
- Fallback responses for degraded service

#### 6.2.2 Asynchronous Communication (Message Queue)

**Use Cases**
- Non-critical operations (notifications, analytics)
- Long-running processes
- Event-driven workflows
- Decoupling services

**Technology**
- RabbitMQ or Apache Kafka
- Dead letter queues for failed messages
- Message persistence for reliability
- Consumer groups for scalability

**Message Patterns**

**Publish-Subscribe**
- Event: SymptomCheckCompleted
- Subscribers: Analytics Service, Notification Service

**Work Queue**
- Task: SendNotification
- Workers: Multiple notification service instances

**Request-Reply**
- Request: GenerateTreatmentPlan
- Reply: TreatmentPlanGenerated

**Event Examples**
```json
{
  "event_type": "SymptomCheckCompleted",
  "event_id": "uuid",
  "timestamp": "2026-02-06T10:30:00Z",
  "data": {
    "user_id": "12345",
    "diagnosis": "Influenza",
    "confidence": 0.78,
    "risk_category": "moderate"
  }
}
```


### 6.3 Authentication and Authorization

#### 6.3.1 Authentication

**Phone-Based Authentication**
- Primary method for rural users
- OTP sent via SMS
- OTP valid for 10 minutes
- Rate limiting: 3 OTPs per hour per phone number

**Authentication Flow**
1. User enters phone number
2. Backend generates 6-digit OTP
3. OTP sent via SMS gateway
4. User enters OTP
5. Backend validates OTP
6. JWT tokens issued (access + refresh)
7. Tokens stored securely on device

**Token Management**
- Access token: 1-hour expiry, contains user claims
- Refresh token: 30-day expiry, used to get new access token
- Tokens signed with RS256 (asymmetric encryption)
- Token revocation list for logout/security

**Token Structure**
```json
{
  "user_id": "12345",
  "phone": "+919876543210",
  "role": "patient",
  "iat": 1675680000,
  "exp": 1675683600
}
```

**Biometric Authentication (Mobile)**
- Fingerprint/Face ID for quick access
- Biometric data never leaves device
- Falls back to PIN if biometric fails
- Re-authentication required after 7 days

#### 6.3.2 Authorization

**Role-Based Access Control (RBAC)**

**Roles**
- Patient: Basic access to own data
- Family Member: Access to linked family profiles
- Healthcare Worker (ASHA/ANM): Access to community data
- Doctor: Access to referred patients
- Admin: System administration
- Auditor: Read-only access for compliance

**Permissions**
```
Patient:
- Read own profile and medical history
- Create symptom checks
- Book appointments
- View treatment recommendations

Healthcare Worker:
- Read patient data (with consent)
- Create symptom checks on behalf of patients
- Override AI recommendations
- Access community health analytics

Doctor:
- Read referred patient data
- Update diagnosis and treatment
- Prescribe medications
- Access patient medical history

Admin:
- Manage users and roles
- Configure system settings
- Access all data (with audit logging)
```

**Authorization Enforcement**
- Middleware checks JWT claims
- Service-level authorization checks
- Database-level row security (PostgreSQL RLS)
- Audit logging of all access attempts

**Consent Management**
- Explicit consent required for data sharing
- Granular consent (per data type, per recipient)
- Consent can be revoked anytime
- Audit trail of consent changes

---

## 7. Frontend Design

### 7.1 Mobile-First and Accessibility Principles

#### 7.1.1 Mobile-First Design

**Design Approach**
- Design for smallest screen first (320px width)
- Progressive enhancement for larger screens
- Touch-friendly UI (minimum 44x44px tap targets)
- Thumb-friendly navigation (bottom navigation bar)

**Performance Optimization**
- Lazy loading of images and components
- Code splitting for faster initial load
- Prefetching of likely next screens
- Optimistic UI updates (show result before server confirms)

**Responsive Breakpoints**
- Mobile: 320px - 767px
- Tablet: 768px - 1023px
- Desktop: 1024px+

**Offline-First UI**
- Clear offline indicator
- Cached content available offline
- Queue actions for later sync
- Graceful degradation of features

#### 7.1.2 Accessibility Features

**Visual Accessibility**
- High contrast mode (WCAG AA compliant)
- Adjustable font size (16px - 24px)
- Color-blind friendly palette
- No information conveyed by color alone
- Clear visual hierarchy

**Screen Reader Support**
- Semantic HTML elements
- ARIA labels for interactive elements
- Descriptive alt text for images
- Logical tab order
- Skip navigation links

**Motor Accessibility**
- Large touch targets (minimum 44x44px)
- No time-based interactions (or generous timeouts)
- Swipe gestures optional (button alternatives)
- Voice input as alternative to typing

**Cognitive Accessibility**
- Simple, clear language
- Consistent navigation patterns
- Progress indicators for multi-step flows
- Error prevention and clear error messages
- Undo functionality where possible

### 7.2 Multilingual and Voice Interface Design

#### 7.2.1 Multilingual Support

**Supported Languages**
- Hindi, English (default)
- Regional: Tamil, Telugu, Bengali, Marathi, Gujarati, Kannada, Malayalam, Punjabi, Odia, Assamese

**Implementation**
- i18n library (react-i18next)
- Language files stored locally and in cloud
- Automatic language detection from device settings
- Manual language switcher always visible
- Right-to-left (RTL) support where needed

**Translation Strategy**
- Professional translation services
- Medical terminology reviewed by regional experts
- Cultural adaptation, not just literal translation
- Regular updates and corrections
- User feedback on translations

**Content Localization**
- Date and time formats
- Number formats
- Currency (₹)
- Measurement units (metric)
- Cultural references and examples

#### 7.2.2 Voice Interface Design

**Voice Input**
- Always-available microphone button
- Visual feedback during recording
- Noise cancellation
- Support for regional accents
- Fallback to text if voice fails

**Voice Output**
- Text-to-speech for all content
- Natural-sounding voices (Google TTS/AWS Polly)
- Adjustable speech rate
- Pause/resume controls
- Headphone-friendly (doesn't auto-play in public)

**Voice Commands**
- "Check symptoms"
- "Find doctor"
- "Emergency"
- "My profile"
- "Repeat" (repeat last message)

**Voice Conversation Flow**
```
System: "नमस्ते! आपकी क्या समस्या है?" (Hello! What's your problem?)
User: "मुझे बुखार है" (I have fever)
System: "कितने दिन से?" (Since how many days?)
User: "दो दिन" (Two days)
System: "और कोई लक्षण?" (Any other symptoms?)
User: "सिर दर्द" (Headache)
System: "ठीक है, मैं जांच कर रहा हूं..." (Okay, checking...)
```

**Voice Error Handling**
- "I didn't understand, please repeat"
- "Did you mean [symptom]?"
- Option to switch to text input
- Retry with clearer pronunciation guidance

### 7.3 Key UI Screens

#### 7.3.1 Home Screen

**Layout**
- Large "Check Symptoms" button (primary action)
- Emergency button (red, always visible)
- Quick access to recent symptom checks
- Health tips carousel
- Scheme enrollment status

**Features**
- Voice activation: "Hey Health" (optional)
- Offline indicator
- Language switcher
- Profile icon (top right)
- Bottom navigation: Home, History, Doctors, Profile

#### 7.3.2 Symptom Input Screen

**Input Methods**
- Voice input (primary)
- Text search with autocomplete
- Guided questionnaire
- Body part selector (visual)

**Progressive Disclosure**
- Start with main symptom
- Follow-up questions based on input
- Optional detailed questions
- Skip option for non-critical questions

**Visual Feedback**
- Selected symptoms shown as chips
- Progress indicator (e.g., "3 of 5 questions")
- Confidence meter (updates as more info provided)

#### 7.3.3 Results Screen

**Layout**
- Risk indicator (color-coded: red/yellow/green)
- Top 3 probable conditions with confidence bars
- Explanation in plain language
- Treatment recommendations (expandable)
- Nearby doctors (map + list)
- Action buttons: Book Doctor, Set Reminder, Share

**Emergency Results**
- Full-screen red alert
- Large "Call Ambulance" button
- First aid instructions
- Nearest hospital information
- Emergency contact quick dial

#### 7.3.4 Doctor Discovery Screen

**Filters**
- Specialty
- Distance (5km, 10km, 25km, 50km)
- Government/Private
- Availability (today, tomorrow, this week)
- Consultation fee range

**Doctor Card**
- Name and photo
- Specialty and qualifications
- Hospital/clinic name
- Distance and travel time
- Availability indicator
- Consultation fee
- Ratings (if available)
- "Book Appointment" button

**Map View**
- Doctors plotted on map
- User location marker
- Tap marker for doctor details
- Navigation integration

#### 7.3.5 Profile Screen

**Sections**
- Personal information (editable)
- Family members (add/edit)
- Medical history
- Emergency contacts
- Language preference
- Privacy settings
- Government scheme status
- App settings

**Medical History**
- Chronic conditions
- Past diagnoses
- Medications
- Allergies
- Surgeries
- Immunizations

---

## 8. Offline-First and Low-Bandwidth Strategy

### 8.1 Offline-First Architecture

#### 8.1.1 Data Caching Strategy

**Essential Data (Always Cached)**
- Symptom questionnaires (all languages)
- Common disease information
- Treatment guidelines for common conditions
- Emergency first aid instructions
- User profile and medical history
- Recent symptom check results
- Nearby doctor list (last fetched)

**Cache Storage**
- Mobile: SQLite database (structured data) + File system (media)
- Web: IndexedDB (structured data) + Cache API (assets)
- Cache size limit: 50 MB (configurable)
- Automatic cleanup of old data (LRU eviction)

**Cache Update Strategy**
- Background sync when connected to WiFi
- Incremental updates (only changed data)
- Version checking to avoid stale data
- User-initiated refresh option

#### 8.1.2 Offline Functionality

**Fully Functional Offline**
- Symptom checking (basic AI model)
- View cached medical content
- Access medical history
- View recent symptom checks
- Emergency first aid instructions
- Doctor information (last cached)

**Partially Functional Offline**
- Doctor search (cached results only)
- Scheme information (may be outdated)
- Treatment recommendations (limited to cached conditions)

**Not Available Offline**
- Real-time doctor availability
- Ambulance booking (SMS fallback available)
- New scheme enrollment
- Live location tracking

#### 8.1.3 Offline AI Model

**Model Characteristics**
- Lightweight version of full model
- Covers 50 most common conditions (90% of cases)
- Smaller feature set (core symptoms only)
- Faster inference (<1 second)
- Model size: <10 MB

**Offline vs. Online Comparison**
- Offline accuracy: ~75% (vs. 85% online)
- Offline coverage: 50 conditions (vs. 500+ online)
- Offline confidence: Generally lower
- Clear indication to user that analysis is offline

**Model Updates**
- Updated monthly via app update or background download
- Differential updates to minimize data usage
- Automatic update on WiFi
- Manual update option

### 8.2 Low-Bandwidth Optimization

#### 8.2.1 Data Minimization

**API Response Optimization**
- Compress responses (gzip/brotli)
- Pagination for large lists (20 items per page)
- Field filtering (only requested fields returned)
- Thumbnail images instead of full resolution
- Lazy loading of non-critical data

**Example: Doctor Search Response**
```json
{
  "doctors": [
    {
      "id": "123",
      "name": "Dr. Sharma",
      "specialty": "General Medicine",
      "distance": 5.2,
      "fee": 100,
      "available": true
      // Full details fetched on demand
    }
  ],
  "total": 45,
  "page": 1,
  "size": 20
}
```

#### 8.2.2 Image Optimization

**Strategies**
- WebP format (30-50% smaller than JPEG)
- Multiple resolutions (serve based on device)
- Lazy loading (load as user scrolls)
- Placeholder images (low-res blur)
- No images for critical flows (use icons)

**Image Sizes**
- Thumbnail: 100x100px, <5 KB
- Small: 300x300px, <20 KB
- Medium: 600x600px, <50 KB
- Large: 1200x1200px, <150 KB

**Adaptive Image Loading**
- 2G: Thumbnails only
- 3G: Small images
- 4G/WiFi: Medium/Large images

#### 8.2.3 Progressive Loading

**Approach**
- Load critical content first
- Show skeleton screens during loading
- Load non-critical content in background
- Prioritize above-the-fold content

**Example: Symptom Results Screen**
1. Load risk indicator and top diagnosis (0.5s)
2. Load explanation and confidence scores (1s)
3. Load treatment recommendations (1.5s)
4. Load doctor recommendations (2s)
5. Load map and additional details (3s)

#### 8.2.4 Network Detection and Adaptation

**Connection Types**
- Offline: Use cached data only
- 2G (<100 kbps): Minimal data, text-only
- 3G (100-1000 kbps): Optimized images, essential features
- 4G/WiFi (>1000 kbps): Full features, high-quality images

**Adaptive Behavior**
```javascript
if (connectionType === '2G') {
  disableImages();
  reducePaginationSize(10);
  disableAutoRefresh();
} else if (connectionType === '3G') {
  enableThumbnails();
  reducePaginationSize(15);
  setAutoRefreshInterval(60000); // 1 minute
} else {
  enableFullImages();
  setPaginationSize(20);
  setAutoRefreshInterval(30000); // 30 seconds
}
```

### 8.3 SMS Fallback System

#### 8.3.1 SMS Commands

**Symptom Check**
```
SMS to: 1234567890
Format: SYMPTOM <symptoms> <duration>
Example: SYMPTOM fever cough 2days

Response:
"Possible: Flu (75%). Rest, fluids, paracetamol. 
See doctor if worsens. Nearby: Dr. Sharma, 5km, 
Ph: 9876543210"
```

**Emergency**
```
SMS: EMERGENCY <location>
Example: EMERGENCY Village Rampur, Dist Meerut

Response:
"Ambulance booked. ETA 15 min. First aid: 
Keep patient calm, lying down. Emergency: 108"
```

**Doctor Search**
```
SMS: DOCTOR <specialty> <pincode>
Example: DOCTOR general 250001

Response:
"1. Dr. Sharma, Govt Hospital, 5km, Free
2. Dr. Verma, City Clinic, 8km, Rs.200
Call 9876543210 for appointment"
```

**Scheme Check**
```
SMS: SCHEME CHECK

Response:
"You are eligible for Ayushman Bharat. 
Benefits: Free treatment up to 5 lakh. 
SMS SCHEME ENROLL to start enrollment."
```

#### 8.3.2 SMS Gateway Integration

**Provider**
- Primary: Twilio
- Secondary: MSG91 (India-specific)
- Tertiary: AWS SNS

**Features**
- Two-way SMS (send and receive)
- Delivery reports
- Retry logic for failed messages
- Rate limiting (to prevent abuse)
- Cost optimization (batching, compression)

**SMS Processing Pipeline**
1. Receive SMS from user
2. Parse command and parameters
3. Validate user (phone number lookup)
4. Process request (call appropriate service)
5. Format response (160 characters max)
6. Send response SMS
7. Log interaction for analytics

---

## 9. Security and Privacy Design

### 9.1 Encryption

#### 9.1.1 Data at Rest

**Database Encryption**
- PostgreSQL: Transparent Data Encryption (TDE)
- Encryption algorithm: AES-256
- Key management: AWS KMS / Azure Key Vault
- Separate keys for different data types
- Key rotation every 90 days

**File Storage Encryption**
- S3 server-side encryption (SSE-S3)
- Encryption for all uploaded files
- Encrypted backups

**Application-Level Encryption**
- Sensitive fields (medical history, allergies) encrypted separately
- Encryption before database storage
- Decryption only when needed
- Keys stored in secure key management system


#### 9.1.2 Data in Transit

**TLS Configuration**
- TLS 1.3 for all API communications
- Strong cipher suites only
- Perfect forward secrecy (PFS)
- Certificate pinning in mobile apps
- HSTS (HTTP Strict Transport Security) enabled

**API Security**
- All APIs accessible only via HTTPS
- No sensitive data in URLs or query parameters
- Request/response encryption for sensitive endpoints
- Mutual TLS for service-to-service communication

**Mobile App Security**
- Certificate pinning to prevent MITM attacks
- Encrypted local storage (SQLite encryption)
- Secure keychain/keystore for tokens
- Obfuscated code to prevent reverse engineering

### 9.2 Consent Management

#### 9.2.1 Consent Framework

**Consent Types**
- Data collection and storage
- AI analysis of health data
- Data sharing with healthcare providers
- Data sharing with government agencies
- Research and analytics (anonymized)
- Marketing communications

**Consent Characteristics**
- Explicit opt-in (no pre-checked boxes)
- Granular (per data type and purpose)
- Revocable at any time
- Time-bound (annual renewal)
- Documented with audit trail

**Consent Flow**
1. User registers
2. Presented with clear consent options
3. Plain language explanation of each consent
4. User selects preferences
5. Consent recorded with timestamp
6. Confirmation sent to user
7. Periodic consent renewal reminders

#### 9.2.2 Data Sharing Controls

**User Controls**
- View who has accessed their data
- Revoke access to specific entities
- Download their data (data portability)
- Delete their account and data
- Opt-out of specific data uses

**Healthcare Provider Access**
- Requires explicit patient consent
- Time-limited access (default: 30 days)
- Purpose-specific (e.g., "for consultation on 2026-02-06")
- Audit trail of all access
- Automatic expiry and notification

**Government Access**
- Only for public health purposes
- Anonymized and aggregated data
- Legal basis documented
- Oversight by data protection officer
- Annual transparency report

### 9.3 Role-Based Access Control (RBAC)

#### 9.3.1 Access Control Matrix

```
Resource              | Patient | Healthcare Worker | Doctor | Admin | Auditor
--------------------- | ------- | ----------------- | ------ | ----- | --------
Own Profile           | RW      | -                 | -      | RW    | R
Own Medical History   | RW      | -                 | -      | RW    | R
Family Member Profile | RW      | -                 | -      | RW    | R
Other Patient Data    | -       | R (with consent)  | R (ref)| RW    | R
Symptom Check         | RW      | RW                | R      | RW    | R
Treatment Plan        | R       | R                 | RW     | RW    | R
Doctor Directory      | R       | R                 | RW     | RW    | R
Scheme Information    | R       | R                 | R      | RW    | R
System Configuration  | -       | -                 | -      | RW    | R
Audit Logs            | -       | -                 | -      | R     | R

R = Read, W = Write, - = No Access
```

#### 9.3.2 Access Control Implementation

**Authentication Layer**
- JWT token validation
- Token expiry checking
- Refresh token rotation
- Session management

**Authorization Layer**
- Role extraction from JWT claims
- Permission checking middleware
- Resource ownership validation
- Consent verification

**Database Layer**
- PostgreSQL Row-Level Security (RLS)
- Policies based on user role
- Automatic filtering of unauthorized data
- Prevents SQL injection attacks

**Example RLS Policy**
```sql
CREATE POLICY patient_own_data ON medical_records
  FOR SELECT
  USING (user_id = current_user_id());

CREATE POLICY doctor_referred_patients ON medical_records
  FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM referrals
      WHERE patient_id = medical_records.user_id
      AND doctor_id = current_user_id()
      AND consent_given = true
      AND expiry_date > NOW()
    )
  );
```

### 9.4 Audit Logging

#### 9.4.1 Logged Events

**User Actions**
- Login/logout
- Profile updates
- Symptom checks
- Doctor bookings
- Consent changes
- Data downloads
- Account deletion

**System Actions**
- Data access by healthcare workers
- AI model predictions
- Emergency alerts
- Ambulance bookings
- Data sharing events
- Configuration changes

**Security Events**
- Failed login attempts
- Unauthorized access attempts
- Token expiry/refresh
- Suspicious activity
- Data breaches (if any)

#### 9.4.2 Audit Log Format

```json
{
  "event_id": "uuid",
  "timestamp": "2026-02-06T10:30:00Z",
  "event_type": "DATA_ACCESS",
  "actor": {
    "user_id": "12345",
    "role": "doctor",
    "ip_address": "192.168.1.1"
  },
  "resource": {
    "type": "medical_record",
    "id": "67890",
    "owner_id": "54321"
  },
  "action": "READ",
  "result": "SUCCESS",
  "metadata": {
    "consent_id": "consent-123",
    "purpose": "consultation"
  }
}
```

#### 9.4.3 Audit Log Management

**Storage**
- Separate audit database (write-only)
- Immutable logs (append-only)
- Encrypted storage
- Long-term retention (7 years minimum)

**Access**
- Restricted to auditors and compliance team
- All access to audit logs is itself logged
- Regular review by data protection officer
- Available for regulatory inspection

**Monitoring**
- Real-time alerting for suspicious patterns
- Automated anomaly detection
- Dashboard for security team
- Regular audit reports

### 9.5 Compliance with Indian Regulations

#### 9.5.1 Digital Personal Data Protection Act (DPDP Act)

**Key Requirements**
- Lawful basis for data processing
- Purpose limitation
- Data minimization
- Accuracy and retention limits
- Security safeguards
- Data subject rights
- Accountability and governance

**Implementation**
- Privacy policy in plain language
- Consent management system
- Data protection impact assessments
- Data protection officer appointed
- Breach notification procedures
- Data localization (data stored in India)

#### 9.5.2 ABDM (Ayushman Bharat Digital Mission) Compliance

**Standards**
- HL7 FHIR for health data exchange
- ABDM Health ID integration
- Consent management as per ABDM framework
- Interoperability with ABDM ecosystem
- Security and privacy as per ABDM guidelines

**Integration**
- Health ID creation and linking
- Health record exchange via ABDM gateway
- Consent artifacts as per ABDM specification
- Participation in ABDM health information exchange

#### 9.5.3 Medical Device Regulations

**Classification**
- Platform positioned as "health information tool"
- Not a medical device (no diagnostic claims)
- Clear disclaimers about AI limitations
- Recommendation to consult healthcare professionals

**Compliance**
- Follow Telemedicine Practice Guidelines (2020)
- No prescription generation without doctor
- Clear distinction between AI assistance and medical advice
- Medical advisory board oversight

#### 9.5.4 Other Regulations

**Information Technology Act, 2000**
- Reasonable security practices
- Data breach notification
- Liability for data breaches

**Consumer Protection Act, 2019**
- Transparent terms of service
- No unfair trade practices
- Grievance redressal mechanism

**Indian Medical Council Regulations**
- No unauthorized practice of medicine
- Respect for doctor-patient relationship
- Ethical guidelines for health information

---

## 10. Scalability, Availability, and Reliability

### 10.1 Scalability

#### 10.1.1 Horizontal Scaling

**Stateless Services**
- All backend services designed to be stateless
- Session state stored in Redis (shared)
- No server affinity required
- Easy to add/remove instances

**Auto-Scaling**
- Kubernetes for container orchestration
- Horizontal Pod Autoscaler (HPA)
- Metrics: CPU, memory, request rate
- Scale up: >70% CPU for 2 minutes
- Scale down: <30% CPU for 5 minutes
- Min replicas: 3, Max replicas: 50

**Load Balancing**
- Application Load Balancer (ALB)
- Round-robin distribution
- Health checks every 30 seconds
- Automatic removal of unhealthy instances
- Session stickiness for WebSocket connections

#### 10.1.2 Database Scaling

**Read Replicas**
- PostgreSQL with streaming replication
- 3 read replicas per region
- Read-heavy queries routed to replicas
- Automatic failover to replica if primary fails

**Sharding Strategy**
- Shard by user_id (consistent hashing)
- Each shard handles 1 million users
- Shard routing in application layer
- Future: Automatic shard rebalancing

**Caching**
- Redis cluster (3 master + 3 replica nodes)
- Cache-aside pattern
- TTL-based expiration
- Cache warming for common queries

**Connection Pooling**
- PgBouncer for PostgreSQL
- Pool size: 100 connections per service
- Connection reuse
- Timeout: 30 seconds

#### 10.1.3 AI Model Scaling

**Model Serving**
- TensorFlow Serving or TorchServe
- GPU instances for inference (optional)
- Batch prediction for efficiency
- Model versioning and A/B testing

**Scaling Strategy**
- Separate service for AI inference
- Auto-scaling based on request queue length
- Async processing for non-urgent predictions
- Caching of common symptom patterns

**Performance Optimization**
- Model quantization (reduce size)
- ONNX runtime for faster inference
- Batch processing (up to 10 requests)
- GPU acceleration for complex models

### 10.2 Availability

#### 10.2.1 High Availability Architecture

**Multi-Region Deployment**
- Primary region: Mumbai (AWS ap-south-1)
- Secondary region: Hyderabad (AWS ap-south-2)
- Active-active for read operations
- Active-passive for write operations
- Automatic failover in case of region failure

**Redundancy**
- Multiple availability zones per region
- Services deployed across 3 AZs
- Database replicas in different AZs
- Load balancer across AZs

**Health Checks**
- Application health endpoint: /health
- Database connectivity check
- Dependency health check (Redis, message queue)
- Liveness and readiness probes in Kubernetes

#### 10.2.2 Disaster Recovery

**Backup Strategy**
- Database: Automated daily backups
- Incremental backups every 6 hours
- Point-in-time recovery (PITR) enabled
- Backup retention: 30 days
- Cross-region backup replication

**Recovery Objectives**
- RTO (Recovery Time Objective): 1 hour
- RPO (Recovery Point Objective): 6 hours
- Emergency services: RTO 15 minutes, RPO 1 hour

**Disaster Recovery Plan**
1. Detect failure (automated monitoring)
2. Assess impact and severity
3. Activate DR plan
4. Failover to secondary region
5. Restore services from backups
6. Verify data integrity
7. Resume normal operations
8. Post-mortem and improvements

#### 10.2.3 Graceful Degradation

**Service Degradation Levels**

**Level 1: Full Functionality**
- All services operational
- Real-time data
- Full AI capabilities

**Level 2: Reduced Functionality**
- Core services operational
- Cached data used
- Simplified AI models
- Non-critical features disabled

**Level 3: Emergency Only**
- Emergency services only
- SMS fallback active
- Cached data only
- Manual processes for critical operations

**Degradation Triggers**
- High error rate (>5%)
- High latency (>5 seconds)
- Database unavailability
- AI service failure
- External API failures

### 10.3 Reliability

#### 10.3.1 Error Handling

**Retry Logic**
- Exponential backoff (1s, 2s, 4s, 8s)
- Maximum 3 retries
- Idempotency for all write operations
- Circuit breaker to prevent cascading failures

**Fallback Mechanisms**
- Cached data if API fails
- Simplified AI model if main model fails
- SMS if push notification fails
- Manual process if automation fails

**Error Monitoring**
- Sentry for error tracking
- Real-time alerts for critical errors
- Error rate monitoring
- Automatic incident creation for high-severity errors

#### 10.3.2 Monitoring and Alerting

**Metrics Collected**
- System metrics: CPU, memory, disk, network
- Application metrics: Request rate, latency, error rate
- Business metrics: Symptom checks, emergency bookings, user registrations
- AI metrics: Model accuracy, inference time, confidence distribution

**Monitoring Tools**
- Prometheus for metrics collection
- Grafana for visualization
- ELK stack for log aggregation
- Jaeger for distributed tracing

**Alerting Rules**
- Critical: Page on-call engineer immediately
  * Emergency service down
  * Database unavailable
  * Error rate >10%
- High: Alert within 15 minutes
  * API latency >5 seconds
  * Error rate >5%
  * Disk usage >80%
- Medium: Alert within 1 hour
  * Cache hit rate <70%
  * AI model accuracy drop >5%
  * Unusual traffic patterns

**On-Call Rotation**
- 24/7 on-call coverage
- Primary and secondary on-call
- Escalation after 15 minutes
- Post-incident reviews

#### 10.3.3 Testing

**Unit Testing**
- 80% code coverage minimum
- Critical modules: 95% coverage
- Automated test execution on every commit

**Integration Testing**
- API contract testing
- Service-to-service integration tests
- Database integration tests
- External API mocking

**End-to-End Testing**
- Critical user flows automated
- Symptom check flow
- Emergency booking flow
- Doctor search flow
- Run daily in staging environment

**Load Testing**
- Simulate 1 million concurrent users
- Identify bottlenecks
- Validate auto-scaling
- Run quarterly

**Chaos Engineering**
- Randomly kill service instances
- Simulate network failures
- Database failover testing
- Region failure simulation
- Run monthly in staging

---

## 11. Deployment Architecture

### 11.1 Infrastructure

#### 11.1.1 Cloud Provider

**Primary: AWS (Amazon Web Services)**
- Mature services and ecosystem
- Strong presence in India (Mumbai, Hyderabad regions)
- Compliance certifications (ISO, SOC, PCI-DSS)
- Government cloud (AWS GovCloud) for sensitive data

**Backup: Azure or GCP**
- Multi-cloud strategy for vendor independence
- Disaster recovery in alternate cloud
- Cost optimization through cloud arbitrage

#### 11.1.2 Compute

**Kubernetes (EKS - Elastic Kubernetes Service)**
- Container orchestration
- Auto-scaling and self-healing
- Rolling updates and rollbacks
- Service mesh (Istio) for advanced traffic management

**Node Groups**
- General purpose: t3.medium (2 vCPU, 4 GB RAM)
- AI inference: g4dn.xlarge (GPU instances)
- Emergency services: c5.large (compute-optimized)
- Min nodes: 10, Max nodes: 100

**Serverless (AWS Lambda)**
- Non-critical background tasks
- Scheduled jobs (reminders, cleanup)
- Event-driven processing
- Cost-effective for sporadic workloads

#### 11.1.3 Storage

**Database**
- Amazon RDS for PostgreSQL (Multi-AZ)
- Instance: db.r5.2xlarge (8 vCPU, 64 GB RAM)
- Storage: 1 TB SSD (auto-scaling enabled)
- Automated backups and PITR

**Cache**
- Amazon ElastiCache for Redis
- Cluster mode enabled (3 shards)
- Multi-AZ with automatic failover
- Instance: cache.r5.large (2 vCPU, 13 GB RAM)

**Object Storage**
- Amazon S3 for files and backups
- Lifecycle policies for cost optimization
- Versioning enabled
- Cross-region replication

**Search**
- Amazon OpenSearch (Elasticsearch)
- 3-node cluster (Multi-AZ)
- Instance: r5.large.search
- Used for doctor search and medical content

#### 11.1.4 Networking

**VPC (Virtual Private Cloud)**
- Isolated network per environment
- Public subnets for load balancers
- Private subnets for application and database
- NAT Gateway for outbound internet access

**Load Balancer**
- Application Load Balancer (ALB)
- SSL/TLS termination
- Path-based routing
- Health checks and auto-scaling integration

**CDN (CloudFront)**
- Global content delivery
- Edge caching for static assets
- DDoS protection
- Custom domain and SSL certificate

**API Gateway**
- Kong or AWS API Gateway
- Rate limiting and throttling
- API versioning
- Request/response transformation


### 11.2 Deployment Pipeline

#### 11.2.1 CI/CD Pipeline

**Source Control**
- Git (GitHub/GitLab)
- Branch strategy: GitFlow
- Protected main branch
- Pull request reviews required

**Continuous Integration**
- GitHub Actions or Jenkins
- Triggered on every commit
- Steps:
  1. Code checkout
  2. Dependency installation
  3. Linting and code quality checks
  4. Unit tests
  5. Integration tests
  6. Security scanning (SAST)
  7. Build Docker images
  8. Push to container registry
  9. Tag with commit SHA

**Continuous Deployment**
- Automated deployment to staging
- Manual approval for production
- Blue-green deployment strategy
- Automatic rollback on failure

**Deployment Stages**
1. Development: Auto-deploy on commit to dev branch
2. Staging: Auto-deploy on commit to staging branch
3. Production: Manual approval after staging validation

#### 11.2.2 Environment Strategy

**Development**
- Shared environment for developers
- Frequent deployments (multiple per day)
- Mock external services
- Reduced resource allocation

**Staging**
- Production-like environment
- Integration with real external services (test accounts)
- Full resource allocation
- Used for final testing before production

**Production**
- Live environment serving real users
- Multi-region deployment
- High availability and redundancy
- Strict change control

**Environment Configuration**
- Environment variables for configuration
- Secrets stored in AWS Secrets Manager
- Infrastructure as Code (Terraform)
- Configuration versioning

### 11.3 Monitoring and Observability

#### 11.3.1 Logging

**Log Aggregation**
- ELK Stack (Elasticsearch, Logstash, Kibana)
- Centralized logging from all services
- Structured logging (JSON format)
- Log retention: 30 days (hot), 1 year (cold)

**Log Levels**
- ERROR: Critical issues requiring immediate attention
- WARN: Potential issues, degraded functionality
- INFO: Important business events
- DEBUG: Detailed diagnostic information (dev/staging only)

**Log Format**
```json
{
  "timestamp": "2026-02-06T10:30:00Z",
  "level": "INFO",
  "service": "symptom-service",
  "trace_id": "abc123",
  "user_id": "12345",
  "message": "Symptom check completed",
  "metadata": {
    "diagnosis": "Influenza",
    "confidence": 0.78,
    "duration_ms": 1234
  }
}
```

#### 11.3.2 Metrics

**Application Metrics**
- Request rate (requests per second)
- Response time (p50, p95, p99)
- Error rate (percentage)
- Active users (concurrent)
- Business metrics (symptom checks, bookings)

**Infrastructure Metrics**
- CPU utilization
- Memory usage
- Disk I/O
- Network throughput
- Database connections

**AI Model Metrics**
- Inference time
- Model accuracy (online evaluation)
- Confidence distribution
- Emergency detection rate

**Dashboards**
- Executive dashboard (high-level KPIs)
- Operations dashboard (system health)
- Business dashboard (user engagement)
- AI dashboard (model performance)

#### 11.3.3 Tracing

**Distributed Tracing**
- Jaeger for trace collection
- Trace every request across services
- Identify bottlenecks and latency issues
- Correlation with logs and metrics

**Trace Context**
- Unique trace ID per request
- Propagated across service boundaries
- Includes user context and metadata
- Sampling: 100% for errors, 10% for success

### 11.4 Security in Deployment

#### 11.4.1 Infrastructure Security

**Network Security**
- VPC with private subnets
- Security groups (firewall rules)
- Network ACLs
- No direct internet access to databases
- VPN for administrative access

**Access Control**
- IAM roles and policies (least privilege)
- Multi-factor authentication (MFA) required
- Temporary credentials (STS)
- Regular access reviews

**Secrets Management**
- AWS Secrets Manager for sensitive data
- Automatic secret rotation
- Encryption at rest and in transit
- Audit logging of secret access

#### 11.4.2 Application Security

**Container Security**
- Base images from trusted sources
- Regular image scanning (Trivy, Clair)
- No secrets in images
- Non-root user in containers
- Read-only file systems where possible

**Dependency Management**
- Regular dependency updates
- Automated vulnerability scanning (Snyk, Dependabot)
- Security patches applied within 7 days
- License compliance checking

**Security Scanning**
- SAST (Static Application Security Testing): SonarQube
- DAST (Dynamic Application Security Testing): OWASP ZAP
- Dependency scanning: Snyk
- Container scanning: Trivy
- Infrastructure scanning: Checkov

#### 11.4.3 Compliance and Auditing

**Compliance Monitoring**
- AWS Config for compliance checking
- Automated remediation of non-compliant resources
- Regular compliance reports
- Third-party audits (annual)

**Audit Logging**
- CloudTrail for AWS API calls
- Application audit logs
- Immutable audit trail
- Regular audit log reviews

---

## 12. Key Design Trade-offs and Decisions

### 12.1 Technology Choices

#### 12.1.1 React Native vs. Native Apps

**Decision: React Native**

**Rationale**
- Single codebase for Android and iOS (faster development)
- Large community and ecosystem
- Good performance for our use case
- Easy to find developers in India

**Trade-offs**
- Slightly lower performance than native
- Some platform-specific features require native modules
- Larger app size

**Mitigation**
- Use native modules for performance-critical features
- Optimize bundle size with code splitting
- Profile and optimize performance regularly

#### 12.1.2 Microservices vs. Monolith

**Decision: Microservices**

**Rationale**
- Independent scaling of services
- Technology flexibility per service
- Fault isolation (one service failure doesn't bring down entire system)
- Team autonomy (different teams can own different services)

**Trade-offs**
- Increased complexity (service discovery, distributed tracing)
- Network latency between services
- Data consistency challenges
- Higher operational overhead

**Mitigation**
- Start with coarse-grained services (not too many)
- Use service mesh for observability
- Implement saga pattern for distributed transactions
- Strong DevOps practices and automation

#### 12.1.3 SQL vs. NoSQL

**Decision: Hybrid (PostgreSQL + MongoDB + Redis)**

**Rationale**
- PostgreSQL: Structured data (users, medical records) - ACID guarantees
- MongoDB: Flexible schema (medical content, treatment guidelines)
- Redis: Caching and session storage

**Trade-offs**
- Multiple databases to manage
- Data synchronization complexity
- Learning curve for team

**Mitigation**
- Clear guidelines on which database for what data
- Automated backup and monitoring for all databases
- Comprehensive documentation

### 12.2 AI/ML Decisions

#### 12.2.1 Rule-Based vs. Pure ML

**Decision: Hybrid (Rules + ML)**

**Rationale**
- Rules provide deterministic behavior for critical cases
- ML handles complex patterns and edge cases
- Rules are explainable and auditable
- ML improves over time with data

**Trade-offs**
- Maintaining both systems
- Potential conflicts between rules and ML
- More complex decision logic

**Mitigation**
- Clear precedence (rules override ML for emergencies)
- Regular review of rules by medical experts
- Monitoring for rule-ML conflicts

#### 12.2.2 Cloud ML vs. On-Device ML

**Decision: Hybrid (Cloud primary, on-device fallback)**

**Rationale**
- Cloud: More powerful models, easier to update
- On-device: Works offline, lower latency, privacy

**Trade-offs**
- On-device models are less accurate
- Model size constraints on device
- Keeping models in sync

**Mitigation**
- Lightweight models for on-device (<10 MB)
- Regular model updates via app updates
- Clear indication to user when using offline model

### 12.3 Architecture Decisions

#### 12.3.1 Synchronous vs. Asynchronous Communication

**Decision: Hybrid**

**Synchronous (REST) for:**
- User-facing requests (symptom check, doctor search)
- Real-time requirements (emergency booking)

**Asynchronous (Message Queue) for:**
- Notifications
- Analytics
- Non-critical background tasks

**Rationale**
- Synchronous: Simpler, immediate response
- Asynchronous: Better scalability, fault tolerance

**Trade-offs**
- Increased complexity with both patterns
- Eventual consistency for async operations

**Mitigation**
- Clear guidelines on when to use each
- Idempotency for all operations
- Monitoring and alerting for message queue health

#### 12.3.2 Multi-Region vs. Single Region

**Decision: Multi-Region (Active-Active for reads, Active-Passive for writes)**

**Rationale**
- High availability (region failure doesn't bring down system)
- Lower latency for users in different regions
- Disaster recovery

**Trade-offs**
- Increased cost (duplicate infrastructure)
- Data consistency challenges
- Complex deployment and testing

**Mitigation**
- Start with two regions (Mumbai, Hyderabad)
- Asynchronous replication for non-critical data
- Strong consistency for critical data (emergency records)
- Regular disaster recovery drills

### 12.4 User Experience Decisions

#### 12.4.1 Voice-First vs. Text-First

**Decision: Voice-First with text fallback**

**Rationale**
- Target users have limited literacy
- Voice is more natural and accessible
- Faster input for describing symptoms

**Trade-offs**
- Voice recognition accuracy challenges (accents, noise)
- Privacy concerns (speaking symptoms in public)
- Higher data usage

**Mitigation**
- High-quality speech recognition (Google/AWS)
- Noise cancellation
- Always provide text alternative
- Offline voice recognition for common phrases

#### 12.4.2 Comprehensive vs. Simple UI

**Decision: Simple UI with progressive disclosure**

**Rationale**
- Target users are not tech-savvy
- Reduce cognitive load
- Faster task completion

**Trade-offs**
- Power users may find it limiting
- More screens/steps for complex tasks

**Mitigation**
- Progressive disclosure (show advanced options on demand)
- Shortcuts for frequent actions
- User testing with target audience

### 12.5 Business Decisions

#### 12.5.1 Free vs. Freemium Model

**Decision: Free for core features, optional premium features**

**Rationale**
- Core mission is to improve healthcare access (should be free)
- Sustainability requires some revenue
- Government partnerships can subsidize costs

**Free Features:**
- Symptom checking
- Emergency services
- Government doctor recommendations
- Government scheme integration

**Premium Features (Future):**
- Priority doctor appointments
- Telemedicine consultations
- Health tracking and analytics
- Family health plans

**Trade-offs**
- Revenue generation challenges
- Potential perception of "pay-to-play"

**Mitigation**
- Clear communication that core features are always free
- Premium features are truly optional
- Government partnerships for sustainability

#### 12.5.2 Government-First vs. Private-First

**Decision: Government-First**

**Rationale**
- Align with mission of improving access for underserved
- Government facilities are free/subsidized
- Build trust with government for partnerships
- Larger reach (government hospitals in every district)

**Trade-offs**
- Government systems may be less reliable
- Slower integration process
- Limited data availability

**Mitigation**
- Fallback to private providers when government unavailable
- Work closely with government for integration
- Provide value to government (analytics, insights)

---

## 13. Future Enhancements

### 13.1 Phase 2 Enhancements (Year 2)

**Telemedicine Video Consultations**
- Video calls with doctors
- Screen sharing for reports
- Prescription generation
- Integration with doctor scheduling

**Lab Test Integration**
- Lab test recommendations based on symptoms
- Lab test booking
- Integration with diagnostic centers
- Digital report delivery

**Medication Reminders**
- Smart medication tracking
- Refill reminders
- Drug interaction warnings
- Adherence tracking

**Health Tracking**
- Vital signs tracking (BP, glucose, weight)
- Symptom diary
- Trend analysis
- Alerts for abnormal values

### 13.2 Phase 3 Enhancements (Year 3)

**Chronic Disease Management**
- Diabetes management program
- Hypertension management
- Asthma management
- Personalized care plans

**Mental Health Services**
- Mental health screening
- Counseling services
- Stress management tools
- Crisis helpline integration

**Wearable Device Integration**
- Fitness tracker integration
- Continuous health monitoring
- Automatic symptom detection
- Emergency alerts based on vitals

**AI Enhancements**
- Predictive health analytics
- Personalized health recommendations
- Disease outbreak prediction
- Population health insights

### 13.3 Phase 4 Enhancements (Year 4+)

**Private Insurance Integration**
- Insurance eligibility checking
- Claim filing assistance
- Cashless treatment facilitation
- Insurance comparison

**E-Pharmacy Integration**
- Medicine ordering
- Home delivery
- Generic medicine suggestions
- Price comparison

**Health Records Management**
- Complete EHR (Electronic Health Record)
- Integration with all healthcare providers
- Health record sharing with consent
- Lifetime health history

**Community Health Features**
- Health forums and support groups
- Peer support for chronic conditions
- Health education content
- Community health challenges

**Advanced AI**
- Medical image analysis (X-rays, scans)
- Genetic risk assessment
- Personalized treatment optimization
- Drug discovery support

### 13.4 Research and Innovation

**AI Research**
- Federated learning for privacy-preserving model training
- Explainable AI for better trust
- Fairness and bias mitigation
- Multi-modal learning (text, voice, images)

**Public Health Research**
- Disease surveillance and outbreak detection
- Health disparity analysis
- Treatment effectiveness studies
- Population health trends

**Partnerships**
- Academic institutions for research
- Healthcare providers for data and validation
- Technology companies for innovation
- NGOs for community outreach

---

## Conclusion

This system design document presents a comprehensive architecture for an AI-powered rural healthcare access platform tailored for India. The design prioritizes accessibility, reliability, and compliance while addressing the unique challenges of rural healthcare delivery.

**Key Strengths:**
- Offline-first architecture for intermittent connectivity
- Voice-first interface for low-literacy users
- Government-first approach for maximum reach
- Robust emergency handling with multiple fallbacks
- Strong privacy and security measures
- Scalable and maintainable architecture

**Success Factors:**
- Close collaboration with government health departments
- Continuous feedback from target users
- Regular medical advisory board reviews
- Strong DevOps and monitoring practices
- Commitment to accessibility and inclusivity

**Next Steps:**
1. Prototype development and user testing
2. Pilot program in 2-3 districts
3. Iterative improvement based on feedback
4. Gradual rollout to additional regions
5. Continuous enhancement and innovation

This platform has the potential to significantly improve healthcare access for millions of underserved Indians, reducing health disparities and improving health outcomes across the country.

---

**Document Version:** 1.0  
**Last Updated:** February 6, 2026  
**Status:** Final  
**Prepared for:** National-Level Hackathon Submission
