
## 📐 OpenAPI Ruleset

All rules and conventions for writing OpenAPI documents are defined in:

```
OpenAPI_RULESET.md
```

The ruleset covers topics such as:

* File structure and naming
* Path and operation conventions
* Request/response schemas
* Error response standards

---

## ⚙️ Code Generation

### React Client Generation

We use **openapi-generator-cli** to generate a TypeScript API client for React applications.

**Generator:** `typescript-axios`

### 📦 Prerequisites

* Node.js (LTS recommended)
* npm

---

### ▶️ How to Generate Code

From the project root:

```bash
cd example-frontend
npm run generate-api
```

This command:

* Reads the OpenAPI specification
* Generates TypeScript models and API clients
* Places the output into predefined folders

---

### 📁 Generated Folder Structure

```text
src/
 ├── api/        # Generated API clients (Axios-based)
 │   └── *.ts
 ├── models/     # Generated TypeScript models / DTOs
 │   └── *.ts
 └── index.ts    # Barrel export
```

> ⚠️ **Do not manually edit generated files.**  
> Any changes will be overwritten the next time code is generated.

---

## 📎 Related Documents

* `openapi/openapi.yaml` – Main OpenAPI entry file
* `OpenAPI_RULESET.md` – API design rules and conventions

---

Happy coding ✨ OpenAPI makes teams faster and safer!
