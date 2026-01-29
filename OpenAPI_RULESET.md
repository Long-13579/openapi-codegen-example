# OpenAPI – API Documentation Ruleset

This document defines the **official ruleset** for writing, reviewing, and maintaining the EOS API OpenAPI documentation.

All API documentation **MUST** follow these rules. PRs that violate them should be rejected.

---

## 1. General Principles

* OpenAPI is a **contract**, not a code mirror
* Documentation must be understandable **without reading backend code**
* Consistency is more important than personal preference
* Prefer clarity over cleverness

---

## 2. File & Folder Structure (Mandatory)

```text
openapi/
├─ openapi.yaml                 # Entry file (root contract)
├─ paths/                       # Operations only (NO schemas)
│  ├─ teams.yaml
│  ├─ scorecards.yaml
│  ├─ rocks.yaml
│  └─ issues.yaml
│
├─ components/
│  ├─ schemas/                  # Domain data schemas (one schema per file)
│  │  ├─ team/
│  │  │  ├─ team.yaml
│  │  │  ├─ team-create.yaml
│  │  │  └─ team-update.yaml
│  │  ├─ user/
│  │  │  ├─ user.yaml
│  │  │  └─ user-create.yaml
│  │  ├─ blog-post/
│  │  │  ├─ blog-post.yaml
│  │  │  ├─ blog-post-create.yaml
│  │  │  └─ blog-post-update.yaml
│  │  └─ common/
│  │     ├─ error-response.yaml
│  │     └─ pagination.yaml
│  │
│  ├─ request-bodies/           # RequestBody wrappers ONLY
│  │  ├─ create-team.yaml
│  │  ├─ update-team.yaml
│  │  ├─ create-blog-post.yaml
│  │  └─ update-blog-post.yaml
│  │
│  ├─ parameters/               # Reusable parameters (path/query/header) 
│  │  ├─ pagination.yaml          (optional but recommended)
│  │  └─ ids.yaml
│  │
│  ├─ responses/                # Reusable responses (optional but recommended)
│  │  ├─ bad-request.yaml
│  │  ├─ unauthorized.yaml
│  │  ├─ forbidden.yaml
│  │  └─ conflict.yaml
│  │
│  └─ security-schemes.yaml     # Auth definitions (JWT, API key, etc.)

```

### Rules:

#### **Structure & Ownership**  

❌ No schemas defined inline in `paths` files  
❌ No schemas defined inside `request-bodies`  
❌ No schemas defined inside `responses`  
❌ No reusable objects defined outside `components`   

#### **Schemas**

✅ All data structures (domain models, request models, response models, error models) MUST be defined in `components/schemas`   
✅ One schema per file   
✅ Schemas MUST be reusable and free of HTTP concerns (status codes, media types)   

#### **Request Bodies**

✅ All request payloads MUST be defined in `components/request-bodies`  
✅ Request bodies MUST only define:
  - `required`
  - `content`
  - `media type`
  - `$ref` to a schema   

❌ Request bodies MUST NOT define properties or business fields

#### **Responses**

✅ Reusable responses SHOULD be defined in `components/responses`  
✅ Response objects MUST reference schemas via `$ref`  
❌ Response files MUST NOT define schemas inline  

#### **Paths**

✅ One file per domain in `paths`  
✅ Path files MAY ONLY define:  
- operations
- parameters (by `$ref`)
- requestBody (by `$ref`)
- responses (by `$ref`)

#### **Entry Point**

✅ `openapi.yaml` is the single entry point  
✅ All other files MUST be transitively reachable via `$ref` from `openapi.yaml`  
✅ Direct references to every schema or component file in `openapi.yaml` are NOT required

---

## 2.1 Entry File Rules (`openapi.yaml`)

The entry file defines the **root contract** and assembles all parts.

### `openapi.yaml` MUST contain:

* `openapi` version
* `info` (title, version, description)
* `servers`
* `paths` (references only)
* `components` (references only)
* `security` (global, if applicable)

### Restrictions 

❌ Do NOT define operations inline in `openapi.yaml`  
❌ Do NOT define schemas inline in `openapi.yaml`   
❌ Do NOT define request bodies or responses inline in `openapi.yaml`

### Referencing Rules

- `openapi.yaml` MAY reference high-level component files only (e.g. `parameters`, `securitySchemes`)
- `openapi.yaml` MUST NOT reference individual schema, request-body, or response files
- All other OpenAPI files MUST be transitively reachable via $ref starting from `openapi.yaml`

### Example

```yaml
paths:
  /teams:
    $ref: ./paths/teams.yaml
  /scorecards:
    $ref: ./paths/scorecards.yaml

components:
  securitySchemes:
    jwt-bearer:
      $ref: ./components/security-schemes.yaml#/jwt-bearer
```

---

## 2.2 Path File Rules (`paths/*.yaml`)

Path files define **operations only**.

### Path files MUST:

* Define HTTP methods (`get`, `post`, `put`, `delete`)
* Reference schemas, parameters, and responses via `$ref`

### Path files MUST NOT:

* Define schemas
* Define reusable parameters
* Define reusable responses

### Correct reference usage

```yaml
get:
  parameters:
    - $ref: ../components/parameters.yaml#/TeamIdParam
  responses:
    '200':
      description: Team detail
      content:
        application/json:
          schema:
            $ref: ../components/schemas/team.yaml#/Team
    '404': 
      $ref: ../components/responses.yaml#/NotFoundResponse  
```

---

## 3. Path & Operation Rules

### Required for every operation

* `tags`
* `summary`
* `description`
* `responses`

### Summary

* One short sentence
* Verb + resource
* Example: `Get team by ID`

### Description

* Explains **behavior & rules**, not fields
* Mention permissions and constraints

❌ Bad:

```yaml
description: Returns team object
```

✅ Good:

```yaml
description: Retrieves team details visible to the current user
```

---

## 4. HTTP Method Semantics

| Method | Usage                     |
| ------ | ------------------------- |
| GET    | Read data                 |
| POST   | Create new resource       |
| PUT    | Full update               |
| PATCH  | Partial update (optional) |
| DELETE | Remove resource           |

Use correct status codes (`200`, `201`, `204`, `400`, `404`).

---

## 5. Parameter Rules

* Reusable parameters **MUST** be defined in `components/parameters/` folder
* Path parameters are **always required**
* No optional path parameters
* Query parameters are used for filtering, paging, sorting

❌ Do not inline common parameters

✅ Correct usage:

```yaml
parameters:
  - $ref: ../components/parameters.yaml#/TeamIdParam
  - $ref: ../components/parameters.yaml#/PageParam
```

---

## 6. Schema Rules

### General

* Schemas are defined **only** in `components/schemas`
* Schema name represents the **entity**, not usage
* Always define `required`
* Prefer `$ref` over duplication

### Domain Organization

- Each domain MUST have its own folder under `components/schemas`
- All schemas related to a domain MUST live in that domain’s folder
- One schema per file

**Example:**

```
components/schemas/
├─ team/
│  ├─ team.yaml
│  ├─ team-create.yaml
│  └─ team-update.yaml
├─ blog-post/
│  ├─ blog-post.yaml
│  ├─ blog-post-create.yaml
│  └─ blog-post-update.yaml
```

### Restrictions

❌ Do not define schemas in `paths` files  
❌ Do not define schemas in `request-bodies` or `responses`  
❌ Do not over-generalize schemas (e.g. `GenericResponse`, `BaseEntity`)  

### Quality Rules

✅ Keep schemas domain-focused and explicit   
✅ Separate read models and write models when they differ  
✅ Reuse schemas across requests and responses via `$ref`  

---

## 7. Request Body Rules

### Purpose

- Request bodies define how clients send data to the API.
- They describe transport structure, not business data models.

### Location & Structure

- Request bodies are defined only in components/request-bodies
- One request body per file
- File names use kebab-case

**Example**

```
components/request-bodies/
├─ create-team.yaml
├─ update-team.yaml
└─ create-blog-post.yaml
```

### Content Rules

- Request bodies MUST define only:
  - `required`
  - `content`
  - media type (e.g. `application/json`)
  - `$ref` to a schema
- Request bodies MUST reference schemas from `components/schemas`
- Request bodies MUST NOT define properties, fields, or business rules

✅ Correct example

```yaml
CreateTeamRequest:
  required: true
  content:
    application/json:
      schema:
        $ref: ../schemas/team/team-create.yaml#/TeamCreate
```

❌ Incorrect example

```yaml
CreateTeamRequest:
  content:
    application/json:
      schema:
        type: object
        properties:
          name:
            type: string
```

### Usage Rules

- Path operations MUST reference request bodies via `$ref`
- Inline request body definitions in `paths` are not allowed

❌ Bad

```yaml
requestBody:
  content:
    application/json:
      schema:
        type: object
```

✅ Good

```yaml
requestBody:
  $ref: ../components/request-bodies/create-team.yaml#/CreateTeamRequest
```

---

## 8. Response Rules 

### Success Responses

* ❌ No global `OkResponse`
* ✅ Success responses are **inline & domain-specific**
* Schema must match response shape exactly

```yaml
'200':
  description: Team detail
  content:
    application/json:
      schema:
        $ref: ../components/schemas/team.yaml#/Team
```

### Error Responses

* One global `ErrorResponse` schema
* Error responses are **reusable**

Reusable errors:

* `400` BadRequest
* `401` Unauthorized
* `403` Forbidden
* `404` NotFound
* `409` Conflict

---

## 9. Error Handling Rules

* Error schema must include:

  * `code`
  * `message`
* Error codes use `UPPER_SNAKE_CASE`

Example:

```yaml
code: VALIDATION_ERROR
message: Invalid request data
```

❌ No custom error formats per endpoint

---

## 10. Example Rules

* Examples must match schema **exactly**
* Use realistic EOS data
* Use arrays correctly (no extra wrappers)

❌ Invalid:

```yaml
example:
  data: []
```

(if schema is an array)

---

## 11. Security Rules

* Define security schemes in `components/security.yaml`
* Use JWT Bearer authentication

```yaml
security:
  - jwt-bearer: []
```

* Apply security per operation unless globally enforced

---

## 12. Review Checklist (Use in PRs)

Before approving any OpenAPI-related PR, reviewers **MUST** verify:

### 📁 Structure & Entry File

- [ ] File is placed in the correct folder (`paths`, `components/schemas`, `components/request-bodies`, `components/responses`, `components/parameters`)
- [ ] `openapi.yaml` only contains references, no inline definitions
- [ ] No schemas defined inline in `openapi.yaml`
- [ ] All new files are transitively reachable from `openapi.yaml`
- [ ] Domain files are grouped correctly (one domain → one folder)

### Operations

* [ ] `tags` present and correct
* [ ] `summary` is short and action-based
* [ ] `description` explains behavior and rules
* [ ] `operationId` is unique and meaningful

### Parameters

* [ ] Path parameters are required
* [ ] Reusable parameters referenced via `$ref`
* [ ] No pagination/sort parameters at path level

### 📤 Request Bodies

- [ ] Request bodies are defined only in `components/request-bodies`
- [ ] Request bodies do not define properties inline
- [ ] Request bodies reference schemas via `$ref`
- [ ] Path operations reference request bodies via `$ref` only

### Responses

* [ ] Success responses are inline and domain-specific
* [ ] No generic `OkResponse`
* [ ] Error responses use shared components
* [ ] Correct HTTP status codes used

### Examples

* [ ] Examples match schema shape exactly
* [ ] Array examples are not wrapped incorrectly
* [ ] Example data is realistic

### Security

* [ ] Security scheme applied correctly
* [ ] Public endpoints explicitly set `security: []`

---

## 13. Naming Conventions

### Files (all folders)

- Use kebab-case for all file names
- One logical object per file

**Examples**
- `team.yaml`
- `team-create.yaml`
- `error-response.yaml`
- `create-blog-post.yaml`
- `team-id.yaml`

### Schemas (`components/schemas`)

- Use PascalCase
- Name represents the data entity, not HTTP usage
- Avoid status-code or transport wording in schema names

**Examples**
- `Team`
- `User`
- `BlogPost`
- `BlogPostCreate`
- `ErrorResponse`

### Request Bodies (`components/request-bodies`)

- Use PascalCase
- Suffix with `Request`

**Examples**

- `CreateTeamRequest`
- `UpdateTeamRequest`
- `CreateBlogPostRequest`

### Responses (`components/responses`)

- Use PascalCase
- Suffix with `Response`
- Do NOT include HTTP status codes in the name

**Examples**

- `NotFoundResponse`
- `BadRequestResponse`
- `UnauthorizedResponse`

### Parameters (`components/parameters`)

- Use PascalCase
- Suffix with `Param`

**Examples**

- `TeamIdParam`
- `UserIdParam`
- `PageParam`
- `LimitParam`

### Security Schemes (`components/securitySchemes`)

- Use kebab-case keys
- Match the scheme name semantically

**Examples**

- `jwt-bearer`
- `api-key`

### operationId

- Use camelCase
- Format: verb + resource
- Verb MUST be explicit and consistent

**Examples**

- `getTeamById`
- `listTeams`
- `createScorecard`
- `updateBlogPost`
- `archiveTeam`

---

## 14. Versioning Rules

* API versioning is done via base path (`/v1`)
* Breaking changes require a new version
* Schema changes must be backward-compatible within the same version

❌ Do not remove fields  
❌ Do not change field meanings

---

## 15. Deprecation Rules

* Deprecated operations must be marked with `deprecated: true`
* Description must explain the replacement

```yaml
deprecated: true
description: >
  This endpoint is deprecated.
  Use `GET /v1/teams/{id}` instead.
```

---

## 16. Linting & Validation

* OpenAPI spec must pass validation before merge
* Recommended tools:
  * Swagger Editor
  * Redocly CLI

❌ Invalid specs must not be merged

---

## 17. Final Principles

> **The OpenAPI spec is a shared contract.**
>
> If it is unclear in the documentation,
> it is unclear for the frontend,
> unclear for clients,
> and unclear for future maintainers.

**Clarity, consistency, and correctness are mandatory.**

---

*End of OpenAPI Documentation Ruleset*
