# PodCast2Web — Software Specification Document

### (Cahier des Charges)

---

## 1. Executive Summary

**PodCast2Web** is an AI-driven SaaS platform that transforms video podcasts into a fully structured, SEO-optimized, and continuously updated content website — automatically.

Where WordPress requires writers, editors, designers, and plugins to publish content, PodCast2Web takes a single input — a video podcast — and generates an entire content ecosystem:

- High-accuracy transcripts
- Multi-format AI-generated blog articles
- Automated chapters, summaries, quotes, and show notes
- Embedded video player with searchable transcript
- A modern, themeable public website ready to be indexed by Google
- Social-ready clips and snippets

The platform's vision is to become **"the WordPress of the AI era for creators"** — an end-to-end publishing operating system, not a CMS.

---

## 2. Vision & Strategic Positioning

### 2.1 Vision Statement

> _Empower every podcaster to operate like a full media company — without writers, editors, or developers._

### 2.2 Mission

Convert any spoken-word video into a self-maintaining, monetizable content website using AI as the entire production team.

### 2.3 Market Positioning

|Dimension|WordPress|Substack|Descript|**PodCast2Web**|
|---|---|---|---|---|
|Content type|Manual blog|Manual newsletter|Manual editing|**Automated multi-format**|
|AI-native|No|Partial|Partial|**Yes (core)**|
|Output|Articles|Emails|Edited media|**Full website + articles + clips + SEO**|
|Effort per episode|High|High|Medium|**Near zero**|
|Hosting included|Optional|Yes|No|**Yes**|

### 2.4 Differentiators

1. **Single-input pipeline**: upload one video → website grows itself.
2. **Multi-tenant SaaS** with isolated themed sites per creator (`creator.podcast2web.app` or custom domain).
3. **Composable AI workflows** that creators can tune (tone, length, style guides).
4. **Modular microservices** so AI components can scale independently.

---

## 3. Target Users & Use Cases

### 3.1 Primary Personas

**P1 — Independent Podcaster ("The Creator")**

- Releases 1–4 episodes/month
- Has no time for SEO, blog writing, or web development
- Wants a public site to grow audience and monetize

**P2 — Media Company / Network ("The Publisher")**

- Manages 5–50+ shows
- Needs centralized branding, analytics, multi-author workflows
- Wants automation at scale

**P3 — Corporate Marketing Team ("The B2B Publisher")**

- Records internal/external interviews, webinars, customer stories
- Needs them turned into thought-leadership articles and a resource hub

**P4 — Coach / Solopreneur ("The Course Creator")**

- Records long-form video content
- Needs articles, lead magnets, and a searchable knowledge base

### 3.2 Core Use Cases

|ID|Use Case|Actor|
|---|---|---|
|UC-01|Upload or import a video podcast (file / YouTube / RSS)|Creator|
|UC-02|Auto-transcribe with speaker diarization|System|
|UC-03|Generate AI blog article(s) from transcript|System|
|UC-04|Edit / approve generated content|Creator|
|UC-05|Auto-publish to creator's public website|System|
|UC-06|Generate short clips for social media|System|
|UC-07|Configure theme, branding, custom domain|Creator|
|UC-08|View audience analytics|Creator|
|UC-09|Manage subscription / billing|Creator|
|UC-10|Visit and search a podcast website|End Visitor|
|UC-11|Manage multiple shows under one workspace|Publisher|
|UC-12|Invite team members with roles|Publisher|

---

## 4. Functional Scope

### 4.1 MVP Features (Phase 1 — first 4 months)

**Ingestion**

- Direct upload (MP4, MOV, MP3, WAV up to 5 GB)
- Import from public URL / YouTube
- Background processing queue with progress UI

**AI Processing**

- Transcription with timestamps and speaker labels
- Auto-generated:
    - Episode title (alt suggestions)
    - Summary (short + long)
    - Chapters with timestamps
    - 1 long-form blog article per episode
    - Show notes / key takeaways
    - SEO meta (title, description, keywords, OG image)

**Publishing**

- Creator workspace dashboard
- One auto-generated public site per workspace
- Default theme + 2 alternates (light, dark, magazine)
- Episode page with embedded video, transcript, chapters, article
- Custom subdomain (`{slug}.podcast2web.app`)

**Account & Billing**

- Email / OAuth sign-in
- Free tier + paid plans (monthly minutes of audio processed)
- Stripe-based subscriptions

### 4.2 Advanced Features (Phase 2 — months 5–9)

- Custom domains with automated TLS
- Multiple article variants per episode (newsletter, LinkedIn post, Twitter thread)
- Auto-generated short-form vertical clips (with subtitles)
- Multi-language transcription + translation
- Speaker profiles, guest pages
- Embedded subscribe / lead-capture widgets
- Audience analytics dashboard
- Team roles (Owner, Editor, Reviewer, Viewer)
- Webhooks & public API

### 4.3 Future / Phase 3

- Native podcast RSS hosting + distribution to Apple/Spotify
- Monetization: paywalled posts, sponsorships marketplace
- AI "Ask the show" chatbot per podcast
- Marketplace of community themes & prompt packs
- White-label / agency mode

---

## 5. Non-Functional Requirements

|Category|Requirement|
|---|---|
|**Performance**|Article generation < 5 min for a 60-min episode|
|**Scalability**|10k creators, 1M episodes, horizontal scale per service|
|**Availability**|99.9% uptime for public sites; 99.5% for processing|
|**Security**|SOC2-ready, encrypted at rest + in transit, RBAC, audit logs|
|**Privacy**|GDPR & CCPA compliant; data residency options|
|**Cost efficiency**|Per-minute processing cost target < $0.15|
|**Extensibility**|Plugin SDK by Phase 3|
|**Observability**|Centralized logging, tracing, metrics|

---

## 6. System Architecture (Microservices)

### 6.1 Architectural Principles

- **Domain-driven microservices**, communicating via async messaging for long-running jobs and synchronous REST/gRPC for queries.
- **Event-driven pipeline** for the AI workflow (a video upload triggers a chain of independent services).
- **Multi-tenant by design** — every entity is scoped by `workspace_id`.
- **Stateless services** behind a gateway; state lives in Postgres, object storage, and a vector DB.
- **CQRS-lite**: write paths through services; read paths optimized via cached projections for public sites.

### 6.2 Service Map

                        `┌────────────────────────────┐                          │      Web / Mobile UI       │                          │  (Creator Dashboard + CMS) │                          └─────────────┬──────────────┘                                        │ HTTPS                                        ▼                            ┌──────────────────────┐                            │     API Gateway      │                            │  Auth, RBAC, Rate    │                            └─────────┬────────────┘          ┌───────────────────┬───────┼──────────────┬───────────────────┐          ▼                   ▼       ▼              ▼                   ▼   ┌─────────────┐    ┌─────────────┐ │       ┌─────────────┐    ┌──────────────┐   │   Identity  │    │   Workspace │ │       │   Billing   │    │  Analytics   │   │   Service   │    │   Service   │ │       │   Service   │    │   Service    │   └─────────────┘    └─────────────┘ │       └─────────────┘    └──────────────┘                                      │                                      ▼                            ┌────────────────────┐                            │  Media Ingestion   │◄── Upload / YouTube / RSS                            │      Service       │                            └─────────┬──────────┘                                      │ event: media.uploaded                                      ▼                         ┌─────────────────────────┐                         │   Message Broker (NATS  │                         │   / Kafka / RabbitMQ)   │                         └─────────────────────────┘              ┌──────────────┬──────────┴──────────┬───────────────┐              ▼              ▼                     ▼               ▼       ┌────────────┐ ┌────────────┐       ┌────────────┐  ┌────────────┐       │Transcription│ │  AI Blog   │       │  Clip /    │  │   SEO &    │       │  Service   │ │ Generator  │       │ Highlight  │  │  Metadata  │       │  (Whisper) │ │  Service   │       │  Service   │  │  Service   │       └────────────┘ └────────────┘       └────────────┘  └────────────┘              │              │                     │               │              └──────────────┴─────────┬───────────┴───────────────┘                                       ▼                            ┌────────────────────┐                            │  Content Service   │  (Episodes, Articles)                            └─────────┬──────────┘                                      │                                      ▼                            ┌────────────────────┐                            │ Site Renderer /    │ ──► Public Sites (CDN)                            │ Publishing Service │                            └────────────────────┘  Cross-cutting:    - Object Storage (S3/R2) for media + assets    - PostgreSQL for relational data    - Vector DB (pgvector / Pinecone) for semantic search    - Redis for cache & job state    - Observability: OpenTelemetry → Grafana / Loki / Tempo`

### 6.3 Microservices Breakdown

|Service|Responsibility|Sync API|Async Events Consumed|Async Events Produced|
|---|---|---|---|---|
|**API Gateway**|Auth, routing, rate limiting, request shaping|REST/GraphQL|—|—|
|**Identity Service**|Users, sessions, OAuth, MFA|REST|—|`user.created`|
|**Workspace Service**|Workspaces, members, roles, domains|REST|`user.created`|`workspace.created`|
|**Billing Service**|Plans, Stripe, usage metering, quotas|REST|`usage.*`|`plan.changed`|
|**Media Ingestion**|Uploads, YouTube/RSS imports, validation|REST|—|`media.uploaded`|
|**Transcription Service**|ASR (Whisper / Deepgram), diarization, alignment|gRPC|`media.uploaded`|`transcript.ready`|
|**AI Blog Generator**|Article(s), summary, chapters, show notes|gRPC|`transcript.ready`|`article.generated`|
|**Clip Service**|Highlight detection, vertical clip rendering, captions|gRPC|`transcript.ready`|`clips.ready`|
|**SEO & Metadata**|Titles, meta, OG image, schema.org, sitemap|gRPC|`article.generated`|`seo.ready`|
|**Content Service**|CRUD on episodes, articles, assets, versioning|REST|`*.ready`|`content.published`|
|**Site Renderer**|SSG/SSR of public sites, theme system, CDN cache invalidation|REST|`content.published`|`site.deployed`|
|**Analytics Service**|Page views, listens, conversion funnels|REST|`site.event.*`|—|
|**Notification Service**|Email, in-app, webhooks|REST|many|—|
|**Search Service**|Full-text + semantic search across episodes/articles|REST|`content.published`|—|

---

## 7. UML Diagrams (Textual Form)

### 7.1 Use Case Diagram

`   Actors:    Creator, Publisher (extends Creator), TeamMember, EndVisitor, AdminOps, BillingSystem(Stripe)  Use Cases:    Creator        → (Sign Up) (Sign In) (Create Workspace) (Upload Episode)                     (Import from YouTube) (Review Article) (Edit Article)                     (Configure Theme) (Connect Custom Domain) (View Analytics)                     (Manage Subscription)    Publisher      → (Manage Multiple Shows) (Invite Team Member) (Assign Role)    TeamMember     → (Edit Article) (Approve Episode)    EndVisitor     → (Browse Site) (Read Article) (Watch Video) (Search) (Subscribe)    AdminOps       → (Monitor Pipeline) (Reprocess Episode) (Manage Quotas)    BillingSystem  → (Send Payment Events) (Validate Subscription)  Includes:    (Upload Episode) includes (Validate Media) includes (Charge Usage)    (Review Article) includes (Generate Article) which includes (Transcribe Audio)  Extends:    (Generate Article) extended by (Generate Newsletter) (Generate Social Posts)    (Connect Custom Domain) extended by (Issue TLS Certificate)   `

### 7.2 Class Diagram (Core Domain)

`   User    id: UUID    email: string    name: string    hashedPassword: string?    createdAt: datetime    -- methods: authenticate(), updateProfile()  Workspace    id: UUID    ownerId: UUID  -> User    name: string    slug: string    plan: enum(Free, Pro, Studio, Enterprise)    customDomain: string?    createdAt: datetime    -- methods: addMember(), changePlan(), getUsage()  Membership    id: UUID    workspaceId: UUID -> Workspace    userId: UUID      -> User    role: enum(Owner, Admin, Editor, Reviewer, Viewer)  Show    id: UUID    workspaceId: UUID -> Workspace    title: string    description: text    branding: JSON (logo, colors, fonts)    themeId: UUID -> Theme  Episode    id: UUID    showId: UUID -> Show    title: string    status: enum(Uploaded, Transcribing, Generating, Ready, Published, Failed)    durationSec: int    videoAssetId: UUID -> Asset    publishedAt: datetime?    createdAt: datetime    -- methods: republish(), regenerateArticle()  Asset    id: UUID    workspaceId: UUID    type: enum(Video, Audio, Image, Caption, Clip)    storageKey: string    mimeType: string    sizeBytes: bigint  Transcript    id: UUID    episodeId: UUID -> Episode    language: string    segments: JSON [{start, end, speaker, text}]    embeddingsId: UUID? -> VectorIndex  Article    id: UUID    episodeId: UUID -> Episode    variant: enum(LongForm, Newsletter, LinkedIn, Twitter)    title: string    bodyMarkdown: text    seoMeta: JSON    status: enum(Draft, Approved, Published)    version: int    authorAI: string (model + prompt id)  Clip    id: UUID    episodeId: UUID    startSec: float    endSec: float    videoAssetId: UUID -> Asset    captionAssetId: UUID -> Asset    platform: enum(TikTok, Reels, Shorts, Generic)  Theme    id: UUID    name: string    templateRef: string    config: JSON  Subscription    id: UUID    workspaceId: UUID    stripeId: string    plan: string    status: enum(Active, PastDue, Canceled)    renewsAt: datetime  UsageRecord    id: UUID    workspaceId: UUID    metric: enum(MinutesProcessed, ArticlesGenerated, StorageGB, Bandwidth)    amount: float    occurredAt: datetime  Relationships:    User 1..* — 1..* Workspace  (via Membership)    Workspace 1 — * Show    Show 1 — * Episode    Episode 1 — 1 Transcript    Episode 1 — * Article    Episode 1 — * Clip    Workspace 1 — * Subscription    Workspace 1 — * UsageRecord   `

### 7.3 Sequence Diagram — "Episode → Published Article"

`   Creator → API Gateway: POST /episodes (multipart upload)  API Gateway → Identity Service: validate JWT  API Gateway → Media Ingestion Service: createUpload(file)  Media Ingestion Service → Object Storage: PUT video.mp4  Media Ingestion Service → Content Service: create Episode(status=Uploaded)  Media Ingestion Service → Broker: emit media.uploaded  Broker → Transcription Service: media.uploaded  Transcription Service → Object Storage: GET video.mp4  Transcription Service → ASR Engine (Whisper): transcribe  Transcription Service → Content Service: save Transcript  Transcription Service → Broker: emit transcript.ready  Broker → AI Blog Generator: transcript.ready  AI Blog Generator → LLM Provider: prompt(template + transcript)  AI Blog Generator → Content Service: save Article(draft)  AI Blog Generator → Broker: emit article.generated  Broker → SEO Service: article.generated  SEO Service → LLM Provider: meta + OG image  SEO Service → Content Service: attach SEO  SEO Service → Broker: emit seo.ready  Broker → Clip Service: transcript.ready (parallel branch)  Clip Service → ML Highlight Model: detect peaks  Clip Service → Renderer: cut + caption clips  Clip Service → Content Service: save Clips  Content Service → Broker: emit content.published (when approved)  Broker → Site Renderer: content.published  Site Renderer → Object Storage / CDN: write static pages  Site Renderer → Notification Service: emit site.deployed  Notification Service → Creator: email "Your episode is live"  EndVisitor → CDN: GET /episodes/{slug}   `

### 7.4 Component Diagram (High-level)

`   [Frontend SPA] ──► [API Gateway] ──► [Domain Services Cluster]                                         │                    ┌────────────────────┼────────────────────┐                    ▼                    ▼                    ▼             [Postgres Cluster]   [Object Storage]      [Vector DB]                    ▲                    ▲                    ▲                    └─────── [Message Broker] ────────────────┘                                         │                                         ▼                              [Worker Pools per AI service]   `

---

## 8. Database Schema (High-level)

Single logical Postgres with per-service schemas (or schema-per-service in a managed Postgres). Cross-service joins are forbidden — services expose APIs.

**Schema: identity**

- `users(id, email, name, hashed_password, created_at)`
- `oauth_accounts(id, user_id, provider, provider_id)`
- `sessions(id, user_id, expires_at)`

**Schema: workspace**

- `workspaces(id, owner_id, name, slug, plan, custom_domain, created_at)`
- `memberships(id, workspace_id, user_id, role)`
- `invitations(id, workspace_id, email, role, token, expires_at)`

**Schema: content**

- `shows(id, workspace_id, title, description, branding_json, theme_id)`
- `episodes(id, show_id, title, status, duration_sec, video_asset_id, published_at, created_at)`
- `transcripts(id, episode_id, language, segments_jsonb)`
- `articles(id, episode_id, variant, title, body_md, seo_jsonb, status, version, author_ai)`
- `clips(id, episode_id, start_sec, end_sec, video_asset_id, caption_asset_id, platform)`
- `assets(id, workspace_id, type, storage_key, mime_type, size_bytes)`
- `themes(id, name, template_ref, config_jsonb)`

**Schema: billing**

- `subscriptions(id, workspace_id, stripe_id, plan, status, renews_at)`
- `usage_records(id, workspace_id, metric, amount, occurred_at)`
- `invoices(id, workspace_id, amount, currency, paid_at)`

**Schema: analytics** (also mirrored to a columnar store like ClickHouse)

- `page_views(id, site_id, path, ts, country, ua)`
- `media_events(id, episode_id, event_type, ts, position_sec)`

**Vector index**

- `episode_chunks(id, episode_id, chunk_text, embedding vector(1536))` (pgvector)

**Indexes & policies**

- `workspace_id` indexed on every multi-tenant table
- Row-Level Security policies enforce tenant isolation
- Soft delete (`deleted_at`) on user-facing entities
- Append-only `outbox` table per service for transactional event publishing

---

## 9. Recommended Tech Stack

|Layer|Choice|Rationale|
|---|---|---|
|**Frontend (Dashboard)**|React + Vite + TypeScript, TanStack Query, shadcn/ui, Tailwind|Fast DX, mature, themeable|
|**Public Sites Renderer**|Next.js (SSG + ISR) on edge|SEO + per-tenant fast pages|
|**API Gateway**|NestJS or Fastify + GraphQL/REST, behind Kong / Envoy|Auth, throttling, schema|
|**Microservices**|Node.js (NestJS) for I/O-heavy + Python (FastAPI) for AI services|Right tool per workload|
|**AI / ASR**|Whisper (self-hosted) + Deepgram (fallback), GPT-class LLM via provider abstraction|Cost & quality balance|
|**Vector / Search**|pgvector + Meilisearch|Semantic + keyword|
|**DB**|PostgreSQL (managed, e.g. Neon / RDS)|Reliable, RLS support|
|**Cache**|Redis|Sessions, rate limit, queues state|
|**Message Broker**|NATS JetStream or Kafka|Event pipeline|
|**Storage / CDN**|S3 / Cloudflare R2 + Cloudflare CDN|Cheap egress, global|
|**Auth**|Clerk or Auth0 (OIDC) + JWT to services|Time-to-market|
|**Billing**|Stripe Billing + metered usage|Standard SaaS|
|**Container / Orchestration**|Docker + Kubernetes (EKS/GKE) or Fly.io for early stage|Scale per-service|
|**CI/CD**|GitHub Actions + ArgoCD|GitOps|
|**Observability**|OpenTelemetry → Grafana / Loki / Tempo / Prometheus + Sentry|Full-stack visibility|
|**IaC**|Terraform + Helm|Reproducible infra|
|**Secrets**|Vault / cloud KMS|Compliance|

---

## 10. End-to-End Data Flow

1. **Capture** — Creator uploads MP4 (or pastes a YouTube URL).
2. **Ingestion** — Media Ingestion Service stores the original in object storage, normalizes to MP3 + low-bitrate proxy MP4, records the `Episode` in `Uploaded` state, and emits `media.uploaded`.
3. **Transcription** — Transcription Service consumes the event, runs ASR with diarization, writes a structured `Transcript`, and emits `transcript.ready`.
4. **AI Generation (parallel fan-out)**
    - Blog Generator produces long-form article + summary + chapters.
    - Clip Service detects highlights and renders vertical clips with burned-in captions.
    - SEO Service produces title variants, meta, OG image, schema.org JSON-LD.
5. **Aggregation** — Content Service stores all artifacts, links them to the Episode, and marks status `Ready`.
6. **Review** — Creator opens the dashboard, edits if desired, hits "Publish" (or auto-publish per workspace setting).
7. **Publishing** — Site Renderer regenerates affected pages via incremental static regeneration, pushes to CDN, invalidates caches, emits `site.deployed`.
8. **Distribution** — Notification Service emails the creator + optionally posts clips to connected social accounts.
9. **Analytics loop** — Public site beacons feed Analytics Service; insights surface back in the dashboard and influence future AI suggestions (e.g., topic recommendations).

---

## 11. UI / UX Structure

### 11.1 Creator Dashboard (authenticated app)

- **Sidebar:** Shows · Episodes · Articles · Clips · Site · Audience · Billing · Settings
- **Top bar:** workspace switcher, search, notifications, profile
- **Pages**
    - **Home** — pipeline status, recent activity, quick "New Episode"
    - **Show detail** — list of episodes, branding, theme picker
    - **Episode workspace** — tabs: Overview · Transcript · Article · Clips · SEO · Publish
    - **Site editor** — theme, navigation, pages, custom domain
    - **Audience** — traffic, top episodes, conversions, geography
    - **Team** — members, roles, invitations
    - **Billing** — plan, usage gauges, invoices

### 11.2 Public Podcast Website (per-tenant)

- **Home** — featured episode, latest episodes, hosts
- **Episode page** — video player, chapter navigation, full article, transcript with click-to-seek, related episodes, share + subscribe
- **Article archive** — searchable, filterable by tag/host/season
- **About / Hosts / Contact**
- **Subscribe** — RSS, Apple, Spotify, email

### 11.3 UX Principles

- Pipeline transparency: every async step shows state and ETA.
- "Trust but edit": AI output is editable inline, never hidden.
- Mobile-first creator dashboard for on-the-go review.
- Accessibility (WCAG 2.1 AA): captions, keyboard nav, reduced motion.

---

## 12. Security, Compliance, Multi-Tenancy

- **AuthN:** OIDC via Clerk/Auth0; service-to-service via signed JWT (mTLS in cluster).
- **AuthZ:** RBAC at Workspace level + ABAC for asset ownership.
- **Tenant isolation:** every row scoped by `workspace_id`; Postgres Row-Level Security enforced; per-tenant signed URLs for storage.
- **Encryption:** TLS 1.3 in transit; AES-256 at rest; envelope encryption for PII.
- **PII:** transcripts may contain PII — region pinning + retention controls.
- **Compliance roadmap:** GDPR (DSAR endpoints), CCPA, SOC 2 Type I → Type II.
- **Abuse controls:** content moderation pass on AI outputs; copyright check for imported YouTube content.
- **Audit log:** append-only event log per workspace.

---

## 13. Scalability & Deployment Strategy

### 13.1 Scaling Tactics

- **Stateless services** behind an autoscaler (HPA on CPU + queue depth).
- **AI workers** scale on queue lag (`transcription_lag`, `generation_lag`).
- **Storage**: object store is infinite; Postgres scales vertically first, then partitions by `workspace_id` and reads via replicas; analytics offloaded to ClickHouse.
- **Caching**: per-tenant CDN caching for public sites; Redis for hot reads.
- **Cost control**: cheap-tier ASR for free plan, premium ASR for paid; LLM router chooses model by plan & length.

### 13.2 Environments

- `dev` (per developer namespace), `staging`, `prod`, `prod-eu` for data residency.

### 13.3 Deployment Pipeline

1. PR → CI: lint, unit tests, contract tests against OpenAPI.
2. Merge → image build, SBOM, vulnerability scan.
3. ArgoCD syncs Helm charts to staging.
4. Smoke + integration tests in staging.
5. Progressive delivery to prod (canary 5% → 25% → 100%) with auto-rollback on SLO breach.

### 13.4 Disaster Recovery

- Postgres PITR (point-in-time recovery), 30-day window.
- Cross-region replication of object storage.
- RPO ≤ 15 min, RTO ≤ 1 h for production.

---

## 14. KPIs & Success Metrics

**Product**

- Time from upload → published site < 10 min (p95)
- Article approval rate without edits > 60%
- Weekly active creators / total creators > 40%

**Business**

- CAC payback < 6 months
- Net revenue retention > 115%
- Gross margin > 70% after AI cost optimization

**Reliability**

- 99.9% uptime on public sites
- < 1% pipeline failure rate per episode

---

## 15. Roadmap (High-level)

|Quarter|Milestone|
|---|---|
|**Q1**|MVP: ingestion, transcription, single article, default theme, Stripe|
|**Q2**|Custom domains, multi-variant articles, clips, analytics|
|**Q3**|Multi-language, team roles, public API, theme marketplace|
|**Q4**|"Ask the show" chatbot, monetization, white-label|

---

## 16. Risks & Mitigations

|Risk|Mitigation|
|---|---|
|AI cost spikes per long episode|Tiered models + caching + chunked summarization|
|ASR quality on multi-speaker / accented audio|Diarization + premium fallback + manual edit UI|
|Copyright on imported YouTube content|Ownership attestation + content fingerprinting|
|Vendor lock-in (LLM/ASR)|Provider abstraction layer + multi-vendor routing|
|Multi-tenant data leak|RLS + automated policy tests in CI|
|Scaling Postgres|Early partitioning by `workspace_id`, read replicas, move analytics to ClickHouse|

---

## 17. Glossary

- **Workspace** — top-level tenant owning shows, sites, billing.
- **Show** — a podcast brand (a workspace can host several).
- **Episode** — one video/audio recording and all generated artifacts.
- **Variant** — alternate AI-generated formats of the same article.
- **Clip** — short-form vertical video derived from an episode.
- **Site** — the public, themed website rendered per show or per workspace.