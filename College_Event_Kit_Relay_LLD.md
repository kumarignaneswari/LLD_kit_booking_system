# College Event Kit Relay

## 1. Project Overview

**College Event Kit Relay** is a college equipment reservation and tracking application. It allows multiple college clubs to reserve one shared event equipment kit, while helping organizers manage checkout, return inspections, missing or damaged equipment, booking disruptions, notifications, and audit history.

### Physical equipment

The application initially manages one physical kit:

- **Kit:** `KIT-001`
- **Camera:** `ITEM-CAM-01`
- **Tripod:** `ITEM-TRI-01`
- **Microphone:** `ITEM-MIC-01`

The design should support adding more equipment items later without rewriting the business logic.

### Core rules

- Only one club may reserve the kit during a given time interval.
- Every booking must have a valid start and end time.
- A mandatory **15-minute inspection gap** is required after a booking before the next booking may start.
- The kit must pass readiness checks before it can be handed over.
- Every required item must be inspected at checkout and return.
- Missing or damaged equipment must be tracked through incidents.
- A confirmed booking can become **at risk** without being automatically cancelled.
- Protected operations must be authorized by the backend.
- Important actions must be recorded in an audit trail.

## 2. Project Goals

1. Allow club representatives to register, log in, and manage their own bookings.
2. Show availability for the kit.
3. Prevent overlapping or otherwise conflicting bookings.
4. Enforce the 15-minute inspection gap.
5. Allow authorized organizers to check equipment out and record returns.
6. Record the condition of each equipment item individually.
7. Track missing, damaged, recovered, replaced, or repaired items.
8. Identify future bookings potentially affected by an incident.
9. Notify affected clubs and organizers.
10. Maintain booking, inspection, incident, and audit history.
11. Enforce authentication and role-based authorization.
12. Provide automated tests for normal cases, failures, and boundary conditions.

## 3. Actors and Permissions

### 3.1 Club Representative

A club representative can:

- Register and log in.
- View kit details and availability.
- Create a booking for their own club.
- View their club's bookings and booking history.
- View an authorized booking's details.
- Cancel an eligible booking according to the cancellation policy.
- View notifications and disruption warnings.

A club representative cannot manage inventory, perform checkout, perform official return inspections, resolve incidents, manage users, or view another club's private booking information.

### 3.2 Equipment Organizer

An equipment organizer can:

- Log in to the organizer dashboard.
- View bookings and the daily schedule.
- View kit readiness and individual equipment status.
- Perform checkout and return inspections.
- Record the condition of the camera, tripod, and microphone.
- Report missing or damaged equipment.
- Create and update incidents.
- Record recovered or replacement equipment.
- Resolve incidents after verifying the outcome.
- Identify potentially affected future bookings.
- Notify affected clubs.
- Cancel or reschedule bookings when authorized, with a recorded reason.
- View relevant audit and notification history.

### 3.3 Administrator

An administrator has organizer permissions and can also:

- Create, update, activate, or deactivate clubs.
- Create or deactivate user accounts.
- Assign roles and link representatives to clubs.
- Configure permitted booking durations and cancellation policies.
- View system-wide audit records.
- Manage application-level settings.

A user must never be allowed to grant themselves administrator privileges.

### 3.4 System Responsibilities

The application automatically:

- Validates booking dates and times.
- Checks conflicts and the inspection gap.
- Prevents concurrent conflicting bookings.
- Updates booking, equipment, and incident state.
- Identifies potentially affected future bookings.
- Creates notification and audit records.
- Retries failed notification delivery where appropriate.

The system is the application itself, not a separate human account.

## 4. Functional Modules

Each module has a focused responsibility. A module may contain multiple models, services, repositories, and API endpoints.

### 4.1 Authentication and Authorization

- Register, log in, and log out.
- Verify the identity of the current user.
- Hash passwords securely.
- Enforce roles and permissions on the backend.
- Restrict access to private bookings and administrative operations.
- Expire sessions or tokens securely.
- Support account recovery if password-based login is used.

### 4.2 User Management

- Maintain user profiles.
- Link club representatives to their clubs.
- Activate or deactivate accounts.
- Assign roles through authorized administration.
- Prevent deactivated users from performing protected actions.

### 4.3 Club Management

- Create and update club details.
- Store contact information.
- Activate or deactivate clubs.
- Display club booking history.
- Prevent inactive clubs from creating new bookings.

### 4.4 Kit and Equipment Management

- Maintain the single kit `KIT-001`.
- Track each equipment item independently.
- Record whether an item is available, missing, damaged, or under repair.
- Calculate kit readiness from item conditions and operational restrictions.
- Preserve equipment and incident history.

### 4.5 Availability Management

- Display available dates and time slots.
- Check all relevant existing bookings.
- Enforce the 15-minute inspection gap.
- Exclude cancelled bookings from reservation conflicts.
- Consider operational restrictions.
- Prevent concurrent requests from reserving conflicting times.

### 4.6 Booking Management

- Create and retrieve bookings.
- View bookings by club, date, or status.
- Cancel eligible bookings.
- Maintain booking lifecycle state.
- Track disruption risk separately from booking status.
- Send confirmation and cancellation notifications.

### 4.7 Checkout, Return, and Inspection

- Validate a booking before checkout.
- Verify the organizer's permissions.
- Record checkout time and item conditions.
- Block handover if required equipment is missing or unusable.
- Record return time and inspect every required item.
- Compare checkout and return results.
- Trigger incident handling when problems are found.

### 4.8 Incident and Disruption Management

- Report missing or damaged equipment.
- Associate an incident with an item and, when applicable, a booking.
- Record the description, reporter, timestamps, and resolution notes.
- Identify future bookings potentially affected by an incident.
- Notify affected clubs and organizers.
- Track recovery, replacement, repair, or cancellation.
- Resolve an incident only after verification.

### 4.9 Notification Management

- Send booking confirmations.
- Send cancellation notifications.
- Send disruption warnings.
- Store notification delivery status.
- Retry failed deliveries.
- Avoid duplicate messages where practical.

### 4.10 Audit and History

- Record booking creation and cancellation.
- Record checkout and return.
- Record equipment status changes.
- Record incident creation and resolution.
- Record who performed each action and when.
- Restrict audit history to authorized staff.

### 4.11 System Monitoring

- Record application errors.
- Track failed notification attempts.
- Log important operational failures.
- Provide a basic health endpoint.
- Avoid storing passwords, tokens, or unnecessary sensitive data in logs.

Advanced monitoring can be added after the core application is working.

## 5. Booking Rules

All business rules must be enforced by backend services, with database safeguards where possible. Frontend validation alone is not sufficient.

### 5.1 Time Validation

- Start time must be earlier than end time.
- Reject bookings in the past.
- Enforce any configured booking duration or operating-hours policy.
- Store timestamps consistently, preferably in UTC.
- Display times in the college's local timezone.
- Reject invalid or ambiguous date/time inputs.

### 5.2 Booking Conflicts and Inspection Gap

If Club A books the kit from **2:00 PM to 3:00 PM**:

- A booking starting at **3:10 PM** must be rejected.
- A booking starting at **3:14 PM** must be rejected.
- A booking starting at **3:15 PM** passes the gap rule.
- A booking starting at **3:30 PM** passes the gap rule.

Passing the time rule does not guarantee the kit is operationally ready.

The system must check both:
1. Overlap with active reservations.
2. The 15-minute gap after the previous reservation.

Cancelled bookings must not block availability.

### 5.3 Concurrency

Two users may submit booking requests at nearly the same time. The backend must make conflict validation and booking creation an atomic operation, using a transaction and suitable locking or database concurrency controls. A simple “check availability, then save later” implementation is insufficient.

### 5.4 Booking Lifecycle

Recommended booking statuses:

- `CONFIRMED`
- `CHECKED_OUT`
- `COMPLETED`
- `CANCELLED`

If college policy requires approval, add `PENDING_APPROVAL`. Do not add approval as a requirement unless it is actually needed.

Track booking risk separately. For example, a booking may remain `CONFIRMED` while its `risk_reason` is `Tripod missing`.

### 5.5 Checkout and Return

- Only an authorized organizer can perform checkout and return.
- Checkout must refer to an eligible booking.
- Every required equipment item must be inspected.
- Missing or unusable equipment blocks handover.
- Return inspection must record each item's condition.
- Missing or damaged equipment must create or update an incident.
- The kit must not become ready solely because 15 minutes have elapsed.
- Readiness requires the necessary inspection and usable equipment.

### 5.6 Cancellation and Disruptions

- Representatives may cancel only eligible bookings belonging to their club.
- Administrators may cancel bookings only according to their permissions and policy.
- Store the cancellation reason and actor.
- Do not automatically cancel all future bookings when an incident occurs.
- Flag potentially affected bookings and notify relevant people.
- After recovery or replacement, verify readiness before clearing the disruption.

## 6. Authentication and Security

Authentication answers **“Who is the user?”** Authorization answers **“What is the user allowed to do?”**

Security requirements:

- Store passwords using a modern password-hashing algorithm.
- Never store plain-text passwords.
- Use one consistent authentication approach: secure sessions or signed, short-lived access tokens.
- Use HTTPS in deployment.
- Check permissions on every protected API request.
- Verify booking ownership using the authenticated user's identity.
- Never trust a role, user ID, or club ID supplied by the browser without verification.
- Rate-limit repeated login attempts.
- Keep secrets in environment variables, never in Git.
- Use secure cookie settings if cookie-based authentication is selected.
- Configure CORS narrowly for the actual frontend origin.
- Never return password hashes or sensitive account fields in API responses.
- Restrict role assignment and administrator management to authorized administrators.

## 7. Domain Models

Models represent the entities in the system. They hold the data they own and may provide small domain-level methods.

### 7.1 User

Attributes:

- `user_id`
- `name`
- `email`
- `password_hash`
- `role`
- `club_id` (optional for staff)
- `is_active`
- `created_at`

Responsibility: represent the account, assigned role, and relationship to a club.

### 7.2 Club

Attributes:

- `club_id`
- `name`
- `contact_email`
- `is_active`
- `created_at`

Responsibility: identify the club and determine whether it may create new bookings.

### 7.3 Booking

Attributes:

- `booking_id`
- `club_id`
- `kit_id`
- `start_time`
- `end_time`
- `status`
- `risk_reason` (optional)
- `created_at`
- `cancelled_at` (optional)
- `cancellation_reason` (optional)

Responsibility: store reservation details and preserve lifecycle and cancellation history.

### 7.4 EventKit

Attributes:

- `kit_id`
- `name`
- `description`

Responsibility: identify the physical kit and provide access to its equipment. Readiness should be calculated from item conditions, inspections, reservations, and operational restrictions rather than relying only on a manually maintained flag.

### 7.5 EquipmentItem

Attributes:

- `item_id`
- `kit_id`
- `name`
- `status`
- `condition_notes`
- `last_inspected_at`

Responsibility: track an item’s identity, condition, and operational status.

Initial items are the camera, tripod, and microphone. Do not hard-code their names into business logic so additional equipment can be supported later.

### 7.6 InspectionRecord

Attributes:

- `inspection_id`
- `booking_id`
- `inspector_id`
- `inspection_type`
- `inspected_at`
- `notes`

Responsibility: record who performed an inspection, when it happened, and whether it was for checkout or return.

### 7.7 InspectionItemResult

Attributes:

- `result_id`
- `inspection_id`
- `item_id`
- `condition`
- `notes`

Responsibility: store the condition of each inspected item. An inspection cannot be complete while required item results are missing.

### 7.8 Incident

Attributes:

- `incident_id`
- `item_id`
- `related_booking_id` (optional)
- `reported_by`
- `description`
- `status`
- `created_at`
- `resolution_notes`
- `resolved_at` (optional)

Responsibility: track the problem, recovery or replacement, and verified resolution.

### 7.9 Notification

Attributes:

- `notification_id`
- `user_id`
- `message`
- `type`
- `delivery_status`
- `created_at`
- `sent_at` (optional)
- `retry_count`

Responsibility: retain notification history and track delivery attempts.

### 7.10 AuditLog

Attributes:

- `audit_id`
- `actor_id`
- `action`
- `entity_type`
- `entity_id`
- `timestamp`
- `details`

Responsibility: provide traceability for important actions.

Audit records should not be editable or deletable through ordinary application features.

### 7.11 Enums

Define enums for:

- User role
- Booking status
- Equipment status
- Inspection type
- Item condition
- Incident status
- Notification delivery status

Example equipment statuses:

- `AVAILABLE`
- `MISSING`
- `DAMAGED`
- `UNDER_REPAIR`

Example inspection types:

- `CHECKOUT`
- `RETURN`

Keep item condition, equipment operational status, booking status, and booking risk separate because they represent different concepts.

## 8. Service Layer

Services coordinate business rules, models, repositories, notifications, and audit logging.

### AuthService

- Register users.
- Verify login credentials.
- Manage logout or session invalidation.

### AuthorizationService

- Check roles.
- Check booking ownership.
- Validate permissions before protected actions.

### UserService

- Retrieve and update profiles.
- Activate or deactivate accounts.
- Coordinate permitted user-management actions.

### ClubService

- Create and update clubs.
- Activate or deactivate clubs.
- Retrieve club booking history.

### BookingService

- Create bookings.
- Retrieve authorized booking details.
- Cancel eligible bookings.
- Apply booking lifecycle rules.
- Coordinate availability checks and persistence.

### AvailabilityService

- Find available time slots.
- Check overlapping reservations.
- Enforce the 15-minute inspection gap.
- Consider cancelled bookings and operational restrictions.

### InventoryService

- Retrieve the kit and its items.
- Check that every required item is usable.
- Determine whether the kit can be handed over.
- Update equipment status through authorized operations.

### HandoverService

- Validate a booking before checkout.
- Record checkout.
- Validate return.
- Coordinate inspection records.
- Block unsafe handover.

### InspectionService

- Create checkout and return records.
- Record item-by-item conditions.
- Validate inspection completeness.
- Identify discrepancies between checkout and return.

### DisruptionService

- Create incidents.
- Track recovery, replacement, and repair.
- Identify potentially affected future bookings.
- Coordinate risk flags and resolution.
- Request appropriate notifications.

### NotificationService

- Create booking confirmations and warnings.
- Record delivery attempts.
- Retry failed deliveries.
- Avoid duplicate messages where practical.

### AuditService

- Record important actions.
- Retrieve authorized audit history.
- Preserve actor, action, timestamp, and relevant entity details.

### SystemHealthService

- Provide a basic health check.
- Track operational errors.
- Report failed notification tasks.

## 9. Repository Layer

Repositories separate database access from business logic.

- `UserRepository` — users and account lookup.
- `ClubRepository` — clubs and status.
- `BookingRepository` — reservations, date queries, and booking history.
- `KitRepository` — kit and equipment retrieval.
- `InspectionRepository` — inspection records and item results.
- `IncidentRepository` — incidents and resolution history.
- `NotificationRepository` — messages and delivery attempts.
- `AuditRepository` — audit records.

Services should call repositories instead of scattering database queries throughout the application.

Use transactions for operations that update multiple related records. For example, a return inspection that discovers missing equipment must not leave the database partially updated if incident creation or status updates fail.

## 10. Database Design

Use SQLite for local development and PostgreSQL for production if needed.

Required tables:

- `users`
- `clubs`
- `kits`
- `equipment_items`
- `bookings`
- `inspection_records`
- `inspection_item_results`
- `incidents`
- `notifications`
- `notification_attempts` (optional but useful)
- `audit_logs`

Relationships:

- One club has many bookings.
- A booking belongs to one club and reserves one kit.
- A kit contains many equipment items.
- A booking has checkout and return inspection records.
- An inspection has multiple item results.
- An equipment item can have multiple incidents over time.
- A notification belongs to its recipient.
- An audit record references its actor and affected entity where possible.

Database requirements:

- Primary keys and foreign keys.
- Indexes for common booking, item, and incident queries.
- Validation of required fields and allowed values.
- Transactions for related writes.
- Concurrency protection for conflicting bookings.
- Consistent timestamp storage.
- Preservation of booking, inspection, incident, and audit history.
- No deletion of records needed for traceability.

A unique constraint on kit ID and start time alone does not prevent every overlapping reservation. Use a transactional conflict check and suitable locking or other database concurrency controls.

## 11. Technology Stack

Use this initial stack:

- **Language:** Python 3.12 or newer.
- **Backend:** FastAPI.
- **Database:** SQLite for development; PostgreSQL for production.
- **ORM:** SQLAlchemy.
- **Request validation:** Pydantic.
- **Authentication:** Secure password hashing with either session-based authentication or signed access tokens.
- **Frontend:** React with TypeScript, or HTML/CSS/JavaScript for a simpler UI.
- **Testing:** pytest and FastAPI TestClient.
- **API exploration:** FastAPI generated API documentation and Postman if needed.
- **Version control:** Git and GitHub.
- **Configuration:** Environment variables and an uncommitted local `.env` file.
- **Deployment:** A suitable Python application host and managed database if published online.

Keep the backend as a modular monolith. Do not introduce microservices, Redis, Kubernetes, or a message broker unless a genuine requirement justifies them.

## 12. Frontend Pages

### Public pages

- Login
- Registration
- Password recovery, if implemented

### Club Representative Dashboard

- Overview of upcoming bookings
- Availability calendar
- Create booking form
- My bookings
- Booking details and cancellation
- Notifications
- Booking history

### Organizer Dashboard

- Daily schedule and booking overview
- Kit readiness overview
- Equipment inventory
- Checkout inspection
- Return inspection
- Incident list and incident details
- Recovery or replacement actions
- Affected-booking list
- Notification delivery history
- Audit history

### Administrator Pages

- User management
- Role assignment
- Club management
- Booking policy settings, if configurable
- System health information

Every page must use the backend API for data and actions. Hiding a button in the frontend is not enough; the backend must enforce the same permissions.

## 13. API Design

Use consistent API routes, request schemas, response schemas, status codes, and error messages.

### Authentication and Users

- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/logout`
- `GET /api/users/me`
- `GET /api/admin/users`
- `PATCH /api/admin/users/{user_id}`

### Clubs

- `GET /api/clubs`
- `POST /api/admin/clubs`
- `PATCH /api/admin/clubs/{club_id}`

### Kit and Availability

- `GET /api/kit`
- `GET /api/kit/items`
- `GET /api/availability?date=...`

### Bookings

- `POST /api/bookings`
- `GET /api/bookings`
- `GET /api/bookings/{booking_id}`
- `POST /api/bookings/{booking_id}/cancel`
- `GET /api/clubs/{club_id}/bookings`

### Checkout, Return, and Inspection

- `POST /api/bookings/{booking_id}/checkout`
- `POST /api/bookings/{booking_id}/return`
- `GET /api/bookings/{booking_id}/inspections`

### Incidents and Notifications

- `GET /api/incidents`
- `POST /api/incidents`
- `POST /api/incidents/{incident_id}/recover`
- `POST /api/incidents/{incident_id}/replace`
- `POST /api/incidents/{incident_id}/resolve`
- `GET /api/notifications`
- `POST /api/notifications/{notification_id}/read`

### Audit and Health

- `GET /api/admin/audit-logs`
- `GET /api/health`

The backend must check roles and ownership for every protected route. Knowing a booking ID must not grant access to that booking.

Use appropriate HTTP status codes, including `401` for unauthenticated requests, `403` for forbidden actions, `404` for unavailable resources, `409` for booking conflicts, and `422` for validation errors where appropriate.

## 14. Main Workflows

### Workflow A — Create Booking

1. Club representative submits a booking request.
2. Backend verifies identity and club access.
3. BookingService validates the request.
4. AvailabilityService checks overlaps and the 15-minute gap.
5. Booking is saved atomically if valid.
6. A confirmation notification is queued.
7. Audit history records the booking action.
8. If invalid, the API returns a clear rejection reason.

### Workflow B — Checkout and Return

1. Organizer selects an eligible booking.
2. System checks organizer permissions and booking state.
3. Organizer records the condition of all three items.
4. System verifies that all required inspection results exist.
5. If all items are usable, checkout is committed and handover may proceed.
6. On return, the organizer inspects each item again.
7. System saves the return inspection and compares it with checkout.
8. If an item is missing or damaged, an incident is created and unsafe handover is blocked.
9. If all items are usable, return is completed and readiness is recalculated.

### Workflow C — Equipment Incident

1. Organizer reports missing or damaged equipment.
2. System records the incident and updates item status.
3. System finds future bookings potentially affected.
4. Affected bookings are marked at risk without automatically changing booking status.
5. Notifications are sent to affected clubs and organizers.
6. Organizer records recovery, replacement, or another approved resolution.
7. System verifies item condition and recalculates kit readiness.
8. Incident resolution is recorded in audit history.

### Workflow D — Failed Notification

1. Booking or incident transaction commits successfully.
2. A notification task is persisted.
3. Delivery is attempted.
4. Failed attempts are recorded and retried according to a bounded retry policy.
5. Notification failure does not roll back the booking or incident.
6. Duplicate notifications are avoided where practical.

For reliability, save the notification task in the database as part of the relevant transaction, then process it separately. A simple database-backed retry mechanism is sufficient for the first version.

## 15. Recommended GitHub Folder Structure

```text
college-event-kit-relay/
├── app/
│   ├── main.py
│   ├── config.py
│   ├── database.py
│   ├── models/
│   ├── schemas/
│   ├── services/
│   ├── repositories/
│   ├── api/
│   └── security/
├── frontend/
├── tests/
│   ├── test_authentication.py
│   ├── test_authorization.py
│   ├── test_booking.py
│   ├── test_availability.py
│   ├── test_concurrent_bookings.py
│   ├── test_inspections.py
│   ├── test_incidents.py
│   └── test_notifications.py
├── migrations/
├── .env.example
├── .gitignore
├── requirements.txt
├── README.md
└── LICENSE
```

The folder structure may be expanded as implementation progresses. Keep responsibilities separated and avoid creating empty modules that are never used.

## 16. Automated Testing Requirements

### Booking tests

- A valid booking is created.
- Overlapping bookings are rejected.
- A booking starting 14 minutes after the previous booking is rejected.
- A booking starting exactly 15 minutes later passes the gap check.
- Cancelled bookings no longer block availability.
- Inactive clubs cannot create bookings.
- Invalid and past times are rejected.
- Concurrent conflicting requests cannot both succeed.

### Authentication and Authorization tests

- Valid credentials allow login.
- Invalid credentials are rejected.
- Unauthenticated users cannot access protected endpoints.
- Club representatives cannot view another club's private booking.
- Club representatives cannot perform checkout or resolve incidents.
- Only authorized administrators can manage roles.

### Inspection and Incident tests

- Checkout is blocked if required equipment is missing.
- Checkout is blocked if a required item is unusable.
- An inspection cannot be completed while required results are missing.
- Return inspection detects missing or damaged items.
- Incidents mark affected future bookings at risk.
- Recovery does not mark the kit ready if another item remains unusable.
- Cancelled bookings cannot be checked out.

### Notification and Audit tests

- Booking confirmation is recorded after successful booking creation.
- Notification failure does not undo a committed booking.
- Retrying does not unnecessarily duplicate notifications.
- Important user and admin actions create audit records.

## 17. GitHub Requirements

Include:

- `README.md` with project description, features, architecture, installation, configuration, database setup, startup commands, test commands, and screenshots where useful.
- `requirements.txt` for Python dependencies.
- `.env.example` with safe placeholders, never real secrets.
- `.gitignore` excluding `.env`, virtual environments, caches, logs, and generated files.
- Database migrations if using Alembic.
- Automated tests.
- Clear commit messages.
- A license if the project will be distributed.

Never push real passwords, signing secrets, API keys, personal information, or production database credentials to GitHub.

## 18. Implementation Plan

Build and verify the project in stages:

1. Inspect the workspace and present an implementation plan.
2. Create the backend foundation, models, database, and migrations.
3. Implement authentication and authorization.
4. Implement availability and booking logic.
5. Implement checkout, return, and inspections.
6. Implement incidents, disruptions, notifications, and audit history.
7. Implement and connect the frontend.
8. Add automated tests and run them.
9. Fix errors and verify all workflows.
10. Update the README with accurate setup instructions.

After each stage, summarize what was implemented and run relevant tests. Do not claim a feature is complete unless it has been implemented and verified.

If a detail is unspecified, choose the simplest secure solution that satisfies the requirements and document the choice. Do not add unnecessary features or design patterns without a clear purpose.

## 19. Definition of Done

The first release is complete when:

- Club representatives can register, log in, view availability, and create valid bookings.
- Booking conflicts and the 15-minute gap are enforced by the backend.
- Concurrent requests cannot create conflicting reservations.
- Organizers can inspect, check out, and receive returned equipment.
- All three equipment items have individually recorded inspection results.
- Missing or damaged equipment creates a trackable incident.
- Affected future bookings are flagged and relevant users are notified.
- Booking, equipment, and incident states remain consistent.
- Role-based access is implemented and tested.
- Audit history is preserved.
- Automated tests pass.
- The README explains how to run the application from a fresh clone.

**Start by inspecting the workspace and presenting the implementation plan. Then implement the project stage by stage.**
