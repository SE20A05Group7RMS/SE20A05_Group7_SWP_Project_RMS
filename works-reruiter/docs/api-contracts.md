# API Contracts — Works Reruiter

**Generated:** 2026-05-20  
**Gateway:** `services/gateway/` (:3001)  
**Swagger:** `http://localhost:3001/api` (auto-generated)

---

## Contract Architecture

All API contracts originate from `@wr/contracts` and flow to two consumers:

```
@wr/contracts (Zod schemas)
    ├──→ Gateway (class-validator DTOs, validated at HTTP layer)
    └──→ Webapp (Zod .parse() at fetch layer)
```

**Strict rules:**
- `@wr/contracts` is the SINGLE SOURCE OF TRUTH for all types/enums
- Gateway DTOs must mirror contract schemas exactly
- Microservices never validate inbound TCP messages (already validated by gateway)

## Gateway Endpoints by Controller

### Health (`/health`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/health` | ❌ | Health check |

### Identity (`/api/identity/`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/auth/register` | ❌ | Register new user |
| `POST` | `/auth/login` | ❌ | Login → JWT |
| `GET` | `/users/me` | ✅ | Current user profile |
| `POST` | `/organizations` | ✅ | Create organization |
| `GET` | `/organizations` | ✅ | List user's organizations |
| `POST` | `/organizations/:id/departments` | ✅ | Create department |
| `GET` | `/organizations/:id/departments` | ✅ | List departments |
| `POST` | `/organizations/:id/approval-chains` | ✅ | Configure approval chain |

### Recruiting (`/api/recruiting/`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/hiring-requests` | ✅ | Create hiring request (DRAFT) |
| `PATCH` | `/hiring-requests/:id/submit` | ✅ | Submit for approval |
| `PATCH` | `/hiring-requests/:id/approve` | ✅ | Approve request |
| `PATCH` | `/hiring-requests/:id/reject` | ✅ | Reject request |
| `PATCH` | `/hiring-requests/:id/revise` | ✅ | Request revision |
| `POST` | `/roles` | ✅ | Create role from approved request |
| `GET` | `/roles` | ✅ | List roles |
| `GET` | `/roles/:id` | ✅ | Get role details |
| `POST` | `/roles/:id/publish` | ✅ | Publish role for candidates |
| `POST` | `/applications` | ✅ | Apply to role |
| `POST` | `/evaluations` | ✅ | Trigger evaluation run |
| `GET` | `/evaluations/:id` | ✅ | Get evaluation results |
| `POST` | `/talent-search` | ✅ | Hybrid search (RRF) |

### Profiles (`/api/profiles/`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/candidate-profiles` | ✅ | Create candidate profile |
| `GET` | `/candidate-profiles/me` | ✅ | Get own profile |
| `PATCH` | `/candidate-profiles/:id` | ✅ | Update profile |
| `POST` | `/documents` | ✅ | Upload CV document |
| `GET` | `/documents/:id` | ✅ | Get document + parse status |
| `DELETE` | `/documents/:id` | ✅ | Delete document |

### Review (`/api/review/`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/feedback` | ✅ | Submit feedback (agree/challenge/comment) |
| `GET` | `/feedback/:applicationId` | ✅ | List feedback for application |
| `POST` | `/packets` | ✅ | Generate candidate packet |
| `GET` | `/packets/:id` | ✅ | Get candidate packet |

## Message Patterns (TCP)

Each gateway endpoint maps to a `@MessagePattern` in the target microservice:

| Pattern | Service | Description |
|---------|---------|-------------|
| `identity.auth.register` | Identity | User registration |
| `identity.auth.login` | Identity | Login |
| `identity.users.me` | Identity | Get current user |
| `identity.organizations.create` | Identity | Create org |
| `identity.organizations.list` | Identity | List orgs |
| `identity.departments.create` | Identity | Create department |
| `identity.departments.list` | Identity | List departments |
| `identity.approval-chains.create` | Identity | Configure approval chain |
| `recruiting.hiring-requests.create` | Recruiting | Create hiring request |
| `recruiting.hiring-requests.submit` | Recruiting | Submit for approval |
| `recruiting.hiring-requests.approve` | Recruiting | Approve |
| `recruiting.hiring-requests.reject` | Recruiting | Reject |
| `recruiting.roles.create` | Recruiting | Create role |
| `recruiting.roles.list` | Recruiting | List roles |
| `recruiting.roles.publish` | Recruiting | Publish role |
| `recruiting.applications.create` | Recruiting | Apply |
| `recruiting.evaluations.run` | Recruiting | Trigger evaluation |
| `recruiting.talent-search.search` | Recruiting | Hybrid search |
| `profiles.candidates.create` | Profiles | Create profile |
| `profiles.candidates.get` | Profiles | Get profile |
| `profiles.candidates.update` | Profiles | Update profile |
| `profiles.documents.upload` | Profiles | Upload CV |
| `profiles.documents.get` | Profiles | Get document |
| `profiles.documents.delete` | Profiles | Delete document |
| `review.feedback.submit` | Review | Submit feedback |
| `review.feedback.list` | Review | List feedback |
| `review.packets.generate` | Review | Generate packet |
| `review.packets.get` | Review | Get packet |

## BullMQ Job Types

| Queue | Job Type | Trigger | Description |
|-------|----------|---------|-------------|
| `cv-parse` | `parse-cv` | Document upload | Extract text, parse CV |
| `cv-parse` | `extract-evidence` | Parse complete | Extract evidence records |
| `embedding` | `generate-embeddings` | Evidence extracted | MiniLM-L6-v2 vectors |
| `evaluation` | `run-evaluation` | Evaluation triggered | Hybrid match + score |

## Error Response Format

```json
{
  "statusCode": 400,
  "message": "Validation failed",
  "error": "Bad Request",
  "details": [
    { "field": "email", "constraints": { "isEmail": "email must be valid" } }
  ]
}
```
