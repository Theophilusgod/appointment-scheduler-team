# User Story — Appointment Rescheduling

## Story

**As a** patient
**I want to** reschedule my appointment to a new date/time
**So that** I can adjust my visit without calling the clinic, while staff are still alerted when I change plans at the last minute

**Priority:** High
**Epic:** Appointment Management

---

## Acceptance Criteria (Gherkin)

### Scenario 1: Patient reschedules an appointment within 24 hours of the original time (late change)

```gherkin
Scenario: Patient reschedules an appointment within 24 hours of window
  Given a patient has a scheduled appointment for "2026-10-15T10:00:00Z"
  When the patient requests a reschedule at "2026-10-14T14:00:00Z" to a new time of "2026-10-20T09:00:00Z"
  Then the system should apply a late-change flag
  And emit an "AppointmentRescheduled" event to the Notification Service
  And display a confirmation message with updated details to the patient
```

### Scenario 2: Patient reschedules an appointment well ahead of the original time (happy path, no flag)

```gherkin
Scenario: Patient reschedules an appointment more than 24 hours before the original time
  Given a patient has a scheduled appointment for "2026-10-15T10:00:00Z"
  When the patient requests a reschedule at "2026-10-10T09:00:00Z" to a new time of "2026-10-20T09:00:00Z"
  Then the system should not apply a late-change flag
  And emit an "AppointmentRescheduled" event to the Notification Service
  And display a confirmation message with updated details to the patient
```

### Scenario 3: Patient attempts to reschedule to a time in the past (invalid request)

```gherkin
Scenario: Patient attempts to reschedule to a past date/time
  Given a patient has a scheduled appointment for "2026-10-15T10:00:00Z"
  When the patient requests a reschedule to "2026-09-01T09:00:00Z"
  Then the system should reject the reschedule request
  And display an error message indicating the new time must be in the future
  And no "AppointmentRescheduled" event should be emitted
```

### Scenario 4: Patient reschedules exactly at the 24-hour boundary (edge case)

```gherkin
Scenario: Patient reschedules exactly 24 hours before the original appointment
  Given a patient has a scheduled appointment for "2026-10-15T10:00:00Z"
  When the patient requests a reschedule at exactly "2026-10-14T10:00:00Z" to a new time of "2026-10-20T09:00:00Z"
  Then the system should not apply a late-change flag
  And emit an "AppointmentRescheduled" event to the Notification Service
  And display a confirmation message with updated details to the patient
```

---

## Definition of Done
- [ ] Late-change flag logic correctly compares the **request timestamp** to the **original appointment time**, not the new appointment time
- [ ] `AppointmentRescheduled` event payload includes: appointment ID, old time, new time, late-change flag (boolean)
- [ ] Confirmation message displayed to patient reflects the updated appointment details
- [ ] Invalid reschedule requests (past dates) are rejected with a clear error and do not emit an event
- [ ] Boundary condition (exactly 24h) is covered by an automated test
- [ ] Context diagram and this file committed to the team GitHub Project Board
