# College Event Kit Relay — Test Cases

## 1. Document Information

- **Project:** College Event Kit Relay / Kit-Booking Application
- **Module:** Kit Booking and Management
- **Document type:** Software Test Case Specification
- **Version:** 1.0
- **Status:** Proposed test cases (not yet executed)

## 2. Scope and Assumptions

The application allows college clubs to share one event kit containing a camera, tripod, and microphone. Clubs can request bookings, administrators can approve or reject requests, and authorized users can check out and return the kit.

Assumptions for these test cases:

1. A booking request must have a valid start time and end time.
2. The start time must be earlier than the end time.
3. Booking requests initially have `PENDING` status.
4. Pending requests do not block other bookings in this design.
5. The administrator rechecks availability when approving a request.
6. Confirmed bookings and bookings currently checked out block overlapping reservations.
7. Cancelled and rejected bookings do not block availability.
8. Back-to-back bookings are allowed when one booking's end time equals the next booking's start time. No handover buffer is assumed.
9. The kit is usable only when all three required items are usable.
10. The database must prevent two overlapping bookings from both being confirmed, including simultaneous requests.

## 3. Test Case Format

Each test case includes an ID, scenario, preconditions, steps/input, expected result, and priority.

Priority meanings:
- **High:** Core business rule, data integrity, access control, or booking conflict.
- **Medium:** Important supporting behavior.
- **Low:** Cosmetic or less critical behavior.

---

## 4. Functional Test Cases

### TC-001 — View available kit

- **Priority:** High
- **Preconditions:** The event kit is registered and ready for use.
- **Steps/Input:** Open the kit availability page.
- **Expected Result:** The event kit is displayed with its camera, tripod, microphone, and current availability/readiness status.

### TC-002 — Submit a valid booking request

- **Priority:** High
- **Preconditions:** The club is registered and the requested slot is available.
- **Steps/Input:** Request the kit for October 12, 10:00 AM–12:00 PM.
- **Expected Result:** A booking is created with a unique booking ID and `PENDING` status.

### TC-003 — Approve a pending booking

- **Priority:** High
- **Preconditions:** A valid pending booking exists and the requested slot remains available.
- **Steps/Input:** An administrator approves the booking.
- **Expected Result:** Availability is rechecked; the booking becomes `CONFIRMED` only if the kit remains available.

### TC-004 — View bookings for a club

- **Priority:** Medium
- **Preconditions:** The club has one or more bookings.
- **Steps/Input:** Open the club's booking history.
- **Expected Result:** The correct bookings and their statuses are displayed for that club.

### TC-005 — Cancel an eligible booking

- **Priority:** High
- **Preconditions:** The club owns a booking that can be cancelled under the booking policy.
- **Steps/Input:** Cancel the booking using its booking ID.
- **Expected Result:** The booking becomes `CANCELLED` and the time slot is released.

### TC-006 — Check out the kit

- **Priority:** High
- **Preconditions:** The booking is confirmed, its start time has arrived, and the user is authorized.
- **Steps/Input:** Record collection of the kit.
- **Expected Result:** The booking changes to `CHECKED_OUT`.

### TC-007 — Return the kit

- **Priority:** High
- **Preconditions:** The booking is `CHECKED_OUT`.
- **Steps/Input:** Record the return and inspect all equipment.
- **Expected Result:** The return is recorded, the booking changes to `COMPLETED`, and equipment condition is updated if needed.

### TC-008 — Reject a booking request

- **Priority:** Medium
- **Preconditions:** A pending booking request exists.
- **Steps/Input:** An administrator rejects the request.
- **Expected Result:** The booking becomes `REJECTED` and does not block future bookings.

### TC-009 — View booking details

- **Priority:** Medium
- **Preconditions:** The booking ID exists and the viewer has permission to access it.
- **Steps/Input:** Open the booking details.
- **Expected Result:** The correct booking ID, club, kit, time range, and status are displayed.

---

## 5. Negative and Validation Test Cases

### TC-010 — Start time is after end time

- **Priority:** High
- **Preconditions:** The club can submit a booking request.
- **Steps/Input:** Request a slot from 2:00 PM to 1:00 PM on the same day.
- **Expected Result:** The request is rejected with a clear validation message; no booking is created.

### TC-011 — Start time equals end time

- **Priority:** High
- **Preconditions:** The club can submit a booking request.
- **Steps/Input:** Request a slot from 10:00 AM to 10:00 AM.
- **Expected Result:** The request is rejected; no booking is created.

### TC-012 — Request a booking in the past

- **Priority:** High
- **Preconditions:** The system date/time is configured.
- **Steps/Input:** Request a time slot that has already ended.
- **Expected Result:** The request is rejected according to the no-past-bookings policy.

### TC-013 — Start time is missing

- **Priority:** Medium
- **Preconditions:** The booking form is open.
- **Steps/Input:** Submit the form without a start time.
- **Expected Result:** A required-field validation message appears; no booking is created.

### TC-014 — End time is missing

- **Priority:** Medium
- **Preconditions:** The booking form is open.
- **Steps/Input:** Submit the form without an end time.
- **Expected Result:** A required-field validation message appears; no booking is created.

### TC-015 — Unregistered club submits a booking

- **Priority:** High
- **Preconditions:** A club ID that does not exist is available for the test.
- **Steps/Input:** Submit a booking request with the unknown club ID.
- **Expected Result:** The request is rejected; no booking is created.

### TC-016 — Cancel a nonexistent booking

- **Priority:** Medium
- **Preconditions:** Booking ID `B9999` does not exist.
- **Steps/Input:** Attempt to cancel booking `B9999`.
- **Expected Result:** A not-found response/message is returned and no data is changed.

### TC-017 — Club attempts to cancel another club's booking

- **Priority:** High
- **Preconditions:** Club A and Club B exist; the booking belongs to Club B.
- **Steps/Input:** Club A attempts to cancel Club B's booking.
- **Expected Result:** The operation is denied; the booking remains unchanged.

### TC-018 — Cancel the same booking twice

- **Priority:** Medium
- **Preconditions:** The booking has already been cancelled.
- **Steps/Input:** Submit a second cancellation request for the same booking.
- **Expected Result:** The system handles the repeated request safely, gives a clear response, and does not corrupt booking data.

### TC-019 — Check out a cancelled booking

- **Priority:** High
- **Preconditions:** The booking status is `CANCELLED`.
- **Steps/Input:** Attempt to check out the kit for that booking.
- **Expected Result:** Checkout is rejected because the booking is not active.

### TC-020 — Approve a rejected booking

- **Priority:** Medium
- **Preconditions:** The booking status is `REJECTED`.
- **Steps/Input:** Attempt to approve the rejected booking without creating a new request.
- **Expected Result:** The invalid status transition is rejected.

---

## 6. Booking Conflict and Boundary Test Cases

Assume Club A has a **confirmed booking from 10:00 AM to 12:00 PM** on the same date.

### TC-021 — Request exactly the same slot

- **Priority:** High
- **Preconditions:** Club A's booking is confirmed from 10:00 AM to 12:00 PM.
- **Steps/Input:** Club B requests 10:00 AM–12:00 PM.
- **Expected Result:** The overlapping request must not be confirmed.

### TC-022 — Request a partially overlapping slot at the beginning

- **Priority:** High
- **Preconditions:** Club A's booking is confirmed from 10:00 AM to 12:00 PM.
- **Steps/Input:** Club B requests 11:00 AM–1:00 PM.
- **Expected Result:** The request must not be confirmed because the slots overlap.

### TC-023 — Request a partially overlapping slot at the end

- **Priority:** High
- **Preconditions:** Club A's booking is confirmed from 10:00 AM to 12:00 PM.
- **Steps/Input:** Club B requests 9:00 AM–11:00 AM.
- **Expected Result:** The request must not be confirmed because the slots overlap.

### TC-024 — Request a slot that contains the existing booking

- **Priority:** High
- **Preconditions:** Club A's booking is confirmed from 10:00 AM to 12:00 PM.
- **Steps/Input:** Club B requests 9:00 AM–1:00 PM.
- **Expected Result:** The request must not be confirmed because it contains the existing booking interval.

### TC-025 — Request a back-to-back slot

- **Priority:** High
- **Preconditions:** Club A's booking is confirmed from 10:00 AM to 12:00 PM; no handover buffer is required.
- **Steps/Input:** Club B requests 12:00 PM–2:00 PM.
- **Expected Result:** The request may be confirmed if the kit is ready and no other conflicts exist.

### TC-026 — Request a slot ending when the existing slot starts

- **Priority:** Medium
- **Preconditions:** Club A's booking is confirmed from 10:00 AM to 12:00 PM.
- **Steps/Input:** Club B requests 8:00 AM–10:00 AM.
- **Expected Result:** The request may be confirmed if the kit is ready and no other conflicts exist.

### TC-027 — Request a slot previously held by a cancelled booking

- **Priority:** High
- **Preconditions:** A booking for the requested slot has status `CANCELLED`; no other conflicts exist.
- **Steps/Input:** Submit a new request for that slot.
- **Expected Result:** The cancelled booking does not block the new request.

### TC-028 — Request a slot that overlaps a pending request

- **Priority:** High
- **Preconditions:** A pending request exists for the requested time; pending requests do not reserve the slot in this design.
- **Steps/Input:** Submit another request for the same time and attempt approval of both requests.
- **Expected Result:** Pending status alone does not block the slot. Approval must recheck availability, and no two overlapping bookings may both become confirmed.

---

## 7. Equipment and Kit Management Test Cases

### TC-029 — All equipment is usable

- **Priority:** High
- **Preconditions:** Camera, tripod, and microphone are all in usable condition.
- **Steps/Input:** Check kit readiness.
- **Expected Result:** The kit is marked ready for booking if no other restriction applies.

### TC-030 — Camera is damaged

- **Priority:** High
- **Preconditions:** The camera is marked damaged; all three items are required for the kit.
- **Steps/Input:** Check kit availability or attempt a booking.
- **Expected Result:** The kit is marked unavailable and cannot be confirmed for booking until repaired.

### TC-031 — Tripod is under maintenance

- **Priority:** High
- **Preconditions:** The tripod is marked as under maintenance.
- **Steps/Input:** Check availability or attempt a booking.
- **Expected Result:** The kit is unavailable until the tripod is usable again.

### TC-032 — Microphone is missing

- **Priority:** High
- **Preconditions:** The microphone is marked missing.
- **Steps/Input:** Check availability or attempt a booking.
- **Expected Result:** The kit is marked unavailable and the missing item is recorded.

### TC-033 — Update equipment condition after return

- **Priority:** High
- **Preconditions:** The kit has been returned and inspection is in progress.
- **Steps/Input:** Mark an item as damaged during inspection.
- **Expected Result:** The condition update is saved and kit readiness reflects the damage.

### TC-034 — Unauthorized user updates equipment

- **Priority:** High
- **Preconditions:** A non-administrator account is signed in.
- **Steps/Input:** Attempt to change an equipment condition or maintenance status.
- **Expected Result:** The operation is denied.

### TC-035 — Check out an already checked-out kit

- **Priority:** High
- **Preconditions:** The booking or kit is already checked out.
- **Steps/Input:** Attempt another checkout.
- **Expected Result:** The duplicate checkout is rejected and the existing checkout record remains intact.

---

## 8. Integration, Data Integrity, and Concurrency Test Cases

### TC-036 — BookingService uses AvailabilityService

- **Priority:** High
- **Preconditions:** Booking and availability services are available.
- **Steps/Input:** Submit a valid booking request.
- **Expected Result:** BookingService checks availability before proceeding with booking creation/confirmation.

### TC-037 — AvailabilityService finds a conflict

- **Priority:** High
- **Preconditions:** A confirmed overlapping booking exists.
- **Steps/Input:** Check availability for the conflicting slot.
- **Expected Result:** AvailabilityService reports the conflict and the new booking is not confirmed.

### TC-038 — Database save fails

- **Priority:** High
- **Preconditions:** A test environment can simulate a database failure.
- **Steps/Input:** Submit a booking while the database write fails.
- **Expected Result:** The application reports failure; it does not falsely report success or leave a partially committed booking.

### TC-039 — Two clubs request the same slot simultaneously

- **Priority:** High
- **Preconditions:** The kit is initially available; two clubs can send requests concurrently.
- **Steps/Input:** Submit and approve overlapping booking requests at nearly the same time.
- **Expected Result:** At most one overlapping booking becomes confirmed. The database transaction or concurrency-control mechanism prevents double booking.

### TC-040 — Availability changes before approval

- **Priority:** High
- **Preconditions:** A pending request exists; another booking is confirmed for an overlapping slot before approval.
- **Steps/Input:** Attempt to approve the pending request.
- **Expected Result:** Approval fails safely because availability is rechecked; the request remains unconfirmed.

### TC-041 — Cancellation persists after refresh

- **Priority:** Medium
- **Preconditions:** A booking is successfully cancelled.
- **Steps/Input:** Refresh the application or reopen the booking list.
- **Expected Result:** The booking remains `CANCELLED` and its slot is no longer blocked by that booking.

### TC-042 — Refresh after booking submission

- **Priority:** Medium
- **Preconditions:** A booking request has been saved successfully.
- **Steps/Input:** Refresh the page or reopen the booking history.
- **Expected Result:** The saved booking remains visible and refreshing does not create a duplicate booking.

---

## 9. Test Execution Record Template

Use this template when executing a test. Do not mark a test as Pass until the actual result has been checked against the expected result.

| Test Case ID | Actual Result | Status (Not Run/Pass/Fail/Blocked) | Defect ID / Remarks |
|---|---|---|---|
| TC-001 |  | Not Run |  |
| TC-002 |  | Not Run |  |
| TC-003 |  | Not Run |  |
| TC-004 |  | Not Run |  |
| TC-005 |  | Not Run |  |

Add rows for the remaining test cases as testing proceeds.

## 10. Recommended Priority for Initial Testing

Run these first because they cover the most important business rules:

1. **TC-002:** Submit a valid booking.
2. **TC-003:** Approve a booking.
3. **TC-010:** Reject an invalid time range.
4. **TC-022:** Reject an overlapping booking.
5. **TC-025:** Allow a back-to-back booking.
6. **TC-030:** Handle damaged equipment.
7. **TC-039:** Prevent simultaneous conflicting confirmations.
8. **TC-040:** Recheck availability before approval.

## 11. Exit Criteria

Testing can be considered ready for review when:

- All high-priority test cases have been executed.
- No unresolved critical or high-severity defects remain.
- Overlapping bookings cannot both be confirmed.
- Invalid time ranges and unauthorized actions are rejected.
- Booking and equipment statuses persist correctly.
- Any failed or blocked tests are documented with remarks.

**Current status:** This document defines 42 proposed test cases. No application has been executed as part of preparing this document, so no test results are claimed.
