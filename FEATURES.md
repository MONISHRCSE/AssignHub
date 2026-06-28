# AssignHub — Implemented Features

> **Last Updated:** 26 June 2026  
> **Tech Stack:** Flutter (Frontend) • FastAPI + PostgreSQL (Backend) • AWS S3 (File Storage) • Firebase Cloud Messaging (Push Notifications) • Groq LLM (AI Query Engine)

---

## 1. Authentication & Security

### 1.1 Admin Registration & OTP Verification
- Admin signs up with full name, email, and password.
- A one-time password (OTP) is sent to the admin's email via Gmail SMTP.
- Admin verifies the OTP to activate their account.
- Upon verification, JWT access token and refresh token are issued.

### 1.2 Secure Login (All Roles)
- Unified login endpoint for Admin, Mentor, and Student roles.
- Students and Mentors must provide a unique Registration ID alongside email and password.
- Passwords are hashed using bcrypt before storage.
- On login, FCM device token is registered for push notification delivery.

### 1.3 Persistent Authentication
- JWT access tokens (60-minute expiry) and refresh tokens (7-day expiry) are stored securely in Flutter's `SharedPreferences`.
- On app launch, the Splash Screen attempts token restoration via the `/auth/me` endpoint.
- If the token is valid, the user is auto-redirected to their dashboard (no re-login needed).
- If the token is expired, a silent refresh is attempted using the refresh token.
- If both tokens are invalid, the user is redirected to the Login screen.

### 1.4 Logout
- Logout revokes the refresh token on the backend and clears all local credentials.

---

## 2. Role-Based Access Control (RBAC)

| Role | Capabilities |
|------|-------------|
| **Admin** | Create classes, provision mentors/students, manage approvals, view all analytics, AI query, exports |
| **Mentor** | Manage assigned class(es), create/publish assignments, approve/reject students, view tracker & analytics |
| **Student** | View assigned assignments, submit work, view personal analytics, set custom reminders |

- Every API endpoint is guarded by `require_role()` middleware that validates the JWT and checks the user's role.
- Class-level access is further enforced via `verify_admin_class_access()` and `verify_mentor_class_access()`.

---

## 3. Class Management

### 3.1 Create Class (Admin)
- Admin creates a class with a name, description, and academic year.
- Class analytics record is auto-initialized upon creation.

### 3.2 View Classes
- Admin sees all classes they own with student count, mentor count, and status.
- Mentors see only classes they are assigned to via the "My Classes" endpoint.

### 3.3 Edit / Archive Class (Admin)
- Admin can update class name, description, and status.
- When a class is archived, all active members receive a `CLASS_ARCHIVED` notification.

### 3.4 Class Detail View
- Shows class metadata, student count, assignment count, primary mentor, and co-mentors.

---

## 4. User Provisioning

### 4.1 Manual Mentor Creation (Admin)
- Admin manually creates a mentor account with name, email, password, and class assignment.
- A unique Registration ID (e.g., `MENTOR-KFRZCKXJ`) is auto-generated.
- Mentor receives an invitation email via Brevo SMTP with their credentials.

### 4.2 Manual Student Creation (Admin / Mentor)
- Admin or Mentor manually creates a student with name, email, password, and Registration ID.
- Student is placed in `PENDING` approval status.
- Student receives an invitation email with their credentials.

### 4.3 Add Co-Mentor (Admin)
- Admin can add co-mentors to any class.
- Co-mentor account is auto-created with a random password and Registration ID.
- Invitation email is sent; co-mentor receives an in-app `CO_MENTOR_ADDED` notification.

### 4.4 Bulk Import via Excel (Admin)
- Admin downloads a pre-formatted Excel template (Classes, Mentors, Students sheets).
- Admin uploads the filled Excel file.
- Backend processes the import asynchronously in the background.
- Batch status (UPLOADED → PROCESSING → COMPLETED/FAILED) and per-row errors are trackable.
- Successfully imported users receive invitation emails automatically.

---

## 5. Student Approval Workflow

### 5.1 Pending Approvals List (Admin / Mentor)
- Admin and Mentor see a list of students with `PENDING` membership status.
- Each entry shows student name, email, Registration ID, request timestamp, and join method.

### 5.2 Approve Student
- Approving a student activates their class membership.
- Student analytics record is auto-initialized.
- Student receives:
  - In-app notification: `"Access Granted — You can now log in to {Class Name}"`
  - FCM push notification (works even if app is closed).

### 5.3 Reject Student
- Rejecting a student sets their status to `REJECTED` with a reason.
- Student receives:
  - In-app notification: `"Access Denied — {reason}"`
  - FCM push notification.

### 5.4 Login Guard
- Only students with `ACTIVE` membership can log in and access assignments.
- Students with `PENDING`, `REJECTED`, `BLOCKED`, or `INACTIVE` status are shown appropriate error messages on login.

---

## 6. Assignment Management

### 6.1 Create Assignment (Admin / Mentor)
- Supports three content types:
  - **PDF** — Upload a PDF file to AWS S3 and attach the URL.
  - **Link** — Provide an external URL.
  - **Rich Text** — Enter formatted text directly in the app.
- Supports two submission types:
  - **File Upload** — Students upload a file.
  - **Text Input** — Students type their answer.
- Optional deadline with date and time picker.
- Assignment is saved as `DRAFT` initially.

### 6.2 Publish Assignment
- Publishing changes the status from `DRAFT` to `PUBLISHED`.
- Assignment analytics record is auto-initialized (total targets = active student count).
- All active students in the class receive:
  - In-app notification: `"New Assignment — {title}"`
  - FCM push notification with deadline info.
- If a deadline is set, the following scheduled jobs are auto-created:
  - **24-hour reminder** — Sent to students who haven't submitted yet.
  - **2-hour final reminder** — Sent to students who haven't submitted yet.
  - **Auto-close job** — Automatically closes the assignment at the deadline.

### 6.3 Close Assignment (Admin / Mentor)
- Manually closes an assignment (or auto-closed at deadline).
- On close:
  - All analytics are recomputed (student-level, class-level, assignment-level).
  - Students who didn't submit receive a `MISSED_DEADLINE` notification + FCM push.
  - WebSocket broadcast updates the real-time tracker for connected mentors.

### 6.4 View Assignments
- Students see only `PUBLISHED` and `CLOSED` assignments.
- Mentors/Admin see all assignments including `DRAFT`.
- Each assignment shows creator name, content type, submission type, deadline, and status.

### 6.5 Assignment Detail (Student View)
- Shows full assignment info, content (PDF viewer / link / rich text), and deadline countdown.
- Shows the student's current submission status (submitted, late, pending).
- Allows setting a custom reminder alarm.

---

## 7. Submission System

### 7.1 Submit Assignment (Student)
- Students can submit via file upload (AWS S3) or text input.
- System automatically detects if the submission is late (submitted after deadline).
- Submission versioning is supported — students can resubmit; previous versions are preserved.
- On submission:
  - Assignment analytics (submitted count, late count, completion rate) are updated in real-time.
  - Student receives a `SUBMISSION_RECEIPT` notification with exact timestamp.
  - WebSocket broadcast pushes the new submission to the mentor's live tracker.

### 7.2 View Submissions (Mentor / Admin)
- Full submission list for each assignment with student details.
- Each submission shows: student name, email, Registration ID, submission type, file URL or text answer, submission timestamp, late status, and version number.

### 7.3 My Submissions (Student)
- Students can view all their past submissions across assignments with status indicators (Submitted, Late, Missed, Pending).

---

## 8. Real-Time Submission Tracker

### 8.1 Live Tracker Dashboard (Mentor / Admin)
- Real-time tracker showing every student's submission status for a specific assignment.
- Status categories: `SUBMITTED` (green), `LATE` (orange), `MISSED` (red), `PENDING` (grey).
- Summary counts: Total Submitted, Pending, Late, Missed.
- Completion percentage displayed prominently.

### 8.2 WebSocket Real-Time Updates
- Mentors connect via WebSocket to the assignment tracker.
- When a student submits, the tracker auto-updates without page refresh.
- When an assignment auto-closes at deadline, the tracker instantly reflects the final state.
- Automatic reconnection with exponential backoff (2s, 4s, 8s — max 3 attempts).

---

## 9. Push Notifications (Firebase Cloud Messaging)

### 9.1 System Tray Notifications (Even When App is Closed)
- Firebase Cloud Messaging (FCM) is fully integrated:
  - `google-services.json` configured for Android.
  - `firebase_core` and `firebase_messaging` Flutter packages installed.
  - Background message handler registered with `@pragma('vm:entry-point')`.
  - Backend Service Account credentials configured in `.env`.

### 9.2 Notification Events
| Event | Who Receives | Title | Body |
|-------|-------------|-------|------|
| Student approved | Student | Access Granted | You can now log in to {Class} |
| Student rejected | Student | Access Denied | {Reason} |
| New assignment published | All students in class | New Assignment | {Title} — due {deadline} |
| 24-hour deadline reminder | Students who haven't submitted | Reminder | {Title} due tomorrow |
| 2-hour deadline reminder | Students who haven't submitted | Final Reminder | {Title} due in 2 hours |
| Custom student reminder | The student who set it | Reminder | Don't forget: {Title} |
| Missed deadline | Students who didn't submit | Missed Deadline | You missed {Title} |
| Submission receipt | The student who submitted | Submitted | Assignment {Title} submitted at {time} |
| Class archived | All class members | Class Archived | {Class} has been archived |
| Co-mentor added | The new co-mentor | Added as Co-Mentor | You were added to {Class} |

### 9.3 In-App Notification Center
- Notification bell icon on the student dashboard with unread count badge.
- "Recent Alerts" section shows all notifications (ordered by latest first).
- Mark individual notifications as read or mark all as read.
- Background polling every 60 seconds for new notifications (silent — no page flicker).
- Live in-app alarm banner (Snackbar) pops up when a new notification arrives while the dashboard is open.

---

## 10. Custom Reminder Alarms (Student)

- Students can set a custom reminder for any published assignment.
- The student picks a future date and time for the reminder.
- The reminder is scheduled as a background job on the server (APScheduler with PostgreSQL jobstore).
- When the scheduled time arrives:
  - Backend creates a `DEADLINE_REMINDER` notification in the database.
  - FCM push notification is sent to the student's device (works even when app is closed).
  - If the student is on the dashboard, an in-app alarm banner appears automatically.

---

## 11. Analytics & Dashboards

### 11.1 Student Analytics
- **Per-student metrics:** Total assigned, total submitted, total missed, total late, completion rate (%).
- **Streaks:** Current submission streak and longest submission streak.
- **Risk level:** Automatically computed — `NORMAL`, `MEDIUM`, `HIGH`, or `CRITICAL` based on consecutive misses and completion rate.
- **Average submission delay:** Average hours between deadline and submission time.
- **Assignment history:** Full history of every assignment with individual status (Submitted / Late / Missed / Pending).
- **Class average comparison:** Student's completion rate compared to the class average.

### 11.2 Class Analytics
- **Class-level metrics:** Total students, average completion rate, total assignments.
- **Risk distribution:** Count of students at each risk level (Normal / Medium / High / Critical).
- **Assignment breakdown:** Per-assignment completion rates with bottleneck detection.
- **Bottleneck assignments:** Assignments with < 50% completion rate are flagged automatically.

### 11.3 Admin Overview Dashboard
- Cross-class analytics aggregation.
- Total classes, total students, total assignments, overall completion rate.
- Per-class summary cards with student counts and completion rates.
- At-risk student highlights.

### 11.4 Mentor Dashboard
- Class-level stats: Active students, total assignments, pending approvals.
- Recent assignments with quick actions (view tracker, view submissions).
- At-risk students highlighted with risk level badges.

### 11.5 Student Dashboard
- Personal completion rate and risk level display.
- Upcoming assignments list with deadlines and countdown timers.
- Recent alerts section with notification history.

---

## 12. AI Query Engine (Admin / Mentor)

- Natural language query interface for class analytics.
- Powered by Groq LLM (Llama model).
- Example queries:
  - "Who are the at-risk students in this class?"
  - "Which assignment has the lowest completion rate?"
  - "Show me students who have missed more than 2 assignments."
- AI processes the query against real database records and returns structured, contextual responses.
- Query history is maintained for reference.

---

## 13. Export System (Admin / Mentor)

### 13.1 Assignment Tracker Export
- Export the full submission tracker for any published/closed assignment.
- Export is generated asynchronously in the background.
- Output is an Excel file uploaded to AWS S3.
- Export job status is trackable (PENDING → PROCESSING → COMPLETED).
- Download URL is provided once the export is ready.

---

## 14. File Storage (AWS S3)

- All file uploads (assignment PDFs, student submission files, exports, bulk import templates) are stored in AWS S3.
- Pre-signed upload URLs generated for secure direct-to-S3 uploads from the Flutter app.
- Pre-signed download URLs generated for secure time-limited file access.
- Bucket: `assignhub-files-praveen` in `eu-north-1` region.

---

## 15. Email System

### 15.1 Admin OTP Emails
- Sent via Gmail SMTP (TLS, port 587).
- Used for admin account verification during signup.

### 15.2 Invitation Emails
- Sent via Brevo SMTP relay.
- Used when admins provision mentors or students (manual or bulk import).
- Email includes: welcome message, login credentials, Registration ID, and class name.

---

## 16. Background Job Scheduler

- **APScheduler** with PostgreSQL-backed jobstore for persistence across server restarts.
- Scheduled jobs:
  - **Auto-close assignment** at deadline.
  - **24-hour deadline reminder** to students with pending submissions.
  - **2-hour final deadline reminder** to students with pending submissions.
  - **Custom student reminders** at user-specified times.
- All jobs survive server restarts (stored in the database, not in memory).

---

## 17. Deployment & Infrastructure

| Component | Service |
|-----------|---------|
| Backend API | FastAPI on Render / local server |
| Database | PostgreSQL on Render |
| File Storage | AWS S3 (eu-north-1) |
| Push Notifications | Firebase Cloud Messaging |
| Email (OTP) | Gmail SMTP |
| Email (Invitations) | Brevo SMTP Relay |
| AI Engine | Groq Cloud API (Llama) |
| Frontend | Flutter (Android) |

---

## API Endpoints Summary

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/admin/signup` | Admin registration |
| POST | `/auth/admin/verify-otp` | OTP verification |
| POST | `/auth/login` | Login (all roles) |
| POST | `/auth/refresh` | Token refresh |
| POST | `/auth/logout` | Logout |
| GET | `/auth/me` | Current user info |
| POST | `/classes` | Create class |
| GET | `/classes` | List admin's classes |
| GET | `/classes/my-classes` | List mentor's classes |
| GET | `/classes/{id}` | Class detail |
| PATCH | `/classes/{id}` | Update class |
| GET | `/classes/{id}/students` | List class students |
| GET | `/classes/{id}/approvals` | Pending approvals |
| PATCH | `/classes/{id}/students/{sid}/approve` | Approve student |
| PATCH | `/classes/{id}/students/{sid}/reject` | Reject student |
| POST | `/classes/{id}/co-mentors` | Add co-mentor |
| POST | `/assignments` | Create assignment |
| GET | `/assignments?class_id=` | List assignments |
| GET | `/assignments/{id}` | Assignment detail |
| POST | `/assignments/{id}/publish` | Publish assignment |
| POST | `/assignments/{id}/close` | Close assignment |
| GET | `/assignments/{id}/tracker` | Submission tracker |
| POST | `/submissions/assignments/{id}/submit` | Submit assignment |
| GET | `/submissions/assignments/{id}` | View submissions |
| GET | `/submissions/students/{id}` | Student's submissions |
| GET | `/analytics/students/{id}` | Student analytics |
| GET | `/analytics/classes/{id}` | Class analytics |
| GET | `/analytics/admin/overview` | Admin overview |
| GET | `/notifications` | List notifications |
| PATCH | `/notifications/read-all` | Mark all read |
| PATCH | `/notifications/{id}/read` | Mark one read |
| POST | `/notifications/reminder` | Set custom reminder |
| POST | `/ai/query` | AI natural language query |
| POST | `/exports/assignment-tracker` | Export tracker |
| GET | `/exports/{id}` | Export job status |
| POST | `/provisioning/manual/mentor` | Create mentor |
| POST | `/provisioning/manual/student` | Create student |
| GET | `/provisioning/bulk-import/template` | Download template |
| POST | `/provisioning/bulk-import` | Upload bulk import |
| GET | `/provisioning/bulk-import/{id}` | Import batch status |
| POST | `/storage/presigned-upload` | Get S3 upload URL |
| WS | `/ws/tracker/{id}` | Real-time tracker WebSocket |
