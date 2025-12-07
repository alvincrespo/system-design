# Consistent Hashing in Healthcare and Pharmacy: Real-World Scenarios

## Table of Contents

- [Introduction](#introduction)
- [Scenario 1: Multi-Location Pharmacy Network - Prescription Routing and Fulfillment](#scenario-1-multi-location-pharmacy-network---prescription-routing-and-fulfillment)
  - [The System](#the-system)
  - [Why Consistent Hashing Matters](#why-consistent-hashing-matters)
  - [Additional Complexity: Pharmacy Capacity](#additional-complexity-pharmacy-capacity)
- [Scenario 2: Patient Medication History Caching in Electronic Health Records (EHR)](#scenario-2-patient-medication-history-caching-in-electronic-health-records-ehr)
  - [The System](#the-system-1)
  - [Why Consistent Hashing Matters](#why-consistent-hashing-matters-1)
- [Scenario 3: Insurance Eligibility Verification Across a Network of Processing Nodes](#scenario-3-insurance-eligibility-verification-across-a-network-of-processing-nodes)
  - [The System](#the-system-2)
  - [Why Consistent Hashing Matters](#why-consistent-hashing-matters-2)
  - [Additional Healthcare Complexity: Plan Changes](#additional-healthcare-complexity-plan-changes)
- [Key Takeaways for Healthcare/Pharmacy Systems](#key-takeaways-for-healthcarepharmacy-systems)
  - [When to Apply Consistent Hashing in Healthcare Systems](#when-to-apply-consistent-hashing-in-healthcare-systems)

## Introduction

While [consistent hashing](../CONCEPTS/CONSISTENCY_HASHING.md) is a general distributed systems technique, healthcare and pharmacy systems face unique challenges that make it critical. These industries deal with time-sensitive data, strict compliance requirements, and patient safety implications. The three scenarios below illustrate why consistent hashing isn't just a technical optimization—it's essential for safe, reliable healthcare delivery.

---

## Scenario 1: Multi-Location Pharmacy Network - Prescription Routing and Fulfillment

### The System

You're designing the backend for a national pharmacy chain with 2,000 locations. Each location has:
- Its own inventory management system
- Local prescription fulfillment capacity
- A connection to the central routing system

When a prescription comes in (online or in-store), the system must:
1. Route it to an appropriate pharmacy location
2. Check local inventory
3. Manage the fulfillment queue
4. Track real-time status

### Why Consistent Hashing Matters

**Using Modulo Hashing (The Problem):**

Prescriptions are routed using `pharmacy_location = hash(prescription_id) % N` where N = 2,000 locations.

- A prescription gets routed to Pharmacy #573 for fulfillment
- The queue manager caches this prescription's status (pending → awaiting pharmacist review → ready for pickup)
- At 2pm, Pharmacy #453 goes offline for emergency maintenance
- Now N = 1,999
- The hash calculation changes: `hash(prescription_id) % 1999` now points to a different pharmacy
- That prescription (and 99.95% of others) suddenly appear in the wrong pharmacy's system
- Fulfillment queues become corrupted; prescriptions get lost in routing
- Patients show up to pick up medications that the system shows as still pending at different locations
- Staff can't find prescriptions; operations grind to a halt

**Real Impact:**
- Patients waiting for time-sensitive medications (antibiotics, pain management, cancer treatments) experience delays
- Staff spends hours manually reconciling which prescriptions are where
- Pharmacy fills wrong prescriptions because the system shows them in the wrong queue
- Potential patient safety incident if wrong medication is dispensed
- Compliance violation if medications aren't dispensed within promised timeframes

**Using Consistent Hashing (The Solution):**

- Prescription routed to Pharmacy #573
- Status cached locally on Pharmacy #573's queue manager
- Pharmacy #453 goes offline
- Only prescriptions that were assigned to Pharmacy #453 need rerouting (~1 out of 2,000 prescriptions are affected)
- The other 1,999 pharmacies' prescriptions stay exactly where they are
- Affected prescriptions intelligently reroute to nearby locations
- System remains stable; operations continue normally
- Patients experience minimal delays

### Additional Complexity: Pharmacy Capacity

Real pharmacy systems need to consider:
- **Pharmacy-specific inventory:** Different prescriptions may only be fillable at certain locations
- **Service hours:** Some pharmacies are 24-hour; others close at 9pm
- **Specialization:** Pediatric pharmacies, compounding pharmacies, oncology specialists

Consistent hashing with weighted virtual nodes can account for pharmacy capacity and specialization, ensuring traffic is distributed proportional to each location's capability.

---

## Scenario 2: Patient Medication History Caching in Electronic Health Records (EHR)

### The System

You're designing the caching layer for a large hospital network's EHR system serving 50+ facilities. When a healthcare provider opens a patient record, they need instant access to:
- Complete medication history (past 10 years)
- Current medications and dosages
- Drug allergies
- Adverse reactions
- Interaction warnings

This data must be cached across 15 distributed servers because querying the main database for every EHR load would be too slow. Response time SLA: < 200ms.

### Why Consistent Hashing Matters

**Using Modulo Hashing (The Problem):**

Patient medications are cached using `cache_server = hash(patient_id) % 15`.

- Patient #12345's medication history cached on Server 7
- Provider at Children's Hospital loads the patient's record → cached data served in 50ms
- At 3pm during daily system maintenance, one cache server is restarted
- N changes from 15 to 14 (then back to 15, but during the transition, hash values shift)
- `hash(patient_id) % 14` now points most patient records to different servers
- All 50+ facilities suddenly experience cache misses
- Each facility's providers query the main database simultaneously
- Main database connection pool gets hammered with 2,000+ concurrent requests
- Database response times spike from 100ms to 30+ seconds
- EHR interface becomes sluggish; providers experience 10-30 second waits to load patient records
- In critical situations (ED, ICU), slow record access can delay diagnosis and treatment

**Cascade Effect:**
- Slow EHR → providers take longer to see patient information
- Slower rounds → fewer patients seen per shift
- Staff frustration increases → higher turnover
- Care quality decreases because providers spend less time per patient
- Potential patient safety incidents if medications are missed or interactions overlooked

**Real Scenario:** A patient on warfarin (blood thinner) is admitted. The EHR is slow during a cache failure. The new doctor doesn't see the warfarin in the history (it's being re-fetched). Doctor prescribes an NSAID. Drug interaction undetected. Patient has bleeding complications.

**Using Consistent Hashing (The Solution):**

- Patient #12345's medication history on Server 7
- Cache server is restarted
- Only ~1/15th of patient records (~6.7%) remap to different servers
- 14 out of 15 providers' cache hits remain successful
- The 1/15th that remapped create cache misses, but only ~130 patients out of 2,000
- Database receives 130 queries instead of 2,000
- Database stays responsive (< 500ms per query)
- Overall system response time stays < 200ms for 93% of users
- EHR responsiveness maintained; patient care proceeds normally

---

## Scenario 3: Insurance Eligibility Verification Across a Network of Processing Nodes

### The System

You're building the real-time insurance eligibility verification system for a major pharmacy benefits manager (PBM) serving 50,000 pharmacy locations. When a patient tries to fill a prescription:

1. Pharmacy submits eligibility request with patient ID
2. System queries insurance eligibility status in real-time (coverage, copay, restrictions)
3. Pharmacy gets response within 2 seconds
4. Pharmacist can proceed with filling prescription

The system handles 10,000 requests per second during peak times (early morning, end of month). Eligibility data is cached across 20 distributed processing nodes because querying the insurance companies' systems for every request is too slow and costly.

### Why Consistent Hashing Matters

**Using Modulo Hashing (The Problem):**

Eligibility data cached using `processor_node = hash(member_id + insurance_plan_id) % 20`.

Each node caches:
- Patient's current coverage status
- Copay amounts
- Prescription restrictions and prior-auth requirements
- Plan termination dates (critical—expired coverage must be caught)

During peak hours at 9am:
- System is handling 10,000 requests/second
- Nodes are at 80% CPU capacity
- One node experiences a hardware failure
- N changes from 20 to 19
- Nearly all 10,000 requests per second now hash to different nodes
- 9,500 requests experience cache misses simultaneously
- Each miss requires querying the actual insurance company systems
- Insurance company APIs have rate limits (e.g., 1,000 queries/sec per PBM)
- Request queue explodes; 2-second SLA becomes 30+ seconds
- Pharmacy calls start coming in: "Why is the system down?"
- Pharmacists manually verify coverage (phone calls to insurance)
- Prescription filling backlog forms; patients wait hours for medications
- Pharmacy suspends filling operations until system recovers

**Worse Case Scenario - Patient Safety:**
- System returns stale data (cache miss, fallback to last-known-good status)
- System shows coverage active, but patient's plan was terminated 2 weeks ago
- Pharmacy fills $500 prescription; insurance won't cover it
- Patient gets bill they can't pay; medication becomes unaffordable
- Patient stops taking critical medication (e.g., blood pressure, diabetes, psychiatric meds)
- Patient's condition worsens; health outcomes suffer

**Compliance Impact:**
- NCPDP (National Council for the Promotion of Drug Standards) requires verification accuracy
- Failed verifications recorded; could trigger audit
- System downtime may violate SLAs with insurance partners
- Fines from CMS if coverage status errors occur

**Using Consistent Hashing (The Solution):**

- Eligibility data distributed across 20 processing nodes using consistent hashing
- Same 9am hardware failure
- Only ~1/20th (5%) of eligibility data needs to be re-fetched
- 500 requests per second experience cache miss (instead of 9,500)
- Insurance company API rate limits: 1,000 queries/sec → still OK with 500 queries/sec
- Remaining 9,500 requests/sec get cache hits within 10ms
- Overall system latency stays < 2 seconds for 95% of requests
- 5% of requests take 3-5 seconds (querying insurance company)
- System remains operational; pharmacy operations continue
- Patient coverage data remains accurate

### Additional Healthcare Complexity: Plan Changes

Real insurance eligibility systems must handle:
- **Plan terminations:** Patient loses coverage; system must reflect instantly
- **Plan changes:** Patient switches plans mid-month; effective dates matter
- **Retroactive changes:** Insurance corrections requiring data to be recalculated
- **Group transitions:** Corporate plan changes affecting thousands of employees at once

With modulo hashing, any of these large-scale updates would require rehashing a massive portion of the cache, potentially breaking the system during critical updates. Consistent hashing allows you to invalidate and refresh only the affected subset of data.

---

## Key Takeaways for Healthcare/Pharmacy Systems

1. **Patient Safety:** System instability can directly impact patient safety through delayed medication access, missed drug interactions, or coverage errors.

2. **Regulatory Compliance:** Healthcare systems are audited; system failures can trigger compliance violations and fines.

3. **Cost Impact:** Downtime costs money (staff, pharmacy closures, insurance penalties). Consistent hashing prevents expensive cascading failures.

4. **Trust:** Patients and healthcare providers trust these systems. System failures erode that trust and can damage an organization's reputation.

5. **Scale:** Healthcare organizations are massive (thousands of locations, millions of patients, tens of thousands of transactions per second). At this scale, the difference between "lose 5% of cache" and "lose 95% of cache" is the difference between operational stability and system collapse.

### When to Apply Consistent Hashing in Healthcare Systems

- **Distributed caching** of patient data, medications, or insurance information
- **Prescription routing** across multiple pharmacy locations
- **Load balancing** of clinical decision support systems across multiple servers
- **Drug interaction checking** systems serving thousands of clinicians simultaneously
- **Lab results caching** across hospital network
- **Real-time compliance checking** (controlled substance tracking, age restrictions)
- **HIPAA audit log** distribution across multiple storage nodes

Consistent hashing is not an academic exercise in healthcare—it's a foundational technique that enables reliable, safe patient care at scale.
