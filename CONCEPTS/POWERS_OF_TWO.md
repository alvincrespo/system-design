# Powers of Two for System Design

A practical guide for back-of-the-envelope calculations, with examples from the healthcare/pharmacy domain.

---

## The Essential Reference Table

| Power | Exact Value | Approx Value | Bytes |
|-------|-------------|--------------|-------|
| 7 | 128 | | |
| 8 | 256 | | |
| 10 | 1,024 | 1 thousand | 1 KB |
| 16 | 65,536 | | 64 KB |
| 20 | 1,048,576 | 1 million | 1 MB |
| 30 | 1,073,741,824 | 1 billion | 1 GB |
| 32 | 4,294,967,296 | | 4 GB |
| 40 | 1,099,511,627,776 | 1 trillion | 1 TB |
| 50 | 1,125,899,906,842,624 | 1 quadrillion | 1 PB |

---

## Why This Matters

The powers-of-two table serves as a mental calculator for quickly estimating storage, memory, and bandwidth requirements. These estimates help engineers reason about scale, compare architectural approaches, and identify potential bottlenecks before committing to a design.

**The core insight**: You can chain approximations. If you know 2^10 ≈ 1 thousand and 2^20 ≈ 1 million, then you can quickly estimate that 2^30 ≈ 1 billion (just add the exponents).

---

## The Mental Math Technique

When working through calculations, chain these relationships:

- **Thousands → KB** (2^10)
- **Millions → MB** (2^20)
- **Billions → GB** (2^30)
- **Trillions → TB** (2^40)
- **Quadrillions → PB** (2^50)

### Key Rules

1. **Approximate real numbers with powers of two**
   - 1 billion ≈ 2^30
   - 256 = 2^8 (exactly)

2. **When multiplying powers of two, add the exponents**
   - 2^30 × 2^8 = 2^(30+8) = 2^38

3. **Convert small multipliers to powers of two**
   - 4 = 2^2
   - So 4 × 2^30 = 2^2 × 2^30 = 2^32

4. **Look up the final exponent in your table**
   - 2^40 → 1 TB

---

## Worked Example: Prescription Storage Calculation

**Problem:** How much storage is needed for 5 years of prescription data in a national pharmacy system?

### Step 1: Estimate prescription volume

- US has ~330 million people
- Average person fills ~12 prescriptions/year
- Total: 330M × 12 ≈ 4 billion prescriptions/year

### Step 2: Estimate size per prescription record

| Field | Size |
|-------|------|
| Patient ID | 8 bytes |
| Drug ID | 8 bytes |
| Prescriber ID | 8 bytes |
| Pharmacy ID | 8 bytes |
| Dosage info | 32 bytes |
| Timestamps | 16 bytes |
| Status/metadata | 48 bytes |
| **Total** | **≈ 128 bytes (round to 256 for safety)** |

### Step 3: Calculate total storage

```
4 billion prescriptions/year × 256 bytes per prescription = Total storage
```

Breaking down the math:

```
4      ×    2^30    ×    2^8     =    2^32    ×    2^8    =    2^40    =    1 TB
│            │           │              │           │           │           │
│            │           │              │           │           │           └── From table: 2^40 = 1 TB
│            │           │              │           │           │
│            │           │              │           │           └── Exponent addition: 32 + 8 = 40
│            │           │              │           │
│            │           │              │           └── Carrying over the 2^8
│            │           │              │
│            │           │              └── Because 4 = 2^2, and 2^2 × 2^30 = 2^32
│            │           │
│            │           └── 256 bytes per prescription (2^8 = 256)
│            │
│            └── 1 billion (2^30 ≈ 1 billion)
│
└── The "4" in "4 billion prescriptions"
```

### Final Answer

- 1 TB per year
- 5 years = 5 TB (before replication/indexing)
- With 3x replication + indexes: ~20-30 TB

---

## Understanding QPS (Queries Per Second)

**QPS = Queries Per Second** — a measure of throughput indicating how many requests a system handles each second.

### Related Terms

| Term | Meaning | Example |
|------|---------|---------|
| **QPS** | Queries per second | "Our read QPS is 10,000" |
| **RPS** | Requests per second | Often used interchangeably with QPS |
| **TPS** | Transactions per second | Usually implies writes or atomic operations |
| **Throughput** | General term for volume over time | "What's the throughput of this pipeline?" |

### Why QPS Matters

QPS helps determine:

- Whether a single server can handle the load, or if multiple servers are needed
- Whether caching is necessary
- What kind of database can support the expected load

**For context:** A single modern server can typically handle around 10,000-50,000 simple QPS, while complex queries with multiple joins might only achieve 100-1,000 QPS on the same hardware.

---

## Worked Example: QPS Estimation for Drug Interaction Checks

**Problem:** Every prescription needs a drug interaction check. What's the expected QPS?

### Step 1: Daily prescriptions

```
4 billion/year ÷ 365 days ≈ 11 million/day
```

### Step 2: Distribution matters

Pharmacies aren't open 24 hours—they're typically busy for about 12 hours/day.

```
11 million prescriptions/day
──────────────────────────── ≈ 250 QPS average
   12 hours × 3600 seconds
```

The full unit conversion:

```
4 billion prescriptions     1 year        1 day
─────────────────────── × ────────── × ──────────── = QPS
        year               365 days    3600 seconds
```

### Step 3: Peak load

**Rule of thumb:** Peak is typically 3-5x average.

- Peak: ~1000 QPS

### Step 4: Real-world considerations

- 1000 QPS is modest—a single well-optimized service can handle this
- But each check might query multiple drugs, so actual DB reads could be 5-10x
- Real load: 5,000-10,000 reads/second

---

## Time Conversion Shortcuts

These conversions come up frequently when converting between "per day" or "per year" metrics and QPS.

| Time Unit | Seconds | Quick Approximation |
|-----------|---------|---------------------|
| 1 minute | 60 | |
| 1 hour | 3,600 | |
| 1 day | 86,400 | ~100K for quick math |
| 1 year | ~31.5 million | ~2^25 |

---

## Quick Reference: Healthcare Data Sizes

These estimates are useful for capacity planning in healthcare systems.

| Data Type | Typical Size | Notes |
|-----------|--------------|-------|
| Prescription record | ~256 bytes | Structured data only |
| Patient demographics | ~1 KB | Basic registration info |
| Lab result (structured) | ~500 bytes | Single test result |
| Clinical note | ~5-10 KB | Free-text documentation |
| Pharmacy inventory item | ~32 bytes | Per drug per location |

### Medical Imaging Sizes

Medical imaging represents a significant storage challenge. According to industry estimates, a single patient generates close to 80 megabytes each year in imaging and EMR data combined.

| Modality | Size per Image/Study | Notes |
|----------|---------------------|-------|
| Digital X-ray | 5-20 MB | Single image, uncompressed |
| CT scan (single slice) | ~0.5 MB | 512×512 resolution typical |
| CT scan (full study) | 20-200 MB | Hundreds of slices |
| MRI (single slice) | ~0.5-2 MB | Varies by sequence |
| MRI (full study) | 50-500 MB | Multiple sequences |
| Mammography | 30-50 MB per image | High resolution required |
| Ultrasound | 1-10 MB | Video clips can be larger |

DICOM format adds header metadata to each image, typically adding a few KB per file.

---

## Practice Problem

**"50 million users × 1 KB profile each"**

```
50 million × 1 KB
= 50 × (1 million) × 1 KB
= 50 × 2^20 × 2^10        ← 1 million ≈ 2^20, 1 KB = 2^10
≈ 2^6 × 2^20 × 2^10       ← 50 ≈ 64 = 2^6 (round up for safety)
= 2^36
= 2^6 × 2^30              ← Factor it to use your table
= 64 GB                   ← 2^30 = 1 GB, so 2^36 = 64 GB
```

---

## Additional Scenarios

### Patient Medical Records (EHR) with Imaging

Text-based records are manageable:
- ~50 KB per patient per year
- 330M patients × 50 KB = 16.5 TB/year

Imaging changes everything:
- If 10% of patients get one imaging study/year: 33M × 100 MB = 3.3 PB/year

This illustrates why healthcare systems often use tiered storage strategies with separate object storage for imaging data.

### Real-Time Pharmacy Inventory (Memory Estimation)

```
10,000 pharmacy locations × 5,000 drugs × 32 bytes each
= 50 million × 32
= 1.6 billion bytes ≈ 1.6 GB
```

This fits easily in RAM on a single machine, making it feasible to cache the entire inventory dataset.

---

## Common Considerations in Healthcare Systems

1. **Audit logging requirements (HIPAA)**
   - Every access must be logged
   - Audit logs can grow larger than the primary data

2. **Offline operation and synchronization**
   - Pharmacies may need to operate during network outages
   - Estimate queue depth: prescriptions/hour × max offline duration

3. **Search and indexing**
   - Full-text search indexes can be 50-100% of raw data size
   - Factor this into storage estimates

---

## References and Further Reading

### Back-of-the-Envelope Calculations

- **[Napkin Math](https://github.com/sirupsen/napkin-math)** — A maintained, tested, and referenced collection of numbers for system design by Simon Eskildsen. Includes throughput numbers and cost estimates alongside latency figures.

- **[Latency Numbers Every Programmer Should Know](https://colin-scott.github.io/personal_website/research/interactive_latency.html)** — Interactive visualization by Colin Scott showing how latency numbers have changed over time, based on Jeff Dean's original presentation.

- **[System Design: Back of the Envelope](https://systemdesign.one/back-of-the-envelope/)** — Comprehensive guide covering powers of two, data types, availability calculations, and practical examples.

- **[ByteByteGo: Back-of-the-Envelope Estimation](https://bytebytego.com/courses/system-design-interview/back-of-the-envelope-estimation)** — Course material from Alex Xu's System Design Interview resources.

- **[GeeksforGeeks: Back of the Envelope Estimation](https://www.geeksforgeeks.org/back-of-the-envelope-estimation-in-system-design/)** — Overview of estimation techniques including order of magnitude estimation and dimensional analysis.

### Healthcare Data and Standards

- **[NCBI: Obtaining Data From Electronic Health Records](https://www.ncbi.nlm.nih.gov/books/NBK551878/)** — Detailed guide on EHR data types, coding standards (ICD, CPT, LOINC, SNOMED), and data extraction considerations.

- **[Wikipedia: DICOM](https://en.wikipedia.org/wiki/DICOM)** — Overview of the Digital Imaging and Communications in Medicine standard used for medical image storage and transmission.

- **[PMC: Managing DICOM Images](https://pmc.ncbi.nlm.nih.gov/articles/PMC3354356/)** — Practical tips on DICOM file structure, compression, and storage considerations.

- **[HarmonyHIT: Health Data Volumes](https://www.harmonyhit.com/health-data-volumes-skyrocket-legacy-data-archives-rise-hie/)** — Industry estimates on healthcare data growth, including the ~80 MB per patient per year figure for imaging and EMR data combined.

### Latency and Performance

- **[Jeff Dean's Original Numbers](https://gist.github.com/jboner/2841832)** — The widely-referenced GitHub gist containing the original latency numbers from Jeff Dean's 2010 presentation.

- **[High Scalability: More Numbers Every Programmer Must Know](https://highscalability.com/more-numbers-every-awesome-programmer-must-know/)** — Context and interpretation of latency numbers with practical applications.
