# LinkMe (JobSeeker) — Arabic-First Employment & Professional Networking Platform

![ASP.NET Core](https://img.shields.io/badge/ASP.NET_Core_MVC-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)
![EF Core](https://img.shields.io/badge/EF_Core-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)
![SQL Server](https://img.shields.io/badge/MS_SQL_Server-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)
![SignalR](https://img.shields.io/badge/SignalR_Real--Time-0AA7FF?style=for-the-badge&logo=dotnet&logoColor=white)
![Identity](https://img.shields.io/badge/ASP.NET_Identity-7B2CBF?style=for-the-badge&logo=dotnet&logoColor=white)
![Bootstrap 5](https://img.shields.io/badge/Bootstrap_5-7952B3?style=for-the-badge&logo=bootstrap&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)
![Build](https://img.shields.io/badge/Build-Passing-brightgreen?style=for-the-badge)

> Connecting Saudi job seekers and employers through a localized, real-time professional network — built Arabic-first, not translated as an afterthought.

---

## 🎯 Project Overview & Business Value

Global platforms like LinkedIn localize poorly for the Saudi labor market: weak RTL experiences, generic specialty taxonomies, and no cultural fit. **JobSeeker** targets that gap — an employment and professional networking platform purpose-built for the Saudi market, where job seekers build rich professional profiles, employers discover and track talent, and both sides stay engaged through **real-time notifications** and **profile-view analytics**.

---

## ✨ Key Features

- **Role-based experiences** — dedicated controllers, view models, and dashboards for **Job Seekers**, **Employers**, and **Admins**, backed by ASP.NET Core Identity roles.
- **Rich professional profiles** — separate `JobSeekerProfile` and `EmployerProfile` entities with specialty enforcement via a custom `SpecialtyValidationAttribute` against a curated `SpecialtiesList`.
- **Real-time notifications** — a SignalR `NotificationHub` pushes events (profile views, account activity) instantly over WebSockets; a typed `NotificationType` enum and read/unread state keep the feed structured.
- **Profile-view analytics** — every profile visit is recorded (`ProfileView` entity), so users see exactly who has been looking at them.
- **Email notifications** — SMTP-backed `EmailService` (configurable via `EmailSettings`) for account and activity emails.
- **File management** — validated CV / avatar uploads through `FileService`, served from `wwwroot/uploads`.
- **Hardened HTTP layer** — a custom `SecurityHeadersMiddleware` applies security headers on every response.

### 📸 Screenshots

> Add UI captures to a `docs/screenshots/` folder and reference them here:

```html
<p align="center">
  <img src="docs/screenshots/home.png" alt="Landing page" width="700" />
  <img src="docs/screenshots/jobseeker-dashboard.png" alt="Job seeker dashboard" width="700" />
  <img src="docs/screenshots/notifications.png" alt="Real-time notifications" width="700" />
</p>
```

---

## 🏗️ System Architecture & Tech Stack

**Pattern:** Layered ASP.NET Core MVC with a service layer (SOLID, interface-driven).

**Data flow:**

```
Browser ──HTTP──> SecurityHeadersMiddleware ──> Controller ──> Service ──> EF Core ──> SQL Server
   └──── WebSocket ────> NotificationHub (SignalR) <──── NotificationService
```

| Layer | Components | Responsibility |
|---|---|---|
| Presentation | `Views/` (Razor, per-role folders), `ViewModels/` | Role-specific UI and input shaping |
| Controllers | `Account`, `Admin`, `Employer`, `JobSeeker`, `Home`, `Notification` | Request orchestration, authorization |
| Services | `NotificationService`, `EmailService`, `FileService`, `ProfileViewService` (+ interfaces) | Business logic, side effects |
| Real-Time | `Hubs/NotificationHub` | SignalR WebSocket push channel |
| Data | `ApplicationDbContext` (extends `IdentityDbContext<ApplicationUser>`), EF migrations, `DbInitializer` seeding | Persistence & seed data |
| Cross-cutting | `SecurityHeadersMiddleware`, custom validation attributes | Security & validation |

---

## 🗄️ Database Schema & Entity Relationships

- **`ApplicationUser`** (ASP.NET Core Identity) — base account: credentials, email, `UserType` discriminator.
- **`JobSeekerProfile`** — 1:1 with `ApplicationUser`; specialty, headline, CV path.
- **`EmployerProfile`** — 1:1 with `ApplicationUser`; company info.
- **`ProfileView`** — many-to-one into both viewer and viewed user; timestamped for analytics.
- **`Notification`** — many-to-one to recipient; `NotificationType` enum, read/unread flag, timestamp.
- Plus the standard Identity tables (roles, claims, tokens).

```
ApplicationUser 1──1 JobSeekerProfile
ApplicationUser 1──1 EmployerProfile
ApplicationUser 1──* ProfileView *──1 ApplicationUser   (viewer → viewed)
ApplicationUser 1──* Notification
```

---

## 🚀 Getting Started

**Prerequisites:** .NET SDK 10 · SQL Server (or LocalDB) · EF Core CLI (`dotnet tool install --global dotnet-ef`)

```bash
git clone https://github.com/Khalid341/linkme.git
cd linkme
dotnet restore

# 1. Set your connection string in appsettings.json
# 2. Configure EmailSettings (SMTP host, port, credentials) — use user-secrets, never commit real secrets
dotnet user-secrets set "EmailSettings:Password" "your-smtp-password"

dotnet ef database update
dotnet run
```

Seed roles and an admin account are created automatically by `Data/DbInitializer` on first run.

---

## 🔒 Security & Best Practices Implemented

- **Role-based access control** via ASP.NET Core Identity (`Admin` / `Employer` / `JobSeeker`).
- **SQL-injection prevention** — all data access through parameterized EF Core LINQ; zero raw SQL.
- **Custom `SecurityHeadersMiddleware`** — CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy on every response.
- **Input validation** — Data Annotations on view models plus the custom `SpecialtyValidationAttribute`.
- **Upload hardening** — file type/size validation in `FileService`.
- **Secret hygiene** — SMTP credentials via user-secrets / environment variables, never in source control.

---

## 🗺️ Roadmap

- [ ] Job postings & end-to-end application pipeline
- [ ] Advanced search and specialty-based matching
- [ ] Full Arabic / English internationalization (i18n)
- [ ] GitHub Actions CI/CD pipeline + automated test suite
- [ ] Flutter companion app (shared Web API)

## 🤝 Contributing

Contributions are welcome. Fork the repo, create a feature branch (`git checkout -b feature/your-feature`), commit with clear messages, and open a pull request against `main`. Please keep the layered structure (Controllers → Services → EF Core) and add view models rather than binding entities directly.

## 📄 License

Distributed under the MIT License.
