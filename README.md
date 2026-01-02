# Interactive Todo Application (v2.1.0)

A high-performance terminal-based todo application built with Python and Neon Serverless PostgreSQL, featuring an advanced asynchronous architecture, intelligent recurrence, and a rich command-line interface.

## Overview

This project implements an **Advanced Level** task management system using a modernized tech stack. It transitions from a simple menu-driven interface to a powerful command-driven CLI with interactive wizards, enforcing a strict 4-layer Repository Pattern.

- **CLI Layer** (`src/cli.py`, `src/main.py`): Command-driven interface using Typer, Rich, and Questionary.
- **Service Layer** (`src/services.py`, `src/models.py`, `src/recurrence_engine.py`, `src/notification_manager.py`): Business logic, recurrence arithmetic, and system notifications.
- **Repository Layer** (`src/repository.py`): Clean data access abstraction with async execution.
- **Database Layer** (`src/database.py`): Async Neon PostgreSQL connection management with pooling and idempotent migrations.

## Features

- **Intelligent Recurring Tasks**: Automatically regenerate tasks on completion (daily, weekly, monthly) while preserving history.
- **Smart Date Input**: Natural language date parsing (e.g., "tomorrow", "next Monday at 3pm").
- **Interactive wizards**: Guided multi-step forms via `questionary` for advanced users.
- **Desktop Notifications**: Proactive system-level alerts for tasks due within 30 minutes.
- **Full-Text Search**: Search across task titles and descriptions.
- **Rich UI**: Beautifully formatted tables, progress spinners, and colored terminal output.
- **Async & Cloud-Native**: Non-blocking database operations with native Neon Serverless PostgreSQL support.

## Prerequisites

- **Python 3.12 or higher**
- **Neon PostgreSQL account** - [Sign up at neon.tech](https://neon.tech)
- **Environment variables** (configured via `.env` file)

## Installation

### 1. Clone & Setup
```bash
git clone <repository-url>
cd todo-cli
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
```

### 2. Install Dependencies
```bash
pip install -r requirements.txt
```

### 3. Configure Database
Create a `.env` file in the root directory:
```env
DATABASE_URL=postgresql://user:password@ep-cool-name-123456.us-east-2.aws.neon.tech/neondb?sslmode=require
```

## Usage

Start the interactive session:
```bash
python src/main.py
```

### Interactive Commands

| Action | Description |
|--------|-------------|
| **Add** | Interactive wizard with NLP date support and recurrence options. |
| **List** | Filter by status, priority, or category. Sort results dynamically. |
| **Search** | Full-text keyword matching. |
| **Update** | Modify existing tasks using the interactive editor. |
| **History** | View the full lineage of a recurring task series. |
| **Complete** | Mark a task as done (triggering recurrence if enabled). |

## Architecture

The system follows a strictly separated 4-layer architecture:

1. **UC/UX Layer**: `src/main.py` handles the primary interactive loop and boot-time checks (notifications). `src/cli.py` contains command definitions and Typer integration.
2. **Service Layer**: Contains the core business logic. `src/recurrence_engine.py` handles date arithmetic, while `src/notification_manager.py` orchestrates OS alerts.
3. **Repository Layer**: Isolated SQL logic in `src/repository.py` using parameterized queries.
4. **Database Layer**: `src/database.py` manages the async connection pool and idempotent schema migrations.

## Technology Stack

| Component | Technology |
|-----------|------------|
| Language | Python 3.12+ |
| Database | Neon Serverless PostgreSQL |
| CLI Frameworks | Typer, Questionary |
| NLP Date Parsing | dateparser |
| System Alerts | plyer |
| UI Rendering | Rich |
| Async Driver | Psycopg 3 (Async) |

---
**Status**: Production Ready (Feature 002-recurring-tasks complete)
**Version**: 2.1.0
**Architecture**: 4-Layer Async Repository Pattern
