# KIVAS Jukebox - Real-Time Bar Jukebox (Quarkus + React)

> **Privacy Note:** The source code of this project is kept private to comply with EPITA's anti-plagiarism regulations. The code is structured, commented, and available upon request for recruiters during an interview. This repository serves as a technical showcase and provides a testable compiled version.

## 📖 Project Context

KIVAS Jukebox is EPITA's PING project (ING1, June-July 2026), built by a team of 4 students in about 3 weeks. The goal: a full-stack web app that lets bar customers choose the music while the manager keeps control.

- **Customers** scan a QR code and vote for the next song from their phone, with no app and no account. The queue updates live.
- **The bar** runs a player on a computer that plays the songs in vote order.
- **The admin** has a panel to upload MP3s, block songs, skip, pause, and reset the votes.

## ✨ Implemented Features

- **Anonymous voting:** each browser gets an anonymous session ID; the backend enforces one vote per song per session (`409 Conflict` on duplicates).
- **Live queue over WebSocket:** a public `/ws/queue` endpoint pushes a snapshot of the queue (now playing, remaining time, upcoming songs with estimated wait time) to every connected client on each change.
- **Server-side playback sync:** the admin player reports its real playback state (playing/paused, remaining seconds); the backend locks the current song, rotates the queue when it ends, and falls back to a safety timer if the player never reports.
- **Admin panel:** MP3 upload with metadata, catalogue management, block/unblock, skip, vote reset, account settings.
- **Authentication:** JWT access + refresh tokens (SmallRye JWT, RSA-signed), role-based access to admin routes.
- **Imposed file-system API:** CRUD endpoints for files and folders (create, read, move, upload, delete) with path validation.

## 🏗️ Technical Architecture

**Backend - Java 21 / Quarkus 3**

1. **Presentation layer:** JAX-RS resources under `/api/*` (songs, admin songs, queue, users, login, files, folders) with dedicated request/response DTOs, plus the WebSocket endpoint.
2. **Business layer:** services (`SongService`, `QueueService`, `UserService`, `JwtTokenService`, `FileSystemService`...) holding the business rules of the jukebox: vote ranking, queue rotation, playback lock.
3. **Data layer:** Hibernate ORM with Panache repositories mapped to PostgreSQL (`users`, `songs`, `votes`). Converters translate entities to DTOs.
4. **Cross-cutting:** exception mappers for consistent HTTP errors, custom logger (stdout + log files).

**Frontend - React 18 / TypeScript / Vite**

- 8 screens: landing, vote, confirmation, live queue, admin dashboard, moderation, catalogue, settings.
- Tailwind CSS + Radix UI components, TanStack Query for API state, a WebSocket client for the live queue.

**DevOps**

- GitHub Actions CI: runs the full test suite against a PostgreSQL 16 service on every pull request.
- Docker multi-stage build + Docker Compose (app + PostgreSQL).
- ~220 JUnit 5 / RestAssured tests covering endpoints, services and converters.

## 🧠 Technical Challenges Solved

- **Keeping every client in sync:** polling would not scale in a crowded bar, so the queue is pushed over WebSocket and recomputed server-side, the single source of truth.
- **Clock drift between the player and the server:** instead of trusting a server timer, the admin player reports the real remaining time of the audio element; the backend only uses its own timer as a safety net.
- **Anonymous but fair voting:** no login for customers, yet duplicate votes are rejected thanks to a per-session vote table.

## 🛠️ Skills

`Java` · `Quarkus` · `React` · `TypeScript` · `PostgreSQL` · `WebSocket` · `JWT` · `Docker` · `GitHub Actions`

## 🎯 How to test the project?

A pre-compiled version (backend + embedded frontend in a single JAR) is available in the **Releases** section.

**Requirements:** Java 21 and Docker.

1. Download `kivas-jukebox.zip` from the Releases section and extract it.
2. Start the database:
   ```bash
   docker compose up -d
   ```
3. Start the application:
   ```bash
   java -jar kivas-jukebox.jar
   ```
4. Open http://localhost:8080 to vote as a customer, and http://localhost:8080/admin/login for the admin panel (demo credentials are in the release notes).

### 👥 The Team
* **[Wael Akhdar](https://github.com/Wael-Akhdar)**
* **[Jessim Ziani](https://github.com/jessim-ziani)**
* **[Louis-Maximilien Chappuit](https://github.com/louischap31)**
* **[Youcef Zahra](https://github.com/youcefzahra)**
