# College Timetable Generator

[![Timetable Generator CI](https://github.com/harshithabt/college-timetable-generator/actions/workflows/ci.yml/badge.svg)](https://github.com/harshithabt/college-timetable-generator/actions/workflows/ci.yml)
![Python](https://img.shields.io/badge/python-3.12-blue)
![Flask](https://img.shields.io/badge/flask-3.1-lightgrey)

A web application that generates clash-free college timetables. Enter your working days, slots, classrooms and subject offerings (by hand or via CSV), and a backtracking scheduler produces a weekly grid for every class section, with no teacher, class or room double-booked.

Built as a Software Engineering course project at PES University.

---

## Project Team

| Name | SRN |
|---|---|
| Diya B.R. | PES1UG24AM097 |
| Gaurav S. | PES1UG24AM104 |
| Harsha K.R. | PES1UG24AM114 |
| Harshitha B.T. | PES1UG24AM118 |

---

## Table of Contents

- [Features](#features)
- [How It Works](#how-it-works)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Usage](#usage)
- [CSV Format](#csv-format)
- [Sample Data](#sample-data)
- [Testing](#testing)
- [Continuous Integration](#continuous-integration)
- [Deployment](#deployment)
- [Project Documents](#project-documents)
- [Project Team](#project-team)

---

## Features

- **Constraint-based scheduling:** guarantees no teacher, class section or classroom is used twice in the same day and slot.
- **Flexible input:** add subjects through the on-page table or import them from a CSV file (a template is available to download).
- **Teacher availability:** optionally restrict teachers to specific `(day, slot)` pairs; teachers not listed are available all week.
- **Fixed classrooms per section:** assign rooms automatically in order, or manually with one `section, classroom` pair per line.
- **Colour-coded subjects:** automatic per-course colours, or choose your own. A course keeps the same colour across all its sections.
- **Per-section weekly grid:** pick a class section and view its full week.
- **Edit and regenerate:** return to the form with all settings preserved, tweak them, and generate again.
- **Clear validation errors:** problems such as a bad CSV row, unknown teacher, or impossible workload are reported with a specific message and the offending field is highlighted.
- **Light and dark themes.**

## How It Works

1. The Flask layer (`main.py`) validates the form or CSV and normalises it into a list of subject offerings.
2. Each offering is expanded into one task per weekly period.
3. The scheduler (`app/scheduler.py`) orders tasks so that teachers with fewer allowed slots are placed first, then uses **backtracking** to place each task in a `(day, slot, room)` that does not conflict with anything already scheduled.
4. Conflict detection lives in `app/constraints.py` (`has_conflict`, `validate_timetable`), and the finished timetable is re-validated before it is returned.
5. Because each section has one fixed classroom, rooms are applied to the sessions afterwards and the result is validated once more.

If no complete timetable exists (for example, a teacher is asked to teach more periods than there are slots, or there is not enough room-slot capacity), the app returns a descriptive error instead of a partial result.

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Python, Flask |
| Frontend | HTML, CSS, vanilla JavaScript (Jinja2 templates) |
| Testing | pytest |
| CI | GitHub Actions |
| Deployment | Render (gunicorn) |

## Project Structure

```
college-timetable-generator/
├── .github/workflows/
│   ├── ci.yml               # CI pipeline
├── app/
│   ├── __init__.py
│   ├── scheduler.py         # Backtracking timetable generator
│   └── constraints.py       # Clash detection and timetable validation
├── jira_sprint/
|   ├── sprint1 burndown chart.png
│   └── sprint1 burnup chart.png
│   └── sprint2 burndown chart.png
│   └── sprint2 burnup chart.png
├── static/
│   ├── app.js               # Dynamic rows, theme toggle, form behaviour
│   └── style.css 
├── templates/
│   ├── index.html           # Configuration form
│   └── results.html         # Weekly timetable view
├── test_data/               # Ready-made valid, impossible and invalid CSVs
├── tests/
│   ├── test_scheduler.py    # Scheduler and constraint tests
│   └── test_flask_main.py   # Route, form, CSV and UI tests
├── .gitignore
├── Procfile                 # Process definition (gunicorn)
|── README.md
├── main.py                  # Flask app: routes, form/CSV parsing, validation
├── render.yaml              # Render service configuration
├── requirements.txt

```

## Getting Started

### Prerequisites

- Python 3.9 or newer (CI runs on 3.12)
- pip

### Installation

```bash
git clone https://github.com/harshithabt/college-timetable-generator.git
cd college-timetable-generator

# Recommended: use a virtual environment
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install -r requirements.txt
```

### Run locally

```bash
python main.py
```

Then open <http://127.0.0.1:5000> in your browser.

> The Flask app lives in `main.py`, so use `python main.py` (or `flask --app main run`). There is no `app.py`; `app/` is the package holding the scheduler.

## Usage

1. **Set the week.** Enter working days in timetable order (comma-separated) and the number of slots per day.
2. **Add subjects.** Fill in course code, subject name, teacher, class section and periods per week, or import a CSV.
3. **Choose classrooms.** List the available rooms, then pick automatic or manual fixed assignment for each section.
4. **Pick colours** automatically or manually (manual colours apply to the table form only; CSV imports use automatic colours).
5. *(Optional)* **Restrict teacher availability** with one `teacher, day, slot` line per allowed slot. Slot numbers start at 1.
6. Click **Generate timetable**, then choose a class section to view its weekly grid. Use **Edit settings** or **Regenerate** to iterate.

Default values: Monday to Friday, 6 slots per day, rooms `R101, R102, R103, R104`.

> Use either a CSV upload **or** the manual subject rows in a single run, not both.

## CSV Format

Headers must match exactly, in this order:

```csv
course_code,subject_name,teacher,class_id,periods_per_week
CS201,Data Structures,Dr. Ramesh Kumar,CSE-A,4
CS202,Database Systems,Dr. Anita Rao,CSE-A,4
```

- The file must be UTF-8 and contain at least one data row.
- Every field is required, and `periods_per_week` must be a positive whole number.
- A **Download CSV template** link is available on the form (`/subjects-template.csv`).

## Sample Data

The `test_data/` folder contains CSVs for trying the app and for demonstrating validation:

| File | Purpose |
|---|---|
| `01_normal_timetable.csv` | Typical, solvable input |
| `02_teacher_availability.csv` | Use with availability restrictions |
| `03_limited_classrooms.csv` | Tight classroom supply |
| `04a`–`04c_impossible_*.csv` | Teacher overload, class overload, insufficient room capacity (should produce an error) |
| `05a`–`05h_invalid_*.csv` | Empty file, header only, missing column, blank values, bad numbers, duplicates, wrong headers |

## Testing

```bash
python -m pytest -v
```

The suite has 70 tests covering the scheduler and clash constraints, route behaviour, CSV import, classroom assignment, colour handling, edit/regenerate flow and error reporting.

## Continuous Integration

GitHub Actions (`.github/workflows/ci.yml`) installs dependencies and runs the full pytest suite on Python 3.12 for every push, for pull requests into `main`, and on manual dispatch.

## Deployment

The app is deployed on [College Timetable Generator](https://college-timetable-generator-qkry.onrender.com/). The service is defined in `render.yaml` (build: `pip install -r requirements.txt`, health check on `/`) and served with gunicorn using the entry point `main:app`, as in the `Procfile`:

```
web: gunicorn main:app
```

## Project Documents

Course deliverables are included in the repository:

- [Software Requirements Specification (SRS)](SRS_College_Timetable_Generator.pdf)
- [Software Architecture Document (SAD)](SAD_College_Timetable_Generator.pdf)
- [Software Test Plan (STP)](STP_College_Timetable_Generator.pdf)



[College Timetable Generator on GitHub](https://github.com/harshithabt/college-timetable-generator)
