**MINIONS CALENDAR — IMPLEMENTATION SPEC**

You are tasked with creating the complete initial foundation for `minions-calendar` — a structured calendar and scheduling system that enables time management, event tracking, recurring events, and availability detection. This is part of the Minions ecosystem, a universal structured object system designed for building AI-native tools.

---

**PROJECT OVERVIEW**

`minions-calendar` provides structured time management with recurrence rules, conflict detection, availability tracking, and calendar integration. It allows developers and agents to treat events as first-class objects with full recurrence support, timezone handling, iCal interoperability, and natural language parsing.

The core concept: agents need to manage time-based workflows, schedule themselves, avoid conflicts, and coordinate with external calendar systems. This is time-awareness infrastructure for AI agents.

---

**CONCEPT OVERVIEW**

This project is built on the Minions SDK (`minions-sdk`), which provides the foundational primitives: Minion (structured object instance), Minion Type (schema), and Relation (typed link between minions).

Events can recur using RRULE-compatible specifications. The system detects conflicts across overlapping time slots, computes availability windows, and integrates with external calendar services via `integration_link` relations. Natural language parsing enables human-friendly event creation.

The system supports both TypeScript and Python SDKs with cross-language interoperability (both serialize to the same JSON format). All documentation includes dual-language code examples with tabbed interfaces.

---

**CORE PRIMITIVES**

This project defines the following Minion Types:

- `event` — A scheduled event with start time, end time, title, description, location, and attendees
- `recurring-event` — An event with recurrence rules (RRULE format) for daily, weekly, monthly, or custom patterns
- `reminder` — A notification trigger linked to an event via `references` relation
- `availability` — A time window indicating free/busy status for scheduling purposes
- `booking` — A scheduled appointment with confirmation status and booking metadata
- `timezone` — Timezone configuration for event display and scheduling

---

**MINIONS SDK REFERENCE — REQUIRED DEPENDENCY**

This project depends on `minions-sdk`, a published package that provides the foundational primitives. The GH Agent building this project MUST install it from the public registries and use the APIs documented below — do NOT reimplement minions primitives from scratch.

**Installation:**
```bash
# TypeScript (npm)
npm install minions-sdk
# or: pnpm add minions-sdk

# Python (PyPI) — package name is minions-sdk, but you import as "minions"
pip install minions-sdk
```

**TypeScript SDK — Core Imports:**
```typescript
import {
  // Core types
  type Minion, type MinionType, type Relation,
  type FieldDefinition, type FieldValidation, type FieldType,
  type CreateMinionInput, type UpdateMinionInput, type CreateRelationInput,
  type MinionStatus, type MinionPriority, type RelationType,
  type ExecutionResult, type Executable,
  type ValidationError, type ValidationResult,

  // Validation
  validateField, validateFields,

  // Built-in Schemas (10 MinionType instances — reuse where applicable)
  noteType, linkType, fileType, contactType,
  agentType, teamType, thoughtType, promptTemplateType, testCaseType, taskType,
  builtinTypes,

  // Registry — stores and retrieves MinionTypes by id or slug
  TypeRegistry,

  // Relations — in-memory directed graph with traversal utilities
  RelationGraph,

  // Lifecycle — CRUD operations with validation
  createMinion, updateMinion, softDelete, hardDelete, restoreMinion,

  // Evolution — migrate minions when schemas change (preserves removed fields in _legacy)
  migrateMinion,

  // Utilities
  generateId, now, SPEC_VERSION,
} from 'minions-sdk';
```

**Python SDK — Core Imports:**
```python
from minions import (
    # Types
    Minion, MinionType, Relation, FieldDefinition, FieldValidation,
    CreateMinionInput, UpdateMinionInput, CreateRelationInput,
    ExecutionResult, Executable, ValidationError, ValidationResult,
    # Validation
    validate_field, validate_fields,
    # Built-in Schemas (10 types)
    note_type, link_type, file_type, contact_type,
    agent_type, team_type, thought_type, prompt_template_type,
    test_case_type, task_type, builtin_types,
    # Registry
    TypeRegistry,
    # Relations
    RelationGraph,
    # Lifecycle
    create_minion, update_minion, soft_delete, hard_delete, restore_minion,
    # Evolution
    migrate_minion,
    # Utilities
    generate_id, now, SPEC_VERSION,
)
```

**Key Concepts:**
- A `MinionType` defines a schema (list of `FieldDefinition`s) — each field has `name`, `type`, `label`, `required`, `defaultValue`, `options`, `validation`
- A `Minion` is an instance with `id`, `title`, `minionTypeId`, `fields` (dict), `status`, `tags`, timestamps
- A `Relation` is a typed directional link (12 types: `parent_of`, `depends_on`, `implements`, `relates_to`, `inspired_by`, `triggers`, `references`, `blocks`, `alternative_to`, `part_of`, `follows`, `integration_link`)
- Field types: `string`, `number`, `boolean`, `date`, `select`, `multi-select`, `url`, `email`, `textarea`, `tags`, `json`, `array`
- `TypeRegistry` auto-loads 10 built-in types; register custom types with `registry.register(myType)`
- `createMinion(input, type)` validates fields against the schema and returns `{ minion, validation }` (TS) or `(minion, validation)` tuple (Python)
- Both SDKs serialize to identical camelCase JSON; Python provides `to_dict()` / `from_dict()` for conversion

**IMPORTANT:** Do NOT recreate these primitives. Import them from `minions-sdk` (npm) / `minions` (PyPI). Build your domain-specific types and utilities ON TOP of the SDK.

---

**WHAT YOU NEED TO CREATE**

**1. THE SPECIFICATION** (`/spec`)

Write a complete markdown specification document covering:

- Motivation and goals — why agents need structured time management
- Glossary of terms specific to calendar and scheduling systems
- Core type definitions for all six minion types with full field schemas
- Recurrence rule format (RRULE) — full syntax support for daily, weekly, monthly, yearly patterns
- Conflict detection algorithm — how overlapping events are identified
- Availability calculation — computing free/busy time slots across date ranges
- Timezone handling — converting events across timezones
- iCal format specification — import and export semantics
- Integration patterns — Google Calendar, Outlook, and other external calendar systems via `integration_link`
- Natural language parsing — supported formats for event creation
- Best practices for time-based agent workflows
- Conformance checklist for implementations

**2. THE CORE LIBRARY** (`/packages/core`)

A framework-agnostic TypeScript library built on `minions-sdk`. Must include:

- Full TypeScript type definitions for all calendar-specific types
- `CalendarEngine` class — core scheduling and recurrence engine
  - `expandRecurrence(eventId, startDate, endDate)` — generate event instances from RRULE
  - `validateRRule(ruleString)` — validate recurrence rule syntax
  - `nextOccurrence(eventId, afterDate)` — find next occurrence of recurring event
  - `occurrencesBetween(eventId, startDate, endDate)` — get all occurrences in range
- `ConflictDetector` class — identify scheduling conflicts
  - `findConflicts(eventIds[], dateRange?)` — detect overlapping events
  - `hasConflict(event1, event2)` — check if two events overlap
  - `suggestAlternatives(eventId)` — find conflict-free time slots
- `AvailabilityEngine` class — compute free/busy time
  - `getAvailability(agentId, dateRange)` — return free/busy slots
  - `findFreeSlots(agentId, duration, dateRange)` — find available windows of given duration
  - `bookSlot(agentId, timeSlot)` — create booking in free slot
- `TimezoneManager` class — handle timezone conversions
  - `convert(event, fromTz, toTz)` — convert event times across timezones
  - `localTime(event, timezone)` — display event in local timezone
  - `utcTime(event)` — normalize to UTC
- `ICalExporter` class — export to iCal format
  - `toICal(eventIds[])` — generate .ics file content
  - `fromICal(icsContent)` — parse .ics and create event minions
- `NaturalLanguageParser` class — parse human-readable event descriptions
  - `parse(text)` — extract event details from natural language (e.g., "meeting tomorrow at 3pm")
  - `parseRecurrence(text)` — extract recurrence patterns (e.g., "every Monday")
  - Returns structured event data ready for minion creation
- Clean public API with comprehensive JSDoc documentation
- Zero storage opinions — works with any backend

**3. THE PYTHON SDK** (`/packages/python`)

A complete Python port of the core library with identical functionality:

- Python type hints for all classes and methods
- `CalendarEngine`, `ConflictDetector`, `AvailabilityEngine`, `TimezoneManager`, `ICalExporter`, `NaturalLanguageParser` classes
- Same method signatures as TypeScript version (following Python naming conventions)
- Serializes to identical JSON format as TypeScript SDK (cross-language interoperability)
- Full docstrings compatible with Sphinx documentation generation
- Published to PyPI as `minions-calendar`

**4. THE CLI** (`/packages/cli`)

A command-line tool called `calendar` that provides:

```bash
calendar add "Team standup" --recur daily --time 09:00
# Create recurring daily event at 9am

calendar show --week
# Display current week's events

calendar conflicts
# Show all scheduling conflicts

calendar availability --from today --days 7
# Show free/busy slots for next 7 days

calendar export --format ical
# Export all events to .ics file

calendar import work.ics
# Import events from iCal file

calendar next
# Show next upcoming event
```

Additional features:
- Interactive mode for event creation with prompts for all fields
- Colored output for conflict warnings and availability display
- JSON output mode for programmatic usage
- Natural language event input: `calendar add "meeting tomorrow at 3pm"`
- Config file support (`.calendarrc.json`) for default timezone and settings

**5. THE DOCUMENTATION SITE** (`/apps/docs`)

Built with Astro Starlight. Must include:

- Landing page — "Structured time management for AI agents" positioning
- Getting started guide with both TypeScript and Python examples
- Core concepts:
  - Events vs. recurring events
  - RRULE recurrence syntax and examples
  - Conflict detection and resolution
  - Availability windows and booking
  - Timezone handling best practices
- API reference for both TypeScript and Python
  - Dual-language code tabs for all examples
  - Auto-generated from JSDoc/docstrings where possible
- Guides:
  - Creating recurring events with RRULE
  - Detecting and resolving conflicts
  - Computing availability across timezones
  - Integrating with Google Calendar and Outlook
  - Importing and exporting iCal files
  - Natural language event parsing
  - Building time-aware agent workflows
- CLI reference with example commands
- Integration examples:
  - Google Calendar sync workflow
  - Scheduling agent tasks with time constraints
  - Multi-agent coordination via availability
  - Automated meeting scheduling
- Best practices for calendar management
- Contributing guide

**6. OPTIONAL: THE WEB APP** (`/apps/web`)

A visual calendar interface (optional but recommended):

- Month/week/day view calendar grid
- Event creation and editing interface
- Recurrence rule builder UI
- Conflict visualization with color coding
- Availability timeline view
- iCal import/export interface
- Integration setup for external calendars
- Built with Next.js or SvelteKit (your choice based on ecosystem consistency)

---

**PROJECT STRUCTURE**

Standard Minions ecosystem monorepo structure:

```
minions-calendar/
  packages/
    core/                 # TypeScript core library
      src/
        types.ts          # Type definitions
        CalendarEngine.ts
        ConflictDetector.ts
        AvailabilityEngine.ts
        TimezoneManager.ts
        ICalExporter.ts
        NaturalLanguageParser.ts
        index.ts          # Public API surface
      test/
      package.json
    python/               # Python SDK
      minions_calendar/
        __init__.py
        types.py
        calendar_engine.py
        conflict_detector.py
        availability_engine.py
        timezone_manager.py
        ical_exporter.py
        natural_language_parser.py
      tests/
      pyproject.toml
    cli/                  # CLI tool
      src/
        commands/
          add.ts
          show.ts
          conflicts.ts
          availability.ts
          export.ts
          import.ts
          next.ts
        index.ts
      package.json
  apps/
    docs/                 # Astro Starlight documentation
      src/
        content/
          docs/
            index.md
            getting-started.md
            concepts/
            guides/
            api/
              typescript/
              python/
            cli/
      astro.config.mjs
      package.json
    web/                  # Optional calendar UI
      src/
      package.json
  spec/
    v0.1.md              # Full specification
  examples/
    typescript/
      simple-event.ts
      recurring-event.ts
      conflict-detection.ts
      availability-check.ts
    python/
      simple_event.py
      recurring_event.py
      conflict_detection.py
      availability_check.py
  .github/
    workflows/
      ci.yml             # Lint, test, build for both TS and Python
      publish.yml        # Publish to npm and PyPI
  README.md
  LICENSE                # AGPL-3.0
  package.json           # Workspace root
```

---

**BEYOND STANDARD PATTERN**

These utilities and classes are specific to `@minions-calendar/sdk`:

**CalendarEngine**
- RRULE-compatible recurrence evaluation using standard RFC 5545 syntax
- Methods: `expandRecurrence()`, `validateRRule()`, `nextOccurrence()`, `occurrencesBetween()`
- Handles complex patterns: daily, weekly, monthly, yearly, custom intervals
- Supports UNTIL (end date), COUNT (number of occurrences), and INTERVAL modifiers
- Handles edge cases like last day of month, nth weekday, etc.

**ConflictDetector**
- Identifies overlapping time ranges across multiple events
- Returns conflict sets with all conflicting events grouped
- `suggestAlternatives()` finds next available slots based on availability
- Supports soft conflicts (overlapping but not identical) vs. hard conflicts (same time)
- Can prioritize events by importance/priority field for conflict resolution

**AvailabilityEngine**
- Computes free/busy status across date ranges
- Returns structured availability windows with metadata
- `findFreeSlots()` finds contiguous free time of specified duration
- Respects working hours, time zones, and custom availability rules
- Integrates with `booking` minions to reserve time slots

**TimezoneManager**
- Full timezone conversion using IANA timezone database
- Handles daylight saving time transitions
- Methods: `convert()`, `localTime()`, `utcTime()`
- Validates timezone identifiers
- Display helpers for human-readable timezone names

**ICalExporter**
- Full RFC 5545 iCalendar format support
- Export methods: `toICal()` generates .ics content
- Import methods: `fromICal()` parses .ics and creates minions
- Handles VEVENT, VTODO, VALARM components
- Preserves recurrence rules, timezones, and custom properties
- Compatible with Google Calendar, Apple Calendar, Outlook

**NaturalLanguageParser**
- Extracts event details from human text: "lunch with team tomorrow at noon"
- Parses recurrence: "standup every weekday at 9am"
- Detects dates: "tomorrow", "next Monday", "March 15"
- Detects times: "3pm", "15:00", "noon", "midnight"
- Returns structured event data ready for minion creation
- Fallback to defaults for ambiguous input

**Integration Hooks**
- `integration_link` relations connect events to external calendar systems
- Google Calendar sync via OAuth and Calendar API
- Outlook/Microsoft Graph integration
- Webhook support for real-time updates
- Bidirectional sync patterns (import and export)

---

**CLI COMMANDS**

All commands with detailed specifications:

**`calendar add <title> [options]`**
- Creates new event minion
- Options: `--time HH:MM`, `--date YYYY-MM-DD`, `--duration MINUTES`, `--recur PATTERN`
- Interactive mode if required fields missing
- Natural language support: `calendar add "meeting tomorrow at 3pm"`
- Returns created event ID

**`calendar show [options]`**
- Displays events in specified view
- Options: `--day`, `--week`, `--month`, `--from DATE`, `--to DATE`
- Formatted output with time blocks and event titles
- Color codes conflicts in red
- Accepts `--json` for structured output

**`calendar conflicts [options]`**
- Lists all scheduling conflicts
- Options: `--from DATE`, `--to DATE`, `--resolve`
- Shows conflicting event pairs with overlap details
- `--resolve` suggests alternative times

**`calendar availability [options]`**
- Shows free/busy time slots
- Options: `--from DATE`, `--days N`, `--duration MINUTES`
- Displays availability windows as time blocks
- Highlights bookable slots
- Can filter by minimum duration

**`calendar export [options]`**
- Exports events to iCal format
- Options: `--format ical`, `--output FILE`, `--from DATE`, `--to DATE`
- Outputs to stdout or file
- Includes all recurrence rules and timezones

**`calendar import <file>`**
- Imports events from .ics file
- Creates event minions from VEVENT components
- Links recurring events with proper RRULE
- Returns count of imported events

**`calendar next [options]`**
- Shows next upcoming event
- Options: `--count N` to show next N events
- Displays event title, time, location
- Accounts for current timezone

---

**DUAL SDK REQUIREMENTS**

Critical cross-language compatibility requirements:

**Serialization Parity**
- Both TypeScript and Python SDKs must serialize minions to identical JSON format
- Field names, types, and structure must match exactly
- Relation types and metadata must be interchangeable
- RRULE strings must be identical across languages

**API Consistency**
- Same method names (adjusted for language conventions: TypeScript camelCase, Python snake_case)
- Same parameters and return types
- Same class hierarchies and interfaces

**Documentation Parity**
- Every code example in docs must have both TypeScript and Python versions
- Use Astro Starlight's code tabs: `<Tabs><TabItem label="TypeScript">...</TabItem><TabItem label="Python">...</TabItem></Tabs>`
- API reference must document both languages side by side

**Testing Parity**
- Shared test fixtures (JSON files) that both SDKs can consume
- Identical test case coverage
- Cross-language integration tests (TypeScript SDK creates event, Python SDK reads it)

---

**FIELD SCHEMAS**

Define these Minion Types with full JSON Schema definitions:

**`event`**
```typescript
{
  id: string;
  title: string;
  description?: string;
  startTime: Date;              // ISO 8601 datetime
  endTime: Date;                // ISO 8601 datetime
  location?: string;
  attendees?: string[];         // Array of participant names/emails
  timezone: string;             // IANA timezone identifier (e.g., "America/New_York")
  status: 'confirmed' | 'tentative' | 'cancelled';
  priority?: 'low' | 'medium' | 'high' | 'urgent';
  tags?: string[];
  metadata?: Record<string, any>;
  createdAt: Date;
  updatedAt: Date;
}
```

**`recurring-event`**
```typescript
{
  id: string;
  title: string;
  description?: string;
  startTime: Date;              // Start time of first occurrence
  endTime: Date;                // End time of first occurrence
  rrule: string;                // RRULE format (e.g., "FREQ=DAILY;INTERVAL=1")
  recurrenceEnd?: Date;         // Last occurrence date (UNTIL)
  occurrenceCount?: number;     // Number of occurrences (COUNT)
  location?: string;
  attendees?: string[];
  timezone: string;
  status: 'confirmed' | 'tentative' | 'cancelled';
  priority?: 'low' | 'medium' | 'high' | 'urgent';
  exceptions?: Date[];          // Dates to skip (cancelled occurrences)
  tags?: string[];
  metadata?: Record<string, any>;
  createdAt: Date;
  updatedAt: Date;
}
```

**`reminder`**
```typescript
{
  id: string;
  title: string;
  triggerTime: Date;            // When to trigger the reminder
  method: 'notification' | 'email' | 'webhook';
  message?: string;
  metadata?: Record<string, any>;
  triggered: boolean;
  triggeredAt?: Date;
  createdAt: Date;
  updatedAt: Date;
}
```
Relations: `references` → `event` or `recurring-event`

**`availability`**
```typescript
{
  id: string;
  title: string;
  startTime: Date;
  endTime: Date;
  status: 'free' | 'busy' | 'tentative' | 'out-of-office';
  timezone: string;
  workingHours?: {
    start: string;              // HH:MM format
    end: string;                // HH:MM format
    daysOfWeek: number[];       // 0 = Sunday, 6 = Saturday
  };
  metadata?: Record<string, any>;
  createdAt: Date;
  updatedAt: Date;
}
```

**`booking`**
```typescript
{
  id: string;
  title: string;
  description?: string;
  startTime: Date;
  endTime: Date;
  bookedBy?: string;            // Name or ID of person who booked
  confirmationStatus: 'pending' | 'confirmed' | 'cancelled';
  timezone: string;
  cancellationPolicy?: string;
  metadata?: Record<string, any>;
  createdAt: Date;
  updatedAt: Date;
}
```

**`timezone`**
```typescript
{
  id: string;
  title: string;                // Human-readable name (e.g., "Eastern Time")
  identifier: string;           // IANA timezone (e.g., "America/New_York")
  offset: string;               // UTC offset (e.g., "-05:00")
  metadata?: Record<string, any>;
  createdAt: Date;
  updatedAt: Date;
}
```

---

**TONE AND POSITIONING**

This is a serious tool for time management and agent coordination. Position it as:

- **Structured time awareness for AI agents** — agents can schedule themselves and avoid conflicts
- **RRULE-compatible recurrence** — industry-standard recurring event support
- **Calendar integration** — not a silo, works with existing calendar systems
- **Production-ready** — not a toy, built for real scheduling workflows

Avoid:
- Treating calendars as trivial to-do lists
- Oversimplifying complex recurrence patterns
- Ignoring timezone complexity

The README should open with a concrete example: creating an event, checking for conflicts, finding availability, and syncing with Google Calendar. Make it immediately tangible.

---

**INTEGRATION EXAMPLES**

Include working examples for:

**Google Calendar Integration** (TypeScript)
```typescript
import { CalendarEngine, ICalExporter } from '@minions-calendar/sdk';
import { google } from 'googleapis';

const calendar = google.calendar('v3');
const engine = new CalendarEngine();

// Export minions events to Google Calendar
const exporter = new ICalExporter();
const icalData = exporter.toICal(eventIds);

await calendar.events.import({
  calendarId: 'primary',
  requestBody: icalData
});
```

**Agent Self-Scheduling** (Python)
```python
from minions_calendar import CalendarEngine, ConflictDetector, AvailabilityEngine

# Agent checks availability before scheduling task
engine = AvailabilityEngine()
free_slots = engine.find_free_slots(
    agent_id='agent-001',
    duration=60,  # 60 minutes needed
    date_range=('2026-02-20', '2026-02-27')
)

# Book first available slot
if free_slots:
    slot = free_slots[0]
    engine.book_slot('agent-001', slot)
```

**Conflict Detection Workflow** (TypeScript)
```typescript
import { ConflictDetector } from '@minions-calendar/sdk';

const detector = new ConflictDetector();
const conflicts = await detector.findConflicts(allEventIds);

if (conflicts.length > 0) {
  // Suggest alternative times
  for (const conflict of conflicts) {
    const alternatives = await detector.suggestAlternatives(conflict.eventId);
    console.log(`Conflict at ${conflict.time}: try ${alternatives[0]}`);
  }
}
```

---

**DELIVERABLES**

Produce all files necessary to bootstrap this project completely:

1. **Full specification** (`/spec/v0.1.md`) — complete enough to implement from
2. **TypeScript core library** (`/packages/core`) — fully functional, well-tested
3. **Python SDK** (`/packages/python`) — feature parity with TypeScript
4. **CLI tool** (`/packages/cli`) — all commands working with helpful output
5. **Documentation site** (`/apps/docs`) — complete with dual-language examples
6. **README** — compelling, clear, with concrete examples
7. **Examples** — working code in both TypeScript and Python
8. **CI/CD setup** — lint, test, and publish workflows for both languages

Every file should be production quality — not stubs, not placeholders. The spec should be complete. The core libraries should be fully functional. The docs should be ready to publish. The CLI should be ready to install and use.

---

**START SYSTEMATICALLY**

1. Write the specification first — nail down RRULE syntax, conflict detection algorithm, and field schemas
2. Implement TypeScript core library with full type definitions
3. Port to Python maintaining exact serialization compatibility
4. Build CLI using the core library
5. Write documentation with dual-language examples throughout
6. Create working examples demonstrating key workflows
7. Write the README with concrete use cases

This is time-awareness infrastructure for AI agents. Agents need to schedule themselves, coordinate with other agents, and integrate with human calendar systems. Get it right.
