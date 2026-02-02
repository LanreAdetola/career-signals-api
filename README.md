# Career Signals API

A FastAPI-based academic signal analysis engine that evaluates student transcripts and provides directional academic strength signals across technical fields.

## Overview

The Career Signals API analyzes academic performance data (courses + grades) and produces **explainable academic strength signals** across categories and higher-level technical fields. The system provides directional insights without making prescriptive career recommendations.

### Key Features

- **Phase-aware analysis**: Students evaluated only on courses they could have taken
- **Explainable signals**: Every output is traceable to source data
- **Credit-weighted scoring**: Accurate category averages based on course credits
- **Field signal calculation**: Identifies strengths across Data Science, Cybersecurity, DevOps, and more
- **Coverage & confidence metrics**: Contextualize results based on data completeness
- **Stateless design**: No data persistence required

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [API Endpoints](#api-endpoints)
  - [GET /courses](#get-courses)
  - [GET /courses/{course_id}](#get-coursescourseid)
  - [POST /analysis](#post-analysis)
- [Data Models](#data-models)
- [Examples](#examples)
- [Testing](#testing)
- [Docker](#docker)

---

## Installation

### Prerequisites

- Python 3.11+
- pip

### Setup

1. Clone the repository:
```bash
git clone https://github.com/LanreAdetola/career-signals-api.git
cd career-signals-api
```

2. Create a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements.txt
```

---

## Quick Start

### Run Locally

```bash
uvicorn app.main:app --reload
```

The API will be available at `http://localhost:8000`

### Interactive API Documentation

Once running, visit:
- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

---

## API Endpoints

### GET /courses

Retrieve the course catalogue with optional filtering.

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `phase` | integer (1-3) | No | Filter courses by phase |
| `category` | string | No | Filter by category |

#### Categories

- `Data`
- `Programming`
- `Security`
- `Business`
- `Communication`
- `Business Intelligence`
- `Hands-On Experience`

#### Example Request

```bash
curl "http://localhost:8000/courses?phase=1&category=Data"
```

#### Example Response

```json
[
  {
    "id": 24,
    "course_name": "Databases",
    "category": "Data",
    "phase": 1,
    "credits": 6
  },
  {
    "id": 25,
    "course_name": "Data Processing & Analysis",
    "category": "Data",
    "phase": 1,
    "credits": 3
  }
]
```

---

### GET /courses/{course_id}

Retrieve a single course by ID.

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `course_id` | integer | Yes | Unique course identifier |

#### Example Request

```bash
curl "http://localhost:8000/courses/24"
```

#### Example Response

```json
{
  "id": 24,
  "course_name": "Databases",
  "category": "Data",
  "phase": 1,
  "credits": 6
}
```

#### Error Response (404)

```json
{
  "detail": "Course not found"
}
```

---

### POST /analysis

Analyze academic performance and generate field signals.

#### Request Body

```json
{
  "current_phase": 2,
  "grades": [
    {
      "course_name": "Databases",
      "grade": 15.5
    },
    {
      "course_name": "Programming Fundamentals",
      "grade": 14.0
    }
  ]
}
```

#### Request Schema

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `current_phase` | integer | Yes | 1-3 | Student's current phase |
| `grades` | array | Yes | - | List of course grades |
| `grades[].course_name` | string | Yes | Must exist in catalogue | Course name |
| `grades[].grade` | float | Yes | 0-20 | Grade value |

#### Response Schema

```json
{
  "phase": 2,
  "coverage": 0.56,
  "confidence": "Medium",
  "category_scores": [
    {
      "category": "Data",
      "average_grade": 15.5,
      "total_credits": 9
    }
  ],
  "field_signals": [
    {
      "field": "Data Science",
      "score": 14.83,
      "signal_strength": "Strong",
      "contributors": {
        "categories": ["Data", "Programming"],
        "courses": ["Databases", "Programming Fundamentals"]
      },
      "evidence_level": "Partial"
    }
  ],
  "warnings": null
}
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `phase` | integer | Current phase analyzed |
| `coverage` | float (0-1) | Proportion of eligible curriculum completed |
| `confidence` | string | "High" (≥0.7), "Medium" (≥0.4), or "Low" (<0.4) |
| `category_scores` | array | Credit-weighted averages per category |
| `field_signals` | array | Calculated field strengths with explanations |
| `warnings` | array or null | Non-fatal issues detected |

#### Signal Strength Labels

| Score | Label |
|-------|-------|
| ≥ 14 | Strong |
| 11-13.9 | Consistent |
| < 11 | Emerging |

#### Evidence Levels

- **Complete**: All required categories for the field are present
- **Partial**: One or more required categories are missing

---

## Examples

### Example 1: Full Phase 1 Analysis

**Request:**

```bash
curl -X POST "http://localhost:8000/analysis" \
  -H "Content-Type: application/json" \
  -d '{
    "current_phase": 1,
    "grades": [
      {"course_name": "Databases", "grade": 16.0},
      {"course_name": "Programming Fundamentals", "grade": 15.5},
      {"course_name": "Computing Fundamentals", "grade": 14.0},
      {"course_name": "Business Fundamentals", "grade": 13.5},
      {"course_name": "Data Processing & Analysis", "grade": 17.0}
    ]
  }'
```

**Response:**

```json
{
  "phase": 1,
  "coverage": 0.47,
  "confidence": "Medium",
  "category_scores": [
    {
      "category": "Data",
      "average_grade": 16.33,
      "total_credits": 9
    },
    {
      "category": "Programming",
      "average_grade": 15.5,
      "total_credits": 6
    },
    {
      "category": "Security",
      "average_grade": 14.0,
      "total_credits": 6
    },
    {
      "category": "Business",
      "average_grade": 13.5,
      "total_credits": 3
    }
  ],
  "field_signals": [
    {
      "field": "Data Science",
      "score": 15.85,
      "signal_strength": "Strong",
      "contributors": {
        "categories": ["Data", "Programming", "Business"],
        "courses": ["Databases", "Data Processing & Analysis", "Programming Fundamentals", "Business Fundamentals"]
      },
      "evidence_level": "Complete"
    },
    {
      "field": "Data Analytics",
      "score": 15.63,
      "signal_strength": "Strong",
      "contributors": {
        "categories": ["Data", "Business"],
        "courses": ["Databases", "Data Processing & Analysis", "Business Fundamentals"]
      },
      "evidence_level": "Complete"
    }
  ],
  "warnings": null
}
```

### Example 2: Low Coverage Warning

**Request:**

```bash
curl -X POST "http://localhost:8000/analysis" \
  -H "Content-Type: application/json" \
  -d '{
    "current_phase": 1,
    "grades": [
      {"course_name": "Databases", "grade": 15.0}
    ]
  }'
```

**Response:**

```json
{
  "phase": 1,
  "coverage": 0.09,
  "confidence": "Low",
  "category_scores": [
    {
      "category": "Data",
      "average_grade": 15.0,
      "total_credits": 6
    }
  ],
  "field_signals": [
    {
      "field": "Data Analytics",
      "score": 15.0,
      "signal_strength": "Strong",
      "contributors": {
        "categories": ["Data"],
        "courses": ["Databases"]
      },
      "evidence_level": "Partial"
    }
  ],
  "warnings": [
    "Very low coverage (9%): Results may not be representative"
  ]
}
```

### Example 3: Validation Error

**Request:**

```bash
curl -X POST "http://localhost:8000/analysis" \
  -H "Content-Type: application/json" \
  -d '{
    "current_phase": 1,
    "grades": [
      {"course_name": "Unknown Course", "grade": 15.0}
    ]
  }'
```

**Response (422):**

```json
{
  "detail": [
    {
      "type": "value_error",
      "loc": ["body", "grades"],
      "msg": "Value error, Unknown course name: 'Unknown Course'. Must be one of the courses in the catalogue.",
      "input": [{"course_name": "Unknown Course", "grade": 15.0}]
    }
  ]
}
```

---

## Data Models

### Supported Fields

The system calculates signals for the following technical fields:

1. **Data Science**
   - Data (60%), Programming (30%), Business (10%)

2. **Data Analytics**
   - Data (70%), Business (30%)

3. **Cybersecurity**
   - Security (70%), Programming (20%), Business (10%)

4. **DevSecOps**
   - Programming (40%), Security (40%), Data (20%)

5. **Business Intelligence**
   - Data (50%), Business (50%)

### Phase System

- **Phase 1**: Fundamentals (first year courses)
- **Phase 2**: Phase 1 + Advanced (second year courses)
- **Phase 3**: Phase 1 + Phase 2 + Specialization (third year courses)

Students in Phase 2 are evaluated on Phase 1 and Phase 2 courses. Students in Phase 3 are evaluated on all courses.

---

## Testing

### Run Tests

```bash
python -m pytest tests/test_analysis_service.py -v
```

### Test Coverage

The test suite includes 26 tests covering:

- Signal strength categorization
- Confidence calculation
- Phase filtering
- Coverage calculation
- Credit-weighted averaging
- Field signal calculation
- Evidence level determination
- Sorting behavior
- Warning generation
- Edge cases

All tests pass ✅

---

## Docker

### Build Image

```bash
docker build -t career-signals-api .
```

### Run Container

```bash
docker run -p 8000:8000 career-signals-api
```

The API will be available at `http://localhost:8000`

---

## Design Principles

### What This System Does

✅ Analyzes academic performance data  
✅ Provides directional academic signals  
✅ Explains all calculations  
✅ Considers only eligible courses  
✅ Weights by course credits  

### What This System Does NOT Do

❌ Make career recommendations  
❌ Assess employability  
❌ Rank students against each other  
❌ Predict salaries or job outcomes  
❌ Store personal data  

---

## Project Structure

```
career-signals-api/
├── app/
│   ├── main.py                 # FastAPI application
│   ├── api/
│   │   ├── courses.py          # Course endpoints
│   │   └── analysis.py         # Analysis endpoint
│   ├── models/
│   │   ├── course.py           # Course data models
│   │   └── analysis.py         # Analysis data models
│   ├── services/
│   │   ├── course_service.py   # Course business logic
│   │   └── analysis_service.py # Analysis business logic
│   └── data/
│       ├── courses.py          # Course catalogue
│       └── fields.py           # Field definitions
├── tests/
│   └── test_analysis_service.py # Unit tests
├── requirements.txt            # Python dependencies
├── Dockerfile                  # Container configuration
└── README.md                   # This file
```

---

## Contributing

This is an MVP implementation. The following are **out of scope** for V1:

- PDF transcript parsing
- User authentication
- Data persistence
- Frontend UI
- ML-based predictions

Future extensions may consider these features.

---

## License

[Add your license information here]

---

## Support

For questions or issues, please open an issue on the [GitHub repository](https://github.com/LanreAdetola/career-signals-api).

---

**Version**: 1.0.0  
**Last Updated**: 2024
