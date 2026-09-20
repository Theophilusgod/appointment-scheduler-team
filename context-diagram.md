# Context Diagram — Appointment Rescheduling Feature

This is a **System Context Diagram**: it shows the feature as a single box, the actors/external systems that interact with it, and the direction of the data/event flow. It intentionally does *not* show internal components — that level of detail belongs in a container/component diagram, not a context diagram.

```mermaid
flowchart TD
    Patient([Patient])
    Staff([Clinic Staff])

    subgraph System["Appointment Scheduling System"]
        Scheduler[Scheduling Service]
    end

    Notification[[Notification Service]]
    DB[(Appointments Database)]

    Patient -- "1. Requests reschedule\n(new date/time)" --> Scheduler
    Staff -- "Manages appointment records" --> Scheduler

    Scheduler -- "2. Reads original appointment time" --> DB
    Scheduler -- "3. Validates timing rule\n(< 24h before original?)" --> Scheduler
    Scheduler -- "4. Writes updated appointment\n+ late-change flag (if applicable)" --> DB
    Scheduler -- "5. Emits AppointmentRescheduled event" --> Notification
    Notification -- "6. Sends confirmation" --> Patient
    Scheduler -- "7. Displays confirmation message" --> Patient
```

## Actors & Systems

| Name | Type | Role |
|---|---|---|
| Patient | External actor | Initiates the reschedule request; receives confirmation |
| Clinic Staff | External actor | Manages/oversees appointment records (context for scope) |
| Scheduling Service | Core system (in scope) | Owns the reschedule logic, including the 24-hour late-change rule |
| Appointments Database | Data store | Source of truth for original appointment times |
| Notification Service | External system | Receives the `AppointmentRescheduled` event and notifies the patient |

## Notes on scope
- The 24-hour rule is evaluated by comparing the **time the reschedule request is received** against the **original appointment's start time** — not the new appointment time. This distinction matters for the acceptance criteria below.
- The Notification Service is treated as an external dependency the Scheduling Service integrates with via an event, not a component we're designing here.
