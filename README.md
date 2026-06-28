# AssignHub: Intelligent Classroom & Assignment Management 🎓

[![Flutter](https://img.shields.io/badge/Flutter-Android-02569B?style=for-the-badge&logo=flutter)](https://flutter.dev/)
[![FastAPI](https://img.shields.io/badge/FastAPI-Python-009688?style=for-the-badge&logo=fastapi)](https://fastapi.tiangolo.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Database-336791?style=for-the-badge&logo=postgresql)](https://www.postgresql.org/)
[![AWS S3](https://img.shields.io/badge/AWS_S3-Storage-569A31?style=for-the-badge&logo=amazon-s3)](https://aws.amazon.com/s3/)
[![Firebase](https://img.shields.io/badge/Firebase-FCM-FFCA28?style=for-the-badge&logo=firebase)](https://firebase.google.com/)

> *DevFusion 3.0 Hackathon — Problem Statement #26ENAH1*

---

## Problem Statement

- **Administrative Overhead:** Traditional classroom management relies on fragmented tools for assignments, grading, and communication, leading to massive administrative burdens for educators.
- **Lack of Real-Time Insights:** Mentors often realize a student is struggling only after they fail an exam or miss multiple assignments. There is no automated, real-time tracking of student "risk levels".
- **Inefficient Provisioning:** Onboarding hundreds of students and mentors at the start of an academic year requires tedious manual data entry, prone to human error.
- **Fragmented Communication:** Important deadlines and grades are easily missed when hidden inside static portals instead of direct mobile push notifications.
- There is a critical need for a **unified, intelligent assignment management platform** that automates administrative tasks, tracks student performance continuously, and centralizes the educational workflow.

---

## Project Objective

**AssignHub** is a comprehensive, mobile-first educational platform engineered to assist educators and students in managing their daily academic workflow with automated analytics and real-time communication.

The platform aims to:
- **Intelligent Analytics Engine:** Automatically flag "At-Risk" students based on their submission delays and missed deadlines, allowing for early intervention.
- **Smart Provisioning:** Empower administrators to instantly onboard entire classrooms using automated Bulk Excel imports.
- **Real-Time Synchronization:** Keep students and mentors perfectly aligned with live WebSockets and Firebase Cloud Messaging (FCM) for instant deadline and grade alerts.
- **Comprehensive Lifecycle Management:** Handle the complete flow of creating, publishing, submitting, versioning, and grading assignments in one secure place.

---

## Sustainable Development Goals (SDGs)

This project aligns with the following United Nations Sustainable Development Goals:

### SDG 4: Quality Education
- **Target 4.1:** Supports effective and inclusive learning environments by providing educators with the analytics needed to intervene early when students are struggling, improving overall educational outcomes.

### SDG 10: Reduced Inequalities
- **Target 10.2:** Democratizes access to high-quality educational tracking tools, ensuring that institutions of all sizes can easily deploy enterprise-grade classroom management without massive IT budgets.

---

## Proposed Solution

AssignHub uses a **Role-Based Architecture** tailored perfectly for the three distinct pillars of education: Administrators, Mentors, and Students.

### Architecture & Workflow:

1. **Smart Provisioning (Admin):** Admins upload an Excel template or manually input data. The backend automatically creates secure accounts and emails OTP invitations to users.
2. **Classroom Setup (Mentor):** Mentors review pending student rosters, approve legitimate students (which generates their analytics profiles), and manage their classrooms.
3. **Assignment Pipeline:** Mentors publish assignments with rich-text instructions. The system blasts push notifications to all enrolled students.
4. **Submission Engine:** Students upload their work (stored securely in AWS S3) or type answers. The system versions their submissions and issues digital receipts.
5. **Analytics & Intelligence:** 
   - **Student Dashboard:** Tracks individual submission rates and deadlines.
   - **Mentor Dashboard:** Aggregates class performance, flags "At-Risk" students, and provides a bird's-eye view of class health.

---

## 🛠️ Technologies Used

### **Frontend Stack (Mobile)**
- **Framework:** Flutter (Dart)
- **Target Platform:** Android 
- **State Management:** Provider / Riverpod 
- **Real-Time Communication:** WebSockets & Firebase Cloud Messaging (FCM)
- **File Handling:** `file_picker` for native Excel and document uploads

### **Backend Stack**
- **Framework:** FastAPI (Python 3.11+)
- **Database:** PostgreSQL (via SQLAlchemy ORM)
- **Authentication:** JWT (JSON Web Tokens) + Email OTPs
- **File Storage:** AWS S3 (boto3)
- **Background Jobs:** APScheduler (for deadline monitoring)
- **Data Processing:** pandas & openpyxl (for Bulk Imports)

---

## 🚀 Installation & Setup

### Prerequisites
- Flutter SDK (Android Toolchain)
- Python 3.9+
- PostgreSQL

### 1. Clone the Repository
```bash
git clone https://github.com/YourUsername/AssignHub.git
cd AssignHub
```

### 2. Start the Backend Server
```bash
cd Backend
python -m venv venv
venv\Scripts\activate        # On Windows
# source venv/bin/activate   # On macOS/Linux

pip install -r requirements.txt
cp .env.example .env         # Fill in your DB, AWS, and SMTP keys

alembic upgrade head         # Run database migrations
uvicorn main:app --reload --port 8000
```
*The API will be available at `http://localhost:8000/docs`*

### 3. Start the Frontend Application
```bash
cd ../Frontend
flutter pub get
flutter run
```
*Make sure an Android Emulator or physical device is connected.*

### 4. Build Production APK
```bash
cd Frontend
flutter build apk --release
```
*Output: `Frontend/build/app/outputs/flutter-apk/app-release.apk`*

---

## 📊 Project Status

| Domain | Status | Notes |
|:-------|:-------|:------|
| **Frontend App** | ✅ **Stable** | Fully responsive Android UI completed |
| **Backend API** | ✅ **Production-Ready** | 70/70 Integration Tests Passing |
| **Authentication** | ✅ **Secure** | JWT & Email OTP verification active |
| **Analytics Engine**| ✅ **Active** | Real-time risk assessment operational |
| **Bulk Import** | ✅ **Active** | Excel (.xlsx) parsing and validation working |

---

## 🎯 Key Features

- ✅ **Context-Aware Dashboards:** Distinct, optimized interfaces for Admins, Mentors, and Students.
- ✅ **Intelligent Risk Alerts:** Automated flagging of students falling behind on deadlines.
- ✅ **Live Push Notifications:** FCM integration for instant grading and assignment alerts.
- ✅ **Frictionless Onboarding:** 1-click bulk import for thousands of users via `.xlsx`.
- ✅ **Immutable Submissions:** Secure version tracking and digital receipts for all student work.
- ✅ **API-First Design:** Fully documented RESTful endpoints with WebSocket support.

---

<p align="center">
  <strong>Empowering Education Through Intelligent Systems</strong><br>
  Built with ❤️ by Monish
</p>
