# Requirements Document

## 1. Application Overview

### 1.1 Application Name
Khulafasco Student Enrollment & Fee Management System (Al-khulafau Arashiduun Islamic Senior High School)

### 1.2 Application Description
This document specifies an **enhancement pass** for the existing system (/workspace/app-ecmnb1zdgzcx, Supabase project ref: uscveouabdtcyxlitiyo). This is an audit, completion, and integration task for a live system, not a new project. All M1/M2 completed functionality must be preserved.

Tech stack: React, Vite, TypeScript, Tailwind CSS, shadcn/ui, Supabase, Vercel.

Brand colors: primary maroon #6B1A2A, deep maroon #3D0D17, rose gold #C4956A, cream #F5E6D3, background #FAFAF8; existing school logo and design system.

Currency: GH₵ (Ghana Cedi).

System UI language: English.

**Robustness Enhancement Scope**: This pass adds 7 identified robustness features (password lifecycle, staff lifecycle, student status workflow, error boundary, dashboard KPIs verification, deployment configuration, session enforcement) to address security, operational, and resilience gaps.

## 2. Users and Use Cases

### 2.1 Target Users

**SYSTEM ADMINISTRATOR**
- Access dashboard
- Student management/enrollment (search/filter/detail)
- Edit enrolled student records
- Update student enrollment status (graduate/withdraw/reactivate) with audit
- Academic year management
- Program management
- House management
- Fee type management
- Staff/user management (create, toggle active/inactive, reset password)
- Finance overview
- Audit log viewing
- Student photo upload/update/delete
- Payment reversal
- Export student list and outstanding balances reports
- Change own password

**FINANCE OFFICER**
- Access dashboard
- Search students by JHS/BECE index number
- Set amount due
- Create/update fee charges
- Record payments with manual allocations (full form: payment method, date, reference, notes)
- View payment history
- Generate receipts
- View outstanding balances
- Financial reconciliation
- Export student list and outstanding balances reports
- Change own password
- No permissions: staff management, enrollment creation/editing, system configuration, payment reversal

**STUDENT**
- Managed as database records only
- No student authentication portal in this phase

### 2.2 Core Use Cases

- Admin creates student enrollment records, uploads photos, manages academic years/programs/houses/fee types
- Admin edits enrolled student records (personal, guardian, academic, enrollment status)
- Admin updates student enrollment status (graduate/withdraw/reactivate) with audit log
- Finance officer sets amount due for students, creates fee charges, records payments with manual allocations (including payment method, date, reference, notes), generates receipts, views financial status
- Admin reverses completed payments with reason
- Admin and finance officer export student list (respecting filters) and outstanding balances report as CSV
- System provides database-backed dashboard statistics, student search/filter, financial reconciliation, audit logs
- Admin creates staff accounts with temporary passwords, toggles staff active/inactive status, resets staff passwords
- All logged-in users (admin and finance officer) change their own passwords
- System enforces session checks for active accounts only
- System uses React ErrorBoundary to handle render crashes gracefully

## 3. Page Structure and Functionality

### 3.1 Page Structure Tree

```
├── Login
├── Dashboard
│   ├── Admin Dashboard
│   └── Finance Officer Dashboard
├── Students
│   ├── Student List
│   ├── Student Detail (with status update UI for admin)
│   ├── Student Enrollment (Admin only)
│   └── Edit Student (Admin only)
├── Finance
│   ├── Set Amount Due
│   ├── Fee Charge Management
│   ├── Record Payment (enhanced form)
│   ├── Payment History
│   ├── Receipt Detail (with reversal option for admin)
│   └── Financial Reconciliation
├── Reports
│   ├── Export Student List (CSV)
│   └── Export Outstanding Balances (CSV)
├── Academic Years (Admin only)
├── Programs (Admin only)
├── Houses (Admin only)
├── Fee Types (Admin only)
├── Staff (Admin only, with active/inactive toggle and password reset)
├── Change Password (all logged-in users)
├── Unauthorized
└── Error/Not Found
```

### 3.2 Login Page

- User enters credentials and logs in
- Verify session and retrieve role
- Check profile.is_active status; if false, sign out and display error
- Redirect to role-specific dashboard on success
- Logout functionality

### 3.3 Dashboard

**Admin Dashboard**
- Display total students, active students, boarding students, day students (database query)
- Display current academic year (from database)
- Display program distribution, house distribution (database statistics)
- Display finance overview: amount due (current year total billed), collected (current year total paid), outstanding balance (current year total outstanding)
- Display recent payments list (last 5-10 payments with date, student, amount)
- No hardcoded or mock data

**Finance Officer Dashboard**
- Display finance-related statistics: amount due (current year total billed), collected (current year total paid), outstanding balance (current year total outstanding)
- Display recent payment records (last 5-10 payments with date, student, amount)
- Quick student search entry

### 3.4 Student Management

**Student List**
- Server-side/database-backed search (by JHS/BECE index number, name)
- Pagination
- Filters: academic year, program, house, student type (boarding/day), enrollment status
- Display student basic info: index number, name, program, house, student type, enrollment status
- Click to enter student detail page
- Export current filtered list as CSV

**Student Detail**
- Personal info: index number, name, gender, date of birth, photo, previous school, region, district
- Guardian info: name, relationship, phone, alternate phone, email, address
- Academic info: program, house, student type, academic year, enrollment status, enrollment date
- Financial info: amount due, total charges, paid, outstanding balance, financial status
- Handle loading, empty data, error, not found states
- Admin can upload/update/delete student photo
- Admin can navigate to edit student page
- **Admin can update student enrollment status** (active/graduated/withdrawn) with optional reason/notes, system records audit log (action: student_status_updated)

**Student Enrollment (Admin only)**
- Input student info: JHS/BECE index number (required, trim spaces, uppercase, uniqueness validation), name, gender, date of birth, previous school, region, district, guardian info, program, house, student type (boarding/day stored lowercase), academic year
- Upload student photo (optional)
- Backend authorization check
- Reject duplicate index numbers
- Record audit log on success
- Do not auto-create fee charges

**Edit Student (Admin only)**
- Admin can edit enrolled student's personal info (name, gender, date of birth, previous school, region, district), guardian info (name, relationship, phone, alternate phone, email, address), academic info (program, house, student type, academic year, enrollment status)
- Cannot edit JHS/BECE index number (immutable identifier)
- Backend RPC `update_student` with authorization check (admin only), input validation, audit logging (action: student_updated)
- Handle loading, validation errors, authorization errors, database errors

**Student Photo Management**
- Photos stored in private `student-photos` bucket
- Database field `photo_path` records path
- No permanent public URLs
- Photo proxy route (src/app/api/student-photo/[jhs_index_number]/route.ts) must verify user identity and authorization (Admin or Finance only) before fetching from Storage

### 3.5 Finance Module

**Set Amount Due**
- Finance officer or admin selects student
- Input amount due (numeric, non-negative, null = not set)
- Backend validation and authorization
- Use existing function `set_student_amount_due(...)`
- Record audit log

**Fee Charge Management**
- Select student, fee type, input amount, optional description
- Use existing function `set_student_fee_charge(...)` (student + fee type + current academic year + amount + description + creator)
- Backend validation and authorization
- Record audit log

**Record Payment (enhanced)**
- Finance officer searches student by JHS/BECE index number
- View student financial summary (amount due, total charges, paid, outstanding balance)
- Input payment amount
- **Required fields**:
  - Payment method: dropdown (cash, bank transfer, mobile money, cheque, other)
  - Payment date: date picker (default today, editable)
  - Reference/transaction number: text input (optional)
  - Notes: textarea (optional)
- Validate payment amount does not exceed outstanding balance
- Manual allocation to specific charges:
  - Allocation rows show charge name, year, amount, paid so far, remaining balance
  - Quick helpers: \"Allocate to remaining balance\" and \"Clear allocations\"
  - Running total must equal payment amount
  - Single allocation cannot exceed charge remaining balance
  - All allocations must belong to same student
  - All allocation references must be valid
- Backend atomically records payment and allocations
- Display success only after database confirmation (not just frontend request sent)
- Generate receipt number (format KHA-YYYY-000001, using `payment_receipt_seq` sequence)
- Display receipt detail and print option
- Record audit log

**Payment History**
- Display receipt number, date, amount, student, allocation details, payment method, recorded by, status, notes
- Support filters
- Role-based access control

**Receipt Detail**
- Display receipt number, date, student info, payment amount, allocation details, payment method, reference number, notes, recorded by
- Print-friendly format
- Database-backed
- **Admin only: Payment reversal option**
  - Button to reverse payment (if status is completed)
  - Input reversal reason (required)
  - Backend RPC atomically: set payment status to 'reversed', restore allocation balances in financial summaries, record audit log (action: payment_reversed)
  - Prevent reversal of already-reversed payments
  - Authorization check (admin only)

**Financial Reconciliation**
- Reconciliation model: amount due, total charges, paid, outstanding balance (max(amount due - completed payments total, 0)), charge difference
- Financial status: amount_due_not_set / balanced / charges_below_total_due / charges_above_total_due
- Database as source of truth
- Payment status: not_set / unpaid / partially_paid / paid, derived from backend authoritative data

### 3.6 Reports

**Export Student List (CSV)**
- Export current filtered student list as CSV
- Columns: JHS/BECE index number, name, gender, program, house, student type, enrollment status, academic year
- Client-side generation from database-backed data
- Respects current filters on student list page
- Available to admin and finance officer

**Export Outstanding Balances (CSV)**
- Export financial reconciliation report as CSV
- Columns: JHS/BECE index number, name, program, house, student type, amount due, total charges, paid, outstanding balance, financial status
- Client-side generation from database-backed data
- Available to admin and finance officer

### 3.7 Academic Year Management (Admin only)

- Create, read, update, delete academic years
- Activate academic year (safe activation, prevent multiple current years)
- Current academic year determined from database (expected 2027/2028 but database is authoritative, no hardcoding)

### 3.8 Program Management (Admin only)

- Manage programs: General Arts, General Science, Business, Home Economics, General Agric
- Database-backed CRUD
- Foreign key references
- Pre-insert validation with live data (no duplicates)

### 3.9 House Management (Admin only)

- Manage houses: Abubakar, Umar, Uthman, Ali
- Database-backed CRUD
- Foreign key references
- Pre-insert validation with live data (no duplicates)

### 3.10 Fee Type Management (Admin only)

- Manage fee types: Admission Form, Hostel Fee, Maintenance Fee, Madrasat Fee
- Database-backed CRUD
- Foreign key references
- Pre-insert validation with live data (no duplicates)
- No hardcoded boarding/day amounts (guideline GH₵750+ boarding / GH₵200+ day for reference only); finance officer inputs actual amounts; NULL amount due displays as \"not set\" (not zero)

### 3.11 Staff Management (Admin only)

- Manage staff/user accounts
- Assign roles (Admin/Finance Officer)
- **Toggle staff active/inactive status**: admin can activate or deactivate staff accounts, system updates profile.is_active field, records audit log (action: staff_toggled_active or staff_toggled_inactive), inactive staff cannot sign in
- **Reset staff password**: admin can reset staff password to new temporary password, system uses backend RPC `reset_staff_password` (admin-only, generates bcrypt temp password, updates auth.users + auth.identities + token columns, records audit log action: staff_password_reset), staff must change password on next login
- Backend authorization check
- Record audit log

### 3.12 Change Password (all logged-in users)

- All logged-in users (admin and finance officer) can change their own password
- User inputs current password, new password, confirm new password
- Frontend validates: new password meets strength requirements, new password matches confirm password
- Backend uses Supabase `updateUser` with old password verification
- On success, display success message and optionally sign out user to re-login with new password
- On failure (wrong current password, weak new password), display error
- Record audit log (action: password_changed)

### 3.13 Unauthorized Page

- Display when user attempts to access unauthorized page
- Provide link to return to dashboard

### 3.14 Error/Not Found Page

- Handle 404, 500, etc.
- Provide friendly error message
- Provide link to return to home

### 3.15 Error Boundary (React ErrorBoundary)

- Wrap entire application with React ErrorBoundary component
- Catch render crashes and display fallback UI with error message and retry/recover options
- Log error details for debugging
- Prevent entire app whitescreen on component render failure

## 4. Business Rules and Logic

### 4.1 Student Identifier Rules

- Authoritative student identifier: JHS/BECE index number (database field `jhs_index_number`)
- Do not use generated alternative IDs
- Normalization: trim spaces, uppercase, required
- Remove incorrect references to old `index_number` identifier

### 4.2 Student Type Rules

- Student type stored lowercase: `boarding`, `day`
- Do not use `Boarding`/`Day` to query database

### 4.3 Enrollment Workflow

- Admin only can create enrollment
- Backend authorization check
- Reject duplicate index numbers
- Record audit event
- Support photo upload
- Do not auto-create fee charges

### 4.4 Student Editing Workflow

- Admin only can edit enrolled student records
- Cannot edit JHS/BECE index number (immutable)
- Backend RPC `update_student` with authorization check, input validation, audit logging (action: student_updated)
- Editable fields: personal info (name, gender, date of birth, previous school, region, district), guardian info (name, relationship, phone, alternate phone, email, address), academic info (program, house, student type, academic year, enrollment status)

### 4.5 Student Status Workflow

- Admin can update student enrollment status (active/graduated/withdrawn) from student detail page
- Optional reason/notes field for status change
- Backend records audit log (action: student_status_updated, description includes old status, new status, reason)
- Status change does not affect financial records (outstanding balances remain)

### 4.6 Finance Rules

- Amount due: numeric, non-negative, null = not set (not zero)
- Fee charges: student + fee type + current academic year + amount + description + creator
- Payment allocations: allocation total = payment amount; single allocation ≤ charge remaining balance; same student; valid references
- Receipt number: use `payment_receipt_seq` sequence, format KHA-YYYY-000001, unique, generated only after payment confirmation
- Reconciliation model: amount due, total charges, paid, outstanding balance, charge difference, financial status
- Payment status derived: not_set / unpaid / partially_paid / paid
- **Payment method, date, reference, notes**: required fields for complete payment recording

### 4.7 Payment Reversal Rules

- Admin only can reverse completed payments
- Input reversal reason (required)
- Backend RPC atomically: set payment status to 'reversed', restore allocation balances in financial summaries, record audit log (action: payment_reversed)
- Prevent reversal of already-reversed payments
- Authorization check (admin only)

### 4.8 Academic Year Rules

- Admin CRUD academic years
- Safe activation (prevent multiple current years)
- Current academic year determined from database (database is authoritative)

### 4.9 Data Integrity Rules

- Programs, houses, fee types: database-backed, foreign key references, pre-insert validation, no duplicates

### 4.10 Photo Access Rules

- Photos stored in private bucket
- Photo proxy route must verify identity and authorization (Admin or Finance only)
- No permanent public URLs

### 4.11 Role-Based Access Control (complete audit)

**Finance Officer CAN:**
- View dashboard
- Search/view students
- Set amount due
- Create/update fee charges
- Record payments (with full form: method, date, reference, notes, allocations)
- View payment history
- Generate receipts
- View financial reconciliation
- Export student list and outstanding balances reports
- Change own password

**Finance Officer CANNOT:**
- Enroll students
- Edit student records
- Update student enrollment status
- Manage staff
- Manage configuration (academic years, programs, houses, fee types)
- Reverse payments
- Toggle staff active/inactive
- Reset staff passwords

**Admin CAN:**
- All finance officer capabilities
- Enroll students
- Edit student records
- Update student enrollment status
- Manage staff (create, toggle active/inactive, reset passwords)
- Manage configuration
- Reverse payments

**Enforcement:**
- All enforced server-side via RLS policies and SECURITY DEFINER RPCs
- Frontend UI hides unauthorized actions but backend is authoritative

### 4.12 Staff Lifecycle Rules

- Admin creates staff accounts with temporary passwords (bcrypt hashed, auth.identities + token columns fully populated)
- Admin can toggle staff active/inactive status (updates profile.is_active field, records audit log)
- Inactive staff cannot sign in (session check enforces is_active = true)
- Admin can reset staff password to new temporary password (backend RPC `reset_staff_password`, records audit log)
- Staff must change password on first login or after password reset

### 4.13 Password Lifecycle Rules

- All logged-in users can change their own password via Change Password page
- Frontend validates new password strength and confirmation match
- Backend uses Supabase `updateUser` with old password verification
- On success, record audit log (action: password_changed)
- On failure, display error (wrong current password, weak new password)

### 4.14 Session Enforcement Rules

- AuthContext checks profile.is_active on session load
- If is_active = false, sign out user and display error message
- Login page checks is_active after authentication, rejects inactive accounts
- All protected routes verify active session

### 4.15 Dashboard KPI Rules

- Admin dashboard displays: total students (count all), active students (count enrollment_status = 'active'), boarding students (count student_type = 'boarding'), day students (count student_type = 'day'), current academic year (from database), program distribution (count by program), house distribution (count by house), finance overview (amount due = current year total billed, collected = current year total paid, outstanding balance = current year total outstanding), recent payments list (last 5-10 payments with date, student, amount)
- Finance officer dashboard displays: finance overview (amount due, collected, outstanding balance for current year), recent payments list (last 5-10 payments with date, student, amount)
- All KPIs database-backed, no hardcoded or mock data

## 5. Database Schema and Migrations

### 5.1 Migration Files Requirement

- Create complete, ordered, re-runnable-from-scratch SQL migration files in repository (e.g., supabase/migrations/*.sql)
- Migrations must cover ENTIRE current schema:
  - Enums (user_role, enrollment_status, payment_status, audit_action, student_type, payment_method)
  - Tables (profiles with is_active field, students, academic_years, programs, houses, fee_types, fee_configurations, student_charges, payments with payment_method/payment_date/reference_number/notes/reversal_reason/reversed_at, payment_allocations, audit_logs)
  - Indexes
  - Constraints (primary keys, foreign keys, unique constraints, check constraints)
  - Triggers (set_updated_at, handle_new_user)
  - SECURITY DEFINER functions (get_my_role, is_admin, is_finance_officer, is_staff, set_updated_at, handle_new_user, enroll_student, search_students, get_dashboard_stats, get_student_financial_summary, get_student_charges_with_balances, get_student_payments, set_student_amount_due, set_student_fee_charge, record_student_payment with allocations + payment_receipt_seq + KHA-YYYY-000001 receipt numbers, update_student, reverse_payment, update_student_photo, activate_academic_year, create_staff, toggle_staff_active, reset_staff_password)
  - RLS policies on all tables
  - Storage bucket (student-photos) + storage policies
  - Seed data (academic years including current 2027/2028, 5 programs, 4 houses, 4 fee types, admin account bootstrap)
- Live database already matches this schema; migrations document and reproduce it
- Migrations must be idempotent where possible (e.g., CREATE IF NOT EXISTS, DROP IF EXISTS)

### 5.2 New Database Changes

**Profiles table (if missing):**
- `is_active` (boolean, default true): staff account active status

**Payments table (if missing):**
- `payment_method` (enum or text: cash, bank transfer, mobile money, cheque, other)
- `payment_date` (date, default today)
- `reference_number` (text, optional)
- `notes` (text, optional)
- `reversal_reason` (text, optional)
- `reversed_at` (timestamp, optional)

**New RPCs:**
- `update_student`: edit enrolled student record (admin only), input validation, audit logging
- `reverse_payment`: reverse completed payment (admin only), restore balances, audit logging
- `toggle_staff_active`: toggle staff active/inactive status (admin only), update profile.is_active, audit logging (action: staff_toggled_active or staff_toggled_inactive)
- `reset_staff_password`: reset staff password to new temporary password (admin only), update auth.users + auth.identities + token columns, audit logging (action: staff_password_reset)

### 5.3 Security Requirements

**Preserve M2 RBAC**
- Login/logout/session verification
- Role retrieval: `get_my_role()`, `is_admin()`, `is_finance_officer()`
- Session verification: `verifySession()`, `getOptionalSession()`
- Authorization checks: `requireAdmin()`, `requireFinanceOrAdmin()`
- Unauthorized page
- Middleware protection
- Server-side authorization for every sensitive operation
- **Session enforcement**: AuthContext checks profile.is_active on load, signs out inactive accounts

**RLS Audit**
- All tables have RLS enabled: students, profiles, payments, payment_allocations, student_charges, academic_years, programs, houses, fee_types, fee_configurations, audit_logs
- Finance officer cannot perform admin-only operations
- SECURITY DEFINER functions require: secure search_path, authentication check, input validation, appropriate authorization

**Audit Logs**
- Record events: enrollment, amount due change, charge change, payment, payment reversal, student update, student status update, configuration change, staff/role change, staff toggle active/inactive, staff password reset, password change
- Record content: user, action, entity type/ID, description, timestamp
- Regular users cannot rewrite audit history

## 6. Deployment Configuration

### 6.1 Vercel Configuration

- Create `vercel.json` in repository root
- Configure SPA rewrites: all routes rewrite to /index.html (except API routes and static assets)
- Example configuration:
  ```json
  {
    \"rewrites\": [
      { \"source\": \"/(.*)\", \"destination\": \"/index.html\" }
    ]
  }
  ```
- Ensure build command and output directory match Vite defaults (build command: `vite build`, output directory: `dist`)

## 7. Legacy Cleanup

### 7.1 Cleanup Items

- Remove incorrect references to old `index_number` identifier
- Remove title-case student type values (Boarding/Day)
- Update outdated README/comments/types/queries/filters
- Remove all mock/fake functionality (fake success, placeholder CRUD, TODO implementations, hardcoded counts, fake payments/receipts/balances, role bypasses)

## 8. Quality Requirements

### 8.1 Error Handling

- All pages and operations handle: loading, empty data, validation errors, authorization errors, not found, database errors, unexpected errors
- Do not expose raw SQL internals to users
- No silent failures
- React ErrorBoundary catches render crashes and displays fallback UI with retry/recover options

### 8.2 UI/UX Consistency

- All pages use school brand colors and design system
- Responsive design (mobile + desktop)
- Pages include: login, dashboard, students, student detail, enrollment, edit student, finance, payment, receipt, reconciliation, reports, academic years, programs, houses, fee types, staff, change password, unauthorized, error/not found
- Typography: Fraunces (headings), Public Sans (body)

### 8.3 Data Layer

- Server-side Supabase client (src/lib/data.ts)
- Typed queries
- Correct joins/pagination/filters
- No mock data

### 8.4 No Mock Functionality

- Prohibit any mock/fake functionality: fake success, placeholder CRUD, TODO implementations, hardcoded counts, fake payments/receipts/balances, role bypasses

## 9. Exceptions and Edge Cases

| Scenario | Handling |
|----------|----------|
| Duplicate JHS/BECE index number | Reject enrollment, display error |
| Amount due is null | Display \"not set\", not zero |
| Payment amount exceeds outstanding balance | Reject payment, display error |
| Payment allocation total ≠ payment amount | Reject payment, display error |
| Single allocation exceeds charge remaining balance | Reject payment, display error |
| Activate academic year when current year exists | Reject activation, display error |
| Finance officer attempts admin function | Redirect to unauthorized page |
| Unauthorized user accesses photo proxy route | Return 403 error |
| Database query fails | Display friendly error, log |
| Student type uses uppercase query | Normalize to lowercase before query |
| Delete referenced program/house/fee type | Reject deletion, display foreign key constraint error |
| Reverse already-reversed payment | Reject reversal, display error |
| Edit student with invalid data | Reject update, display validation errors |
| Export with no data | Generate empty CSV with headers |
| Inactive staff attempts login | Reject login, display error (account inactive) |
| User enters wrong current password on change password | Display error (incorrect current password) |
| New password does not meet strength requirements | Display error (password too weak) |
| Admin resets password for non-existent staff | Display error (staff not found) |
| React component render crash | ErrorBoundary catches error, displays fallback UI with retry option |

## 10. Acceptance Criteria

1. Admin logs in, views dashboard displaying real-time database statistics (total students, active students, boarding students, day students, current academic year, program distribution, house distribution, finance overview with amount due/collected/outstanding for current year, recent payments list)
2. Admin creates new student enrollment (input JHS/BECE index number, name, gender, date of birth, guardian info, program, house, student type boarding/day, academic year), uploads student photo, system rejects duplicate index numbers, records audit log on success
3. Admin edits enrolled student record (personal, guardian, academic, enrollment status fields), system validates input, records audit log (action: student_updated), cannot edit JHS/BECE index number
4. Admin updates student enrollment status (active/graduated/withdrawn) from student detail page with optional reason, system records audit log (action: student_status_updated)
5. Finance officer searches student by JHS/BECE index number, views student detail page displaying personal/guardian/academic/financial info, photo loads securely via proxy route
6. Finance officer sets amount due for student (input numeric value, null displays as \"not set\"), creates fee charge (select fee type, input amount, description), system records audit log
7. Finance officer records student payment using complete form (input payment amount, select payment method, set payment date, enter reference number and notes, manually allocate to specific charges with allocation total equal to payment amount and single allocation not exceeding charge remaining balance), system atomically records payment and allocations, generates receipt number (format KHA-YYYY-000001) only after database confirmation, displays receipt detail and print option
8. Admin reverses completed payment (input reversal reason), system atomically sets payment status to 'reversed', restores allocation balances, records audit log (action: payment_reversed), prevents reversal of already-reversed payments
9. Admin and finance officer export student list as CSV (respecting current filters) and export outstanding balances report as CSV, client-side generation from database-backed data
10. Finance officer views payment history (display receipt number, date, amount, student, allocation details, payment method, reference number, notes, recorded by, status), views financial reconciliation (display amount due, total charges, paid, outstanding balance, charge difference, financial status)
11. Admin manages academic years (create, activate, system prevents multiple current years), manages programs/houses/fee types (CRUD, pre-insert validation no duplicates, deletion checks foreign key constraints)
12. Admin manages staff accounts (create with temporary password, assign roles Admin/Finance Officer, toggle active/inactive status, reset password to new temporary password), system records audit logs (staff_created, staff_toggled_active, staff_toggled_inactive, staff_password_reset), inactive staff cannot sign in
13. All logged-in users (admin and finance officer) change their own password via Change Password page (input current password, new password, confirm new password), system validates new password strength and confirmation match, uses Supabase updateUser with old password verification, records audit log (action: password_changed), displays success or error
14. System enforces session checks: AuthContext checks profile.is_active on load, signs out inactive accounts, login page rejects inactive accounts after authentication
15. System uses React ErrorBoundary to catch render crashes, displays fallback UI with error message and retry/recover options, prevents entire app whitescreen
16. System all pages handle loading/empty data/validation errors/authorization errors/not found/database errors states, do not expose raw SQL info, no silent failures
17. Complete SQL migration files created in repository (supabase/migrations/*.sql) covering entire schema (enums, tables including profiles.is_active and payments.payment_method/payment_date/reference_number/notes/reversal_reason/reversed_at, indexes, constraints, triggers, SECURITY DEFINER functions including update_student, reverse_payment, toggle_staff_active, reset_staff_password, RLS policies, storage bucket + policies, seed data), migrations are ordered, re-runnable from scratch, idempotent where possible
18. Role-based access control complete audit: finance officer CAN set amount due/charges/record payments/view everything needed/change own password, finance officer CANNOT enroll/edit students/update student status/manage staff/config/reverse payments/toggle staff/reset passwords; admin can do everything; all enforced server-side via RLS and RPCs
19. Legacy cleanup executed (remove old index_number references, title-case student type values, outdated README/comments/types/queries/filters, all mock/fake functionality)
20. Vercel deployment configuration created (vercel.json with SPA rewrites, build command and output directory match Vite defaults)

## 11. Out of Scope for This Phase

- Student authentication portal (students cannot log in)
- Student self-service fee/payment query
- Online payment integration (Stripe, PayPal, etc.; offline payment recording only)
- Batch import student data
- Batch export financial reports (beyond CSV exports specified)
- SMS/email notifications
- Native mobile apps
- Multi-language support (English only)
- Advanced reporting and data analytics (beyond CSV exports specified)
- Student academic results management
- Timetable management
- Attendance management
- Library management
- Hostel bed allocation management
- Cafeteria management
- School bus management
- Parent portal
- Teacher portal
- Online assignment submission
- Online examination system
- Two-factor authentication (2FA)
- Password complexity policy enforcement (beyond basic strength validation)
- Password expiration policy
- Account lockout after failed login attempts
- Email verification for staff accounts
- Forgot password self-service (admin resets passwords manually)
