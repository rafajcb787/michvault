# Micheline Bilingual Subscription Platform Concept (GitHub-Ready)

## 1) Brand
- **Primary Name:** **MichiPack**
  - Why: short, memorable, pronounceable in English/Spanish, easy app icon wordmark.
- **Alternate 1:** AulaSprint
- **Alternate 2:** PractiListo

**Taglines**
- **EN:** “Practice made simple for busy families.”
- **ES:** “Práctica escolar simple para familias ocupadas.”

- **EN:** “Monthly learning packs, zero stress.”
- **ES:** “Paquetes mensuales de aprendizaje, cero estrés.”

## 2) One-paragraph concept
**EN:** MichiPack is a bilingual (English/Spanish) subscription platform where parents pay $25/month to access high-quality printable and downloadable practice resources (worksheets, quizzes, exams, answer keys, and study guides), request custom materials, and receive automated monthly packs tailored to grade/subject preferences—through both app notifications and email—so Micheline can run a scalable teaching business with minimal day-to-day manual work.

**ES:** MichiPack es una plataforma de suscripción bilingüe (inglés/español) donde los padres pagan $25/mes para acceder a recursos de práctica de alta calidad, descargables e imprimibles (hojas de trabajo, quizzes, exámenes, claves de respuestas y guías de estudio), enviar solicitudes personalizadas y recibir paquetes mensuales automáticos según preferencias de grado/materia—tanto en la app como por correo—permitiendo que Micheline escale su servicio con el mínimo trabajo operativo diario.

## 3) User personas (at least 3)
1. **Parent Planner Paula (Primary Buyer)**
   - Age 34, works full-time, 2 children.
   - Needs quick, trustworthy materials by grade and subject.
   - Behavior: uses mobile at night, prints from desktop on weekends.
   - Success metric: 10-minute weekly prep and visible child progress.

2. **Parent Supporter Carlos (Spanish-first user)**
   - Age 41, prefers Spanish UI and email.
   - Needs bilingual support and easy reminders.
   - Behavior: responds best to WhatsApp-style concise notifications + email links.
   - Success metric: can find and print practice without translation friction.

3. **Micheline Admin (Teacher/Creator)**
   - Solo operator, limited time.
   - Needs bulk upload, bulk tagging, scheduled releases, canned replies.
   - Behavior: batches work once weekly.
   - Success metric: low support load and high subscription retention.

4. **Future Assistant Admin (Scale Persona)**
   - Part-time helper.
   - Needs permissions for moderation and messaging, not billing secrets.
   - Ensures platform remains multi-tenant ready.

## 4) User journeys (4 flows)
### A) Subscribe → Monthly Pack
1. Parent opens web/app, selects language (auto-detected with manual toggle).
2. Enters email, receives OTP, verifies.
3. Completes onboarding preferences: grades, subjects, languages, weekly goals.
4. Starts Stripe subscription ($25/month).
5. System creates active subscription + next billing date.
6. On monthly cron, system generates personalized pack from tagged content.
7. Parent gets in-app notification + bilingual email with signed download links.
8. Parent opens pack, downloads/prints, tracks completion.

### B) Find worksheet → Print → Mark complete
1. Parent enters Library.
2. Navigates Grade → Subject → Topic/Skill or uses search + filters.
3. Opens content detail (preview, difficulty, estimated time).
4. Clicks Download/Print (signed URL).
5. Marks activity complete.
6. System recommends “next practice” based on same grade/skill progression.

### C) Submit request → Get suggestions
1. Parent opens “Custom Request” form.
2. Fills required fields + optional attachment.
3. Before submit, system suggests existing matching content by tags.
4. Parent either uses suggested content or submits request.
5. Request enters status pipeline: Received → In Review → Accepted → Delivered.
6. Parent receives in-app/email updates each status change.

### D) Micheline upload → Tag → Schedule
1. Admin opens Upload Studio.
2. Drag-and-drop single/bulk files.
3. Applies metadata template and bulk tag rules.
4. Sets publish date + optional inclusion in monthly pack themes.
5. Saves new version if replacing old material.
6. System indexes for search, updates recommendations, schedules announcements.

## 5) Sitemap / IA
### Web (Next.js)
- `/` Landing (pricing, value, testimonials, CTA)
- `/auth/login` Email OTP entry
- `/auth/verify` OTP verification
- `/onboarding` preferences setup
- `/dashboard` parent home (pack status, recommendations)
- `/library` browse/search/filter
- `/library/[contentId]` content detail
- `/packs` monthly packs list
- `/packs/[packId]` pack detail
- `/requests` custom requests list
- `/requests/new` request form
- `/messages` conversation inbox
- `/settings/profile`
- `/settings/language`
- `/settings/subscription`
- `/admin` admin overview
- `/admin/content` content manager
- `/admin/uploads` upload studio
- `/admin/requests` request triage
- `/admin/messages` templates + inbox

### Mobile (Expo)
- Tabs: Home, Library, Packs, Requests, Messages, Settings
- Deep screens: ContentDetail, PackDetail, RequestForm, Subscription, Language, Notifications

## 6) Screen list (minimum 18)
1. Landing page (EN/ES hero, pricing, CTA)
2. OTP login screen
3. OTP verify screen
4. Onboarding preferences
5. Parent dashboard
6. Library browse screen (hierarchy chips)
7. Library search results + filters
8. Content detail (preview + metadata)
9. Download/print confirmation modal
10. Monthly packs list
11. Monthly pack detail
12. Request list + statuses
13. New custom request form
14. Request detail timeline
15. Messages inbox
16. Conversation thread view
17. Settings profile
18. Settings language toggle
19. Settings subscription/billing
20. Admin dashboard metrics
21. Admin upload studio (bulk)
22. Admin content table (bulk tagging/versioning)
23. Admin request queue board
24. Admin template manager (email/canned replies)

## 7) Data model (Postgres + Prisma)
### Core tables
1. `tenants`
   - `id (uuid pk)`, `name`, `slug unique`, `created_at`
   - Index: `slug`

2. `users`
   - `id (uuid pk)`, `tenant_id fk`, `email unique(tenant_id,email)`, `role enum(parent,admin)`,
   - `preferred_language enum(en,es)`, `timezone`, `created_at`, `last_login_at`
   - Indexes: `(tenant_id,email) unique`, `(tenant_id,role)`

3. `auth_otps`
   - `id`, `tenant_id`, `email`, `code_hash`, `expires_at`, `used_at`, `attempt_count`
   - Indexes: `(tenant_id,email,expires_at)`, `(expires_at)`

4. `subscriptions`
   - `id`, `tenant_id`, `user_id`, `stripe_customer_id`, `stripe_subscription_id`,
   - `status enum(trialing,active,past_due,canceled)`, `price_cents`, `currency`, `current_period_end`
   - Indexes: `(tenant_id,user_id)`, `(stripe_subscription_id) unique`, `(status)`

5. `learning_preferences`
   - `id`, `tenant_id`, `user_id`, `grades text[]`, `subjects text[]`, `topics text[]`,
   - `content_types text[]`, `preferred_language enum(en,es,both)`, `weekly_goal int`
   - Index: `(tenant_id,user_id) unique`

6. `content_items`
   - `id`, `tenant_id`, `title`, `description`, `language enum(en,es,bilingual)`,
   - `grade_levels text[]`, `subjects text[]`, `topics text[]`, `difficulty enum(beginner,intermediate,advanced)`,
   - `content_type enum(worksheet,exam,quiz,answer_key,study_guide)`, `estimated_minutes int`,
   - `status enum(draft,published,archived)`, `version int`, `parent_content_id nullable`,
   - `created_by`, `published_at`, `created_at`
   - Indexes: `(tenant_id,status,published_at desc)`, `GIN(grade_levels)`, `GIN(subjects)`, `GIN(topics)`, `(content_type)`, `(language)`

7. `content_assets`
   - `id`, `tenant_id`, `content_item_id`, `storage_bucket`, `storage_key`, `mime_type`, `file_size`, `checksum`, `created_at`
   - Indexes: `(tenant_id,content_item_id)`, `(storage_bucket,storage_key) unique`

8. `monthly_packs`
   - `id`, `tenant_id`, `user_id`, `month date`, `language enum(en,es)`, `status enum(generated,sent,opened)`, `generated_at`, `sent_at`
   - Indexes: `(tenant_id,user_id,month) unique`, `(status)`

9. `monthly_pack_items`
   - `id`, `tenant_id`, `pack_id`, `content_item_id`, `sort_order`
   - Indexes: `(tenant_id,pack_id)`, `(pack_id,content_item_id) unique`

10. `custom_requests`
   - `id`, `tenant_id`, `user_id`, `parent_name`, `parent_email`, `student_grade`, `subject`, `topic_skill`,
   - `struggle_notes text`, `deadline_at nullable`, `preferred_language enum(en,es)`,
   - `status enum(received,in_review,accepted,delivered)`, `created_at`, `updated_at`
   - Indexes: `(tenant_id,user_id,created_at desc)`, `(tenant_id,status,updated_at desc)`

11. `custom_request_attachments`
   - `id`, `tenant_id`, `request_id`, `storage_bucket`, `storage_key`, `mime_type`, `file_size`
   - Index: `(tenant_id,request_id)`

12. `messages`
   - `id`, `tenant_id`, `thread_id`, `sender_user_id`, `recipient_user_id`, `body`, `language enum(en,es)`, `sent_at`, `read_at`
   - Indexes: `(tenant_id,thread_id,sent_at)`, `(recipient_user_id,read_at)`

13. `notifications`
   - `id`, `tenant_id`, `user_id`, `channel enum(in_app,email)`, `template_key`, `payload jsonb`, `language`, `status`, `sent_at`
   - Indexes: `(tenant_id,user_id,status)`, `(template_key,sent_at)`

14. `audit_logs`
   - `id`, `tenant_id`, `actor_user_id`, `action`, `entity_type`, `entity_id`, `metadata jsonb`, `created_at`
   - Indexes: `(tenant_id,created_at desc)`, `(entity_type,entity_id)`

## 8) API endpoints (NestJS REST + Swagger)
### Auth OTP
- `POST /v1/auth/otp/send` { email, locale }
- `POST /v1/auth/otp/verify` { email, code, deviceInfo }
- `POST /v1/auth/logout`
- `GET /v1/auth/me`

### Subscriptions / Billing
- `POST /v1/subscriptions/checkout-session` (Stripe)
- `GET /v1/subscriptions/me`
- `POST /v1/subscriptions/webhook/stripe` (signed)
- `POST /v1/subscriptions/cancel`
- `GET /v1/subscriptions/invoices`

### Library
- `GET /v1/library/content` (q, grade, subject, topic, type, difficulty, language, page)
- `GET /v1/library/content/:id`
- `GET /v1/library/content/:id/download-url` (signed URL)
- `POST /v1/library/content/:id/complete`
- `GET /v1/library/recommendations/next`

### Monthly Packs
- `GET /v1/packs`
- `GET /v1/packs/:id`
- `POST /v1/packs/:id/mark-opened`
- `POST /v1/packs/generate` (admin/cron)
- `POST /v1/packs/send` (admin/cron email + in-app)

### Custom Requests
- `POST /v1/requests/suggestions` (pre-submit related content)
- `POST /v1/requests`
- `GET /v1/requests`
- `GET /v1/requests/:id`
- `PATCH /v1/requests/:id/status` (admin)
- `POST /v1/requests/:id/attachments/upload-url`

### Messaging
- `GET /v1/messages/threads`
- `GET /v1/messages/threads/:threadId`
- `POST /v1/messages/threads/:threadId`
- `POST /v1/messages/templates/:key/send` (admin canned templates)

### Admin Uploads / Content
- `POST /v1/admin/uploads/presign`
- `POST /v1/admin/content`
- `POST /v1/admin/content/bulk`
- `PATCH /v1/admin/content/bulk-tags`
- `PATCH /v1/admin/content/:id`
- `POST /v1/admin/content/:id/new-version`
- `POST /v1/admin/content/:id/schedule`
- `GET /v1/admin/requests`

## 9) RBAC permissions matrix
| Capability | Parent | Admin |
|---|---:|---:|
| Manage own profile + language | ✅ | ✅ |
| Manage own subscription | ✅ | ✅ (support view) |
| Browse/download own allowed content | ✅ | ✅ |
| Mark completion | ✅ | ✅ |
| Submit/view own custom requests | ✅ | ✅ |
| View all custom requests | ❌ | ✅ |
| Change request statuses | ❌ | ✅ |
| Send direct messages | ✅ | ✅ |
| Manage canned templates | ❌ | ✅ |
| Upload/edit/publish content | ❌ | ✅ |
| Bulk tagging/versioning | ❌ | ✅ |
| View tenant analytics | ❌ | ✅ |
| Access audit logs | ❌ | ✅ |

## 10) MVP build plan (6 weeks)
### Week 1: Foundations
- Initialize Turborepo + pnpm workspaces.
- Scaffold Next.js, Expo, NestJS apps.
- Setup Prisma schema + Postgres.
- Add shared package (types, zod, i18n loader).

### Week 2: Auth + i18n + basic UI
- Implement OTP auth (SendGrid) in API.
- Build login/onboarding flow web + mobile.
- Implement EN/ES translation JSON shared across web/mobile.
- Persist language preference in user profile.

### Week 3: Subscription + Library
- Stripe checkout + webhook + subscription states.
- Library data model, listing endpoints, filters/search.
- Web/mobile library screens with print/download actions.

### Week 4: Requests + Messaging
- Custom request form + attachment upload (MinIO/S3).
- Suggestion endpoint before request submit.
- Messaging threads + notifications (in-app + email).

### Week 5: Admin Portal + Automation
- Admin upload studio, bulk metadata, scheduling.
- Monthly pack generator + sender jobs.
- Canned EN/ES templates and FAQ quick replies.

### Week 6: Hardening + Tests + Launch readiness
- E2E critical flows (subscribe, browse, request).
- Security pass (signed URLs, rate limits, audit logs).
- CI/CD pipelines + staging deploy docs.

## 11) GitHub monorepo repo tree (exact folders)
```txt
/
├─ apps/
│  ├─ web/                     # Next.js App Router + Tailwind
│  ├─ mobile/                  # Expo React Native
│  └─ api/                     # NestJS REST + Swagger
├─ packages/
│  └─ shared/
│     ├─ types/
│     ├─ zod/
│     └─ i18n/
│        ├─ en.json
│        └─ es.json
├─ infra/
│  ├─ docker-compose.yml
│  ├─ scripts/
│  └─ env/
├─ turbo.json
├─ pnpm-workspace.yaml
├─ package.json
└─ README.md
```

## 12) Local development
### docker-compose services
- `postgres:16-alpine` (port 5432)
- `minio/minio` (ports 9000, 9001)
- `redis:7-alpine` (optional queue/cache)

### Example `.env.example` files
**Root `.env.example`**
```env
NODE_ENV=development
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/michvault
REDIS_URL=redis://localhost:6379
JWT_SECRET=change-me
APP_URL_WEB=http://localhost:3000
APP_URL_API=http://localhost:4000
```

**`apps/api/.env.example`**
```env
PORT=4000
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/michvault
SENDGRID_API_KEY=sg_xxx
SENDGRID_FROM_EMAIL=no-reply@michipack.com
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
STRIPE_PRICE_ID=price_25_monthly
S3_ENDPOINT=http://localhost:9000
S3_REGION=us-east-1
S3_BUCKET=materials
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minioadmin
S3_FORCE_PATH_STYLE=true
```

**`apps/web/.env.example`**
```env
NEXT_PUBLIC_API_URL=http://localhost:4000/v1
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_xxx
```

**`apps/mobile/.env.example`**
```env
EXPO_PUBLIC_API_URL=http://localhost:4000/v1
EXPO_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_xxx
```

## 13) Scripts (pnpm)
```json
{
  "scripts": {
    "dev": "turbo dev",
    "build": "turbo build",
    "lint": "turbo lint",
    "test": "turbo test",
    "test:e2e": "pnpm --filter web test:e2e && pnpm --filter api test:e2e",
    "db:migrate": "pnpm --filter api prisma migrate dev",
    "db:seed": "pnpm --filter api prisma db seed"
  }
}
```

## 14) CI/CD (copy-paste GitHub Actions)
### `.github/workflows/ci.yml`
```yaml
name: CI
on:
  pull_request:
  push:
    branches: [main]

jobs:
  lint-test-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with:
          version: 9
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm test
      - run: pnpm build
```

### `.github/workflows/integration.yml`
```yaml
name: Integration
on:
  pull_request:
  workflow_dispatch:

jobs:
  api-integration:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: michvault_test
        ports: ["5432:5432"]
        options: >-
          --health-cmd "pg_isready -U postgres"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      minio:
        image: minio/minio
        env:
          MINIO_ROOT_USER: minioadmin
          MINIO_ROOT_PASSWORD: minioadmin
        ports: ["9000:9000", "9001:9001"]
        options: >-
          --health-cmd "curl -f http://localhost:9000/minio/health/live || exit 1"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        command: server /data --console-address ":9001"
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with:
          version: 9
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm --filter api prisma migrate deploy
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/michvault_test
      - run: pnpm --filter api test:integration
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/michvault_test
          S3_ENDPOINT: http://localhost:9000
          S3_ACCESS_KEY: minioadmin
          S3_SECRET_KEY: minioadmin
```

### `.github/workflows/e2e.yml`
```yaml
name: E2E
on:
  pull_request:
  workflow_dispatch:

jobs:
  playwright-e2e:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: michvault_e2e
        ports: ["5432:5432"]
      minio:
        image: minio/minio
        env:
          MINIO_ROOT_USER: minioadmin
          MINIO_ROOT_PASSWORD: minioadmin
        ports: ["9000:9000", "9001:9001"]
        command: server /data --console-address ":9001"
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with:
          version: 9
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm exec playwright install --with-deps
      - run: pnpm --filter api prisma migrate deploy
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/michvault_e2e
      - run: pnpm dev &
      - run: pnpm --filter web test:e2e
```

## 15) Testing strategy (MVP)
### Unit
- API services: OTP validation, pack selection, request status transitions.
- Shared i18n keys parity test (`en.json` vs `es.json` key coverage).
- Web/mobile component tests for language toggle + filters.

### Integration
- Auth OTP issue/verify + expiry.
- Stripe webhook status updates.
- Content search/filter with Postgres + Prisma.
- Signed URL generation against MinIO.
- Request creation with attachment metadata.

### E2E (critical paths)
1. Parent login via OTP → set language → subscribe.
2. Browse library → download worksheet → mark complete → get recommendation.
3. Submit custom request → see suggestion cards → track status updates.
4. Admin upload bulk content → bulk tag → publish schedule.

## 16) Copywriting
### Onboarding text
- **EN:** “Welcome to MichiPack! Tell us your child’s grade and subjects so we can build monthly practice packs for you.”
- **ES:** “¡Bienvenido a MichiPack! Cuéntanos el grado y las materias para crear tus paquetes mensuales de práctica.”

### 10 menu labels EN/ES
1. Home / Inicio
2. Library / Biblioteca
3. Monthly Pack / Paquete Mensual
4. Requests / Solicitudes
5. Messages / Mensajes
6. Subscription / Suscripción
7. Settings / Configuración
8. Language / Idioma
9. Download / Descargar
10. Print / Imprimir

### 5 short system emails EN/ES
1. **OTP Code**
   - EN: “Your MichiPack code is {{code}}. It expires in 10 minutes.”
   - ES: “Tu código de MichiPack es {{code}}. Vence en 10 minutos.”
2. **Welcome**
   - EN: “Welcome! Your subscription is active. Your first pack is ready.”
   - ES: “¡Bienvenido! Tu suscripción está activa. Tu primer paquete está listo.”
3. **Monthly Pack Available**
   - EN: “Your {{month}} practice pack is now available.”
   - ES: “Tu paquete de práctica de {{month}} ya está disponible.”
4. **Request Status Updated**
   - EN: “Your request ‘{{topic}}’ is now {{status}}.”
   - ES: “Tu solicitud ‘{{topic}}’ ahora está en estado {{status}}.”
5. **Payment Receipt**
   - EN: “Payment received: $25 for your monthly subscription.”
   - ES: “Pago recibido: $25 por tu suscripción mensual.”

## 17) Privacy & security
- Minimal child data by design: store grade + learning preferences only; no child name required.
- Encrypt sensitive data in transit (TLS) and at rest (managed disk encryption).
- OTP codes stored hashed, short TTL, attempt throttling + IP/email rate limits.
- Signed, short-lived download URLs for files in S3/MinIO.
- Role-based access control and tenant-scoped queries everywhere.
- Audit logs for admin actions (uploads, status changes, message templates).
- Secrets in environment/secret manager; never commit credentials.
- Data retention policy: request attachments auto-expire/archive after configurable window.

## Low-activity automation ideas (MVP-focused)
1. Automatic monthly pack generation from saved preferences.
2. Automatic pack delivery via in-app + email with receipts.
3. “Recommended next practice” after completion events.
4. Request-time smart suggestions matching grade/subject/topic tags.
5. Weekly nudges if no activity detected.
6. FAQ quick replies and canned message templates in EN/ES.
7. Evergreen 4-week learning paths that run automatically per grade/subject.
8. Bonus pack scheduler (e.g., exam season) with auto-expiry links.
9. Auto-status notifications for request lifecycle changes.
10. Bulk upload metadata presets to reduce repetitive admin tagging.

### Optional enhancements (explicitly optional)
- AI-assisted tagging of uploaded content.
- AI summary of parent request into structured metadata.
