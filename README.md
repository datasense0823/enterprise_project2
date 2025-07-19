# DataSense Gen AI Launchpad – Enterprise Project 2  
## Intelligent Change Detection & Sync System

## Overview

This project aims to build a reliable, agent-based **Change Detection and Synchronization System**. It continuously monitors updates in a source database table, validates them, and syncs relevant changes to a destination table. It supports **undo functionality**, **audit logging**, **event triggers**, and is designed to be extensible across systems.

---

## Objectives

- Monitor a source table in real time or at intervals for data changes (insert, update, delete).
- Automatically detect changes and validate them using intelligent agent logic.
- Synchronize accepted changes to a target table, with metadata tracking.
- Enable rollback (undo) of previously synced changes, safely and traceably.
- Log all events, including change detection, approval, sync, and rollback.
- Provide FastAPI endpoints and Slack/email-based alerts for observability.

---

## Key Features

### Real-Time or Scheduled Change Detection
- Poll or listen for row-level changes (using triggers or periodic scanning).
- Detect changes using hash comparison, row versioning, or `updated_at` timestamps.

### Agent-Based Change Validation
- Agent validates each change using business logic or rules (e.g., skip bot-generated rows).
- Supports semantic reasoning or LLM-assisted validation for free-text fields.

### Change Sync Engine
- Applies validated changes to target tables with optional transformation logic.
- Tracks original and updated values with row-level granularity.
- Only syncs delta changes, not full records.

### Undo / Rollback Support
- Enables undo of one or more synced changes.
- Automatically reverts values in target table.
- Optionally notifies relevant stakeholders via Slack or email.

### Change Logs & Audit Trails
- Stores all events: detected change, agent decision, sync timestamp, undo actions.
- All entries have metadata: user, thread ID, row ID, source→target values.

### Webhooks / Triggers
- Optionally notifies external systems on sync or undo events (via FastAPI or Slack).

---



---

## Database Tables

### `change_events`
| Field        | Type        |
|--------------|-------------|
| id           | UUID        |
| thread_id    | UUID        |
| row_key      | TEXT        |
| table_name   | TEXT        |
| change_type  | ENUM        |
| old_data     | JSONB       |
| new_data     | JSONB       |
| detected_at  | TIMESTAMP   |
| validated    | BOOLEAN     |
| synced       | BOOLEAN     |
| undo_token   | UUID        |

### `sync_log`
| Field         | Type       |
|---------------|------------|
| id            | UUID       |
| source_table  | TEXT       |
| target_table  | TEXT       |
| source_id     | TEXT       |
| diff          | JSONB      |
| synced_at     | TIMESTAMP  |
| performed_by  | TEXT       |
| undo_status   | TEXT       |

### `undo_requests`
| Field         | Type       |
|---------------|------------|
| id            | UUID       |
| change_event  | UUID       |
| requestor     | TEXT       |
| reason        | TEXT       |
| undone_at     | TIMESTAMP  |
| status        | ENUM       |

---

## API Endpoints (FastAPI)

| Endpoint                    | Method | Description |
|-----------------------------|--------|-------------|
| `/change-events/`           | GET    | View all change logs |
| `/sync-manual/`             | POST   | Manually sync a pending change |
| `/undo-request/`            | POST   | Request undo for a synced change |
| `/status/{change_id}`       | GET    | Get sync/undo status of a change |
| `/diff/{row_key}`           | GET    | View old vs new value side-by-side |

---

## Requirements

- Python 3.11+
- FastAPI
- PostgreSQL
- SQLAlchemy / psycopg2
- Alembic (for migrations)
- Slack SDK
- Pydantic / Uvicorn
- Optional: OpenAI for text field validation

---

## Deployment Plan

### Phase 1 – Local Agent Flow
- Run change detector on mock DB
- Validate changes and sync to local target table

### Phase 2 – FastAPI + Undo Flow
- Expose FastAPI endpoints for logs and undo
- Add Docker + Postgres setup

### Phase 3 – Cloud-Ready Deployment
- Deploy on AWS/GCP/Azure
- Add monitoring, cron support, Slack alerts
- Use JWT for API protection

---

## Future Enhancements

- Integrate LLM-based validation for semantic change approval
- Add alerting on anomalous or bulk changes
- Provide time-window rollbacks (e.g., “undo all changes in last 2 hours”)
- Add metrics dashboard (Grafana/Prometheus)
- Add RBAC for who can sync or undo
- Enable syncing across microservices via webhook triggers

---

## Example Scenario

A user updates their address in the CRM system:
- Change detector notices the `address` field updated
- Validates the new address format (non-empty, real zip code)
- Sync agent applies the change to the billing system
- Event is logged in `sync_log`
- Two days later, user requests rollback
- Admin triggers undo from FastAPI
- Agent restores the old value and logs it

---

## Authors

DataSense Team  



