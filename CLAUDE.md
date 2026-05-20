# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

智慧教室管理系统 (Smart Classroom Management System) — a full-stack platform with two deployable units:

| Unit | Backend | Frontend | Port |
|------|---------|----------|------|
| `admin/` | Spring Boot 2.5 + Java 8 | Vue 3 + Vite + Element Plus | 8080 / 80 |
| `app/` | Spring Boot 2.5 + Java 8 | uni-app (Vue 3) cross-platform | 6789 |

Both backends share the same SQLite database. The admin docker-compose mounts `app/data/sqlite` so they operate on the same data. The `demo/` directory contains bundled dev tools (JDK 1.8, Maven 3.9.9, Node 22.12).

## Common commands

### Admin backend (Spring Boot)
```bash
# Build (SQLite profile, skip tests)
cd admin/backend && ./mvnw clean package -Dmaven.test.skip=true -P sqlite

# Build with MySQL profile
cd admin/backend && ./mvnw clean package -Dmaven.test.skip=true -P mysql

# Docker deploy (full stack: backend + nginx frontend)
cd admin && docker-compose up -d --build
```

### Admin frontend (Vue 3)
```bash
cd admin/frontend && npm install
npm run dev          # Dev server with HMR, proxies /classroom-backend to localhost:8080
npm run build        # Production build into dist/
```

### App backend (Spring Boot)
```bash
cd app/backend && ./mvnw clean package -Dmaven.test.skip=true -P sqlite
```

### App frontend (uni-app)
```bash
cd app/frontend && npm install
npm run dev:mp-weixin    # WeChat Mini Program dev
npm run build:h5          # H5 web build
npm run build:app-android # Android build
```

## Architecture notes

### Backend package structure (`com.kuafu`)
Both backends follow the same layered architecture:
- `common/` — Shared framework: cache abstraction (local/Redis), AOP logging/locking, delay task scheduler, dynamic scheduled cron, MyBatis-Plus config, file upload/storage (local/S3), Spring Security + JWT auth, sensitive word filtering, Knife4j API docs
- `common/domin/` — `BaseResponse<T>`, `ErrorCode`, `ResultUtils` (uniform API response envelope)
- `common/login/` — `LoginUser`, `SecurityUtils` (JWT token-based auth with Spring Security)
- `app/backend` additionally has: `common/ai_agent/` (LLM agent framework with SSE streaming), `common/dynamic_config/` (DB-backed config with hot refresh), `common/api/` (dynamic API proxy/gateway), OCR (Tesseract)

### Database
- Default profile is `sqlite` (activated in both pom.xml via `<activeByDefault>true</activeByDefault>`)
- MySQL available as alternative via `-P mysql`
- Flyway manages schema migrations
- The two backends share the same SQLite file via Docker volume mount

### Admin frontend routing
- Static routes in `src/router/index.js` (login, login_manger, api_setting_manger)
- Dynamic routes loaded in `src/permission.js` from `src/router/dynamic` — added to router after authentication
- `VITE_USE_LOGIN` env var (in `.env.development`) toggles auth enforcement
- Base path is `/classroom/`, API proxy path is `/classroom-backend/`

### uni-app frontend
- Components use naming convention: `base-*.vue` files in `src/components/` are auto-resolved via `easycom` in `pages.json`
- Pages are split: main package (`pages/login`, `pages/web_view`) + subpackage (`pagesA/` for admin/home/booking/repair/course pages)
- Uses Tailwind CSS via `weapp-tailwindcss` plugin

### Key env vars
Both backends read these from environment:
- `DB_TYPE` — `sqlite` (default) or `mysql`
- `CACHE_TYPE` — `local` (default) or `redis`
- `LOGIN_ENABLE` — `true` (default) to require auth
- `UPLOAD_PATH` — file upload directory
- `STORAGE_TYPE` — `local` (default) or `s3` (Huawei OBS)

### Security
- Spring Security with JWT (`io.jsonwebtoken:jjwt:0.9.1`)
- Token expiration: 43200 seconds (12 hours)
- `common/annotation/DistributedLock.java` + Redis/Redisson for distributed locking
- Sensitive word filtering via `common/sensitive/`
