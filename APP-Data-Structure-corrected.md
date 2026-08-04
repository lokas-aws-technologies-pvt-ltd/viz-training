# APP Data Structure (Corrected)

This document corrects the schema described in `APP Data Structure.pdf`. The original PDF is left
untouched as the historical source; this file is the version to build migrations from.

## Changelog vs. original PDF

1. **Batches** — removed the nonsensical `student_id → students.id` FK (a batch is a cohort, not
   tied to one student) and added the missing `college_id → colleges.id` (a batch runs at a
   specific college).
2. **Students** — removed the self-referencing `student_id → students.id` FK (same copy-paste
   error as #1, no valid use case) and added `college_id → colleges.id`, since `Student Evaluation`
   already assumed this column existed.
3. **Student Evaluation** — fixed `college_id` to reference `colleges.id` instead of the invalid
   `students.college_id` (a FK must target a table's key, not another table's arbitrary column).
4. **Assessment** — fixed `trainer_id` to reference `trainers.id` instead of `users.id`, matching
   every other `trainer_id` FK in the schema (Billing, Trainer Batch Mapping, Class Session).
5. **Placement Tracking** — added the missing **Company Positions** table (22) so
   `company_position_id` resolves to a real table instead of a dangling reference.
6. **Naming consistency** — renamed `Subject Table` → `Subjects Table` and `Topic Table` →
   `Topics Table` to match the plural convention used everywhere else, and updated the FKs that
   pointed at them (`Modules.subject_id`, `Class Session.topic_id`).
7. **Casing/typing nits** — `Totalweighted_score` → `total_weighted_score`,
   `Igiver_aptitude_weight`/`Igiver_aptitude_score` → `igiver_aptitude_weight`/`igiver_aptitude_score`
   (snake_case, consistent with every other field), `Selected Boolean (Yes/No)` → plain
   `Boolean, Default FALSE`, and `Track Zoho/nonZoho` → `Text, One of ('Zoho','nonZoho')`.
8. **Audit columns** — added the missing `updated_at` to `Student Selection & Onboarding` and
   `Certificates`, since every other table has one.

### Not changed (flagged for a product decision, not fixed here)

- `Colleges.contact_person_{1,2,3}` and the `panel_member_{1,2,3}_*` columns in `Entrance Score
  Weights` / `Entrance Exam Scores` are still hardcoded to exactly 3. Normalizing these into child
  tables (`college_contacts`, `panel_assignments`) would remove the cap but changes the shape of
  the scoring logic built around "3 panelists" — that's a bigger call than a schema-bug fix, so I
  left it as-is. Happy to do it if you want.
- **Phases 5, 6, and 7 are still missing** from the document. If they were meant to exist, they
  need to be supplied — I can't infer their content.
- No billing line-item table linking `Billing` to the `Class Session`/`Availability` hours it was
  computed from. Left as a known gap rather than guessing at the intended structure.

---

## PHASE 1 – COLLEGE

### 1. Colleges Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| college_name | Text (Not Null) |
| address | Text (Not Null) |
| email | Text (Not Null) |
| phone | Text (Not Null) |
| contact_person_1_name | Text (Not Null) |
| contact_person_1_phone | Text (Not Null) |
| contact_person_2_name | Text |
| contact_person_2_phone | Text |
| contact_person_3_name | Text |
| contact_person_3_phone | Text |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 2. Batches Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| college_id | UUID (Foreign Key → colleges.id, Not Null) |
| batch_year | Text (Not Null) |
| start_month | SmallInt (Not Null, 1–12) |
| end_month | SmallInt (Not Null, 1–12) |
| target_hours_min | Integer, Default 300 |
| target_hours_max | Integer, Default 350 |
| batch_status | Text, One of ('planning','active','completed','archived') |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

---

## PHASE 2 – STUDENT

### 3. Students Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| college_id | UUID (Foreign Key → colleges.id, Not Null) |
| student_name | Text (Not Null) |
| email | Text |
| phone | Text |
| course | Text (Not Null) — e.g. B.Com, BCA, BA, MSc |
| parent_guardian_name | Text (Not Null) |
| parent_guardian_phone | Text (Not Null) |
| parent_guardian_email | Text |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 4. Panel Members Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| panel_member_name | Text (Not Null) |
| email | Text (Not Null, Unique) |
| phone | Text (Not Null) |
| address | Text (Not Null) |
| photograph_url | Text (Not Null) |
| pan_number | Text (Not Null) |
| linkedin_url | Text (Not Null) |
| pay_per_hour | Numeric(10,2) (Not Null), Default 0 |
| visit_allowance | Numeric(10,2), Default 0.00 |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 5. Entrance Score Weights Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| batch_id | UUID (Foreign Key → batches.id) |
| university_marks_weight | Numeric(5,4) |
| attendance_weight | Numeric(5,4) |
| igiver_aptitude_weight | Numeric(5,4) |
| hod_feedback_weight | Numeric(5,4) |
| panel_member_1_weight | Numeric(5,4) |
| panel_member_2_weight | Numeric(5,4) |
| panel_member_3_weight | Numeric(5,4) |
| effective_from | Date (Not Null) |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 6. Entrance Exam Scores Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| batch_id | UUID (Foreign Key → batches.id) |
| student_id | UUID (Foreign Key → students.id) |
| year1_attendance | Numeric(5,2) |
| year1_average_score | Numeric(5,2) |
| year2_attendance | Numeric(5,2) |
| year2_average_score | Numeric(5,2) |
| igiver_aptitude_score | Numeric(5,2) |
| hod_feedback_score | Numeric(5,2) |
| panel_member_1_id | UUID (Foreign Key → panel_members.id) |
| panel_member_1_score | Numeric(5,2) |
| panel_member_1_feedback | Text |
| panel_member_2_id | UUID (Foreign Key → panel_members.id) |
| panel_member_2_score | Numeric(5,2) |
| panel_member_2_feedback | Text |
| panel_member_3_id | UUID (Foreign Key → panel_members.id) |
| panel_member_3_score | Numeric(5,2) |
| panel_member_3_feedback | Text |
| family_income | Numeric(12,2) |
| total_weighted_score | Numeric(6,2) |
| selected | Boolean, Default FALSE |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 7. Student Selection & Onboarding Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| batch_id | UUID (Foreign Key → batches.id) |
| student_id | UUID (Foreign Key → students.id) |
| entrance_rank | Integer |
| track | Text, One of ('Zoho','nonZoho') |
| parent_income_certificate_url | Text — scanned signed PDF (< ₹4 lakh income) |
| deposit_amount | Numeric(10,2) |
| deposit_received_date | Date |
| deposit_receipt_url | Text — PDF receipt when deposit collected |
| deposit_refund_date | Date — refunded on course completion |
| course_status | Text, One of ('in_progress','completed','dropout') |
| course_status_description | Text — reason if dropout (e.g. got job, left country) |
| certificate_status | Text, One of ('not_issued','issued') |
| certificate_id | UUID (Foreign Key → certificates.id) |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 8. Student Evaluation Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| college_id | UUID (Foreign Key → colleges.id, Not Null) |
| subject_id | UUID (Foreign Key → subjects.id, Not Null) |
| module_id | UUID (Foreign Key → modules.id, Not Null) |
| evaluation_name | Text (Not Null) |
| score | Numeric(6,2) (Not Null) |
| max_score | Numeric(6,2) (Not Null) |
| entered_by | UUID (Foreign Key → trainers.id, Not Null) |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 9. Certificates Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| certificate_name | Text (Not Null) |
| description | Text |
| template_url | Text |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

---

## PHASE 3 – TRAINER

### 10. Trainers Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| user_id | UUID (Foreign Key → users.id, Unique) |
| trainer_name | Text (Not Null) |
| email | Text (Not Null, Unique) |
| phone | Text (Not Null) |
| address | Text (Not Null) |
| photograph_url | Text (Not Null) |
| pan_number | Text (Not Null) |
| linkedin_url | Text (Not Null) |
| pay_per_hour | Numeric(10,2) (Not Null), Default 0 |
| visit_allowance | Numeric(10,2), Default 0.00 |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 11. Subjects Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| trainer_id | UUID (Foreign Key → trainers.id, Not Null) |
| subject_name | Text (Not Null, Unique) — e.g. JavaScript, HTML, CSS, Zoho Creator |
| description | Text |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 12. Modules Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| subject_id | UUID (Foreign Key → subjects.id, Not Null) |
| module_name | Text (Not Null) — e.g. JS Syntax |
| description | Text — scope, reading links, introductory vs in-depth notes |
| allocated_hours | Numeric(5,2) (Not Null) |
| sort_order | SmallInt, Default 0 |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 13. Topics Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| module_id | UUID (Foreign Key → modules.id, Not Null) |
| topic_name | Text (Not Null) — e.g. JS Statements, JS Comments |
| description | Text — multi-line paragraph; scope, reading material, video links |
| duration_hours | Numeric(4,2) (Not Null, 1.00–2.00) |
| sort_order | SmallInt, Default 0 |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 14. Trainer Batch Mapping Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| trainer_id | UUID (Foreign Key → trainers.id) |
| batch_id | UUID (Foreign Key → batches.id) |
| assigned_at | Timestamp with Time Zone, Default now() |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

---

## PHASE 4 – TRAINING EXECUTION

### 15. Availability Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| batch_id | UUID (Foreign Key → batches.id) |
| available_date | Date (Not Null) |
| start_time | Time (Not Null) |
| end_time | Time (Not Null) |
| total_hours | Numeric(4,2) (Not Null, > 0) |
| class_type | Text, Must be 'Lab' or 'Classroom' |
| slot_status | Text, One of ('available','mapped','consumed') |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 16. Class Session Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| batch_id | UUID (Foreign Key → batches.id) |
| trainer_id | UUID (Foreign Key → trainers.id) |
| topic_id | UUID (Foreign Key → topics.id) |
| availability_id | UUID (Foreign Key → availability.id, Unique when active) |
| planned_class_mode | Text, One of ('Online Lab','Online Classroom','Offline Lab','Offline Classroom') |
| class_status | Text, One of ('pending','conducted','not_conducted') |
| actual_start_time | Time |
| actual_end_time | Time |
| actual_class_mode | Text, One of ('Online Lab','Online Classroom','Offline Lab','Offline Classroom') |
| attendance_register_taken | Boolean |
| trainer_remarks | Text |
| admin_remarks | Text |
| reschedule_required | Boolean |
| mapped_by | UUID (Foreign Key → users.id) |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 17. Assessment Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| module_id | UUID (Foreign Key → modules.id, Not Null) |
| trainer_id | UUID (Foreign Key → trainers.id, Not Null) |
| assessment_date | Date (Not Null) |
| assessment_type | Text (MCQ / Coding / Practical / Type A / Type B / Type C) |
| question_document | Text (Question Document / Excel File Path / URL) |
| assessment_notes | Text (Instructions / Additional Notes) |
| total_marks | Numeric(5,2), Default 100 |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 18. Roles Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| role_name | Text (Not Null, Unique) |
| description | Text |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 19. Users Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| full_name | Text (Not Null) |
| email | Text (Not Null, Unique) |
| password_hash | Text (Not Null) |
| phone | Text |
| role_id | UUID (Foreign Key → roles.id, Not Null) |
| last_login_at | Timestamp with Time Zone |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

---

## PHASE 8 – PLACEMENT

### 20. Companies Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| company_name | Text (Not Null, Unique) |
| contact_person | Text |
| contact_email | Text |
| contact_phone | Text |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 21. Student Interview Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| student_id | UUID (Foreign Key → students.id) |
| company_id | UUID (Foreign Key → companies.id) |
| interview_date | Date (Not Null) |
| interview_time | Time |
| interview_result | Text, One of ('on_hold','next_round','cleared','failed','pairing') |
| notes | Text |
| scheduled_by | UUID (Foreign Key → users.id) |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 22. Company Positions Table *(new — required by Placement Tracking, absent from the original PDF)*
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| company_id | UUID (Foreign Key → companies.id, Not Null) |
| position_title | Text (Not Null) |
| description | Text |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

### 23. Placement Tracking Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| student_id | UUID (Foreign Key → students.id) |
| company_position_id | UUID (Foreign Key → company_positions.id) |
| placement_status | Text, One of ('not_started','in_progress','placed','not_placed') |
| offer_date | Date |
| acknowledgement_doc_url | Text |
| starting_salary | Numeric(12,2) |
| offer_letter_url | Text |
| placement_notes | Text |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

---

## PHASE 9 – PAYMENT

### 24. Billing Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| trainer_id | UUID (Foreign Key → trainers.id) |
| billing_period_start | Date (Not Null) |
| billing_period_end | Date (Not Null) |
| invoice_number | Text |
| subtotal_amount | Numeric(12,2), Default 0 |
| total_amount | Numeric(12,2), Default 0 |
| billing_status | Text, One of ('draft','pending_approval','approved','paid','rejected') |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |

---

## PHASE 10 – NOTIFICATIONS & SYSTEM

### 25. Notification Templates Table
| Field Name | Data Type / Constraints |
|---|---|
| id | UUID (Primary Key) |
| event_type | Text (Not Null) |
| channel | Text, One of ('email','whatsapp') |
| subject | Text |
| body_template | Text (Not Null) |
| active | Boolean, Default TRUE |
| created_at | Timestamp with Time Zone, Default now() |
| updated_at | Timestamp with Time Zone, Default now() |
