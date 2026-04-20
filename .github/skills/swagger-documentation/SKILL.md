---
name: swagger-documentation
description: Guidelines and examples for writing OpenAPI/Swagger documentation in this repository. Emphasizes defining component schemas in Resource classes and fully documenting all controller parameters (including those originating from Request validators and middleware).
---

# Swagger / OpenAPI Documentation Skill ✅

**What this skill helps accomplish**
- Teach consistent, complete, and maintainable Swagger/OpenAPI documentation practices for this Laravel codebase.

**When to use this skill**
- When adding or updating API endpoints, Request validators, Resources, middleware that affect request inputs, or response shapes.

---

## Principles (short & strict) 🔧
- **Always define component schemas in Resource classes.** Resources are the single source of truth for response shapes and can be referenced across the spec.
- **Document every known parameter** the controller action uses: parameters from the route, query/body params from FormRequest validators, and any middleware-introduced parameters (headers, query flags, pagination, locale, etc.).
- **Prefer referencing schemas** (components -> schemas) instead of duplicating JSON schemas inline.
- **Make validators the source for parameter metadata** (type, required, enum/allowed values, min/max). If validator data is ambiguous, add clarifying descriptions in the documentation.

---

## Step-by-step procedure ✍️
1. Resource Schema
   - Add an OpenAPI schema (`@OA\Schema`) in the Resource that fully describes the JSON shape.
   - Include types, descriptions, example values when helpful.
2. Controller endpoint doc-block
   - Add an `@OA\Get`/`@OA\Post`/etc. annotation above the controller method.
   - Reference the Resource schema for `responses` (e.g., `@OA\JsonContent(ref="#/components/schemas/PersonResource")`).
3. Parameters from Request validators
   - Extract parameters and constraints from the FormRequest `rules()` and create corresponding `@OA\Parameter` entries (query/path/header) or an `@OA\RequestBody` JSON schema.
   - Mark required parameters based on the validator's `required` rules.
4. Middleware parameters
   - Explicitly document parameters injected or required by middleware (e.g., `Authorization` header for auth middleware, `X-Request-ID`, `Accept-Language` locale, or pagination defaults).
5. Responses and error cases
   - Document standard success responses and at least the common error statuses (400, 401, 403, 404, 422, 500), referencing error response schemas.

---

## OpenAPI data types & mapping from Laravel validators 💡

**OpenAPI primitive types & common formats**
- **string** — use `type: string` and prefer the most specific `format` when applicable. Common string formats (per OpenAPI):
  - `date` — full-date (RFC3339) (e.g., `2026-01-09`)
  - `date-time` — date-time (RFC3339) (e.g., `2026-01-09T10:00:00Z`)
  - `email` — use `format: email` for email addresses
  - `uuid` — use `format: uuid` for UUIDs
  - `uri` — use `format: uri` for URIs
  - `hostname` — use `format: hostname`
  - `ipv4`, `ipv6` — use for IP address strings
  - `password` — use `format: password` for sensitive strings
  - `byte` — base64-encoded strings (useful for binary payloads encoded as strings)
  - `binary` — for raw binary file uploads (use in request bodies with `multipart/form-data`)
  - When validators use `regex`, add `pattern: "<pattern>"` and note any modifier flags in the `description` (OpenAPI `pattern` does not accept modifiers).
- **number** — formats: `float`, `double`
- **integer** — formats: `int32`, `int64`
- **boolean** — boolean values
- **array** — `items` defines the item schema
- **object** — use `properties` or `additionalProperties` for free-form objects

**Practical guidance**
- Prefer `format` where it adds clarity (e.g., `format: email` or `format: uuid`). The `format` values should follow the OpenAPI specification for consistency (see https://swagger.io/docs/specification/v3_0/data-models/data-types/#strings).
- Choose `date` vs `date-time` based on the validator or expected string pattern (use `date-time` for timestamps with time component).
- For file uploads, document the request as `multipart/form-data` with the file field `schema: { type: string, format: binary }`.
- When copying regex patterns from Laravel validators, strip PHP delimiters and flags and paste the raw pattern into `pattern`; include a brief note describing the original modifiers if relevant.
- For security-sensitive fields (passwords, tokens), add `format: password` and include a short note in `description` reminding readers not to log or expose such values.

**Detailed mapping rules (Laravel validator rule → OpenAPI schema)**

- String-related
  - `string` → `type: string`
  - `email` → `type: string`, `format: email`
  - `url` / `active_url` → `type: string`, `format: uri`
  - `uuid` → `type: string`, `format: uuid`
  - `date` / `date_format:...` → `type: string`, `format: date` or `date-time` depending on format
  - `regex:/.../` → `type: string`, `pattern: "<pattern>"`
    - **Copy the regex pattern from the validator into the schema's `pattern`** — remove the PHP delimiters and flags (e.g., `/^abc$/i` → `^abc$`, and note modifiers like `i` may require adding a note in `description` since OpenAPI `pattern` does not accept flags).
  - `min:N` / `max:N` (for strings) → `minLength` / `maxLength`
  - `size:N` (for strings) → `minLength: N` and `maxLength: N`

- Numeric
  - `integer` / `numeric` → `type: integer` or `type: number` (choose number for floats)
  - `digits:N` / `digits_between` → consider `pattern` or `minimum`/`maximum` when possible
  - `min:N` / `max:N` (numeric) → `minimum` / `maximum`
  - `between:min,max` → `minimum: min`, `maximum: max`

- Boolean
  - `boolean` → `type: boolean`

- Enum / set
  - `in:option1,option2` → `enum: ["option1", "option2"]`

- Arrays
  - `array` with `*` rules → `type: array`, `items: { ... }` where the `*` rules define `items` schema
  - `min:N` / `max:N` / `size:N` (on arrays) → `minItems`, `maxItems`, `minItems=maxItems` respectively
  - `distinct` → `uniqueItems: true`

- Objects / JSON
  - `json` or associative arrays → `type: object` with `properties` defined; if shape is unknown use `additionalProperties: true`

- Files
  - `file`, `image` → document as `type: string`, `format: binary` in `requestBody` (or use `multipart/form-data` with `schema: { type: string, format: binary }`)

**Other rules & flags**
- `nullable` → add `nullable: true` to the schema
- `sometimes` → treat as optional (do not add to `required`) but document clearly in the parameter description
- `present` → document as required to be present (even if null)

**Practical notes**
- When a validator uses `regex`, **copy the pattern exactly** (without PHP delimiter characters and flags) into the OpenAPI `pattern`. If the pattern relies on modifiers (e.g., case-insensitive `/.../i`), add a clarifying note in the `description` because `pattern` itself does not accept modifiers.
- For `in:` rules, include the full enum list as `enum` and add brief descriptions for each value where helpful.
- Prefer defining complex response/request shapes in a `@OA\Schema` in a Resource and referencing it with `$ref`, rather than describing large inline JSON schemas in controllers.

> Tip: if the validator includes `sometimes` or `nullable`, reflect that in `required` and `nullable` accordingly. Also add `pattern`, `minLength`, `maxLength`, `minimum`, `maximum`, `enum`, `minItems`, `maxItems`, or `uniqueItems` where validator rules indicate them.

---

## Middleware parameters & docs ⚠️
- Document headers the middleware expects (e.g., `Authorization` header for token-based auth) as `in="header"` parameters.
- **Always include an `Accept` header parameter** to ensure JSON responses are returned by endpoints that might otherwise redirect. Add it as an `in="header"` parameter like this:

```
*     @OA\Parameter(
*         name="Accept",
*         in="header",
*         description="**REQUIRED**: Must be 'application/json' to receive JSON responses. Without this header, authentication errors may result in redirect responses instead of JSON.",
*         required=true,
*
*         @OA\Schema(type="string", enum={"application/json"}, example="application/json")
*     ),
```

  This ensures consistent behavior for authentication/middleware edge-cases where clients might receive non-JSON redirects without the header.
- Document any global query/headers added by middleware (e.g., feature flags, locale `Accept-Language`) so consumers know the full contract.

---

## Checklist before PR ✅
- [ ] Resource schema exists and is referenced from controller docs
- [ ] All parameters from the FormRequest are documented (type, required, description)
- [ ] Regex patterns from validators are copied into parameter/schema `pattern`
- [ ] Middleware headers/params are documented (including the `Accept` header to ensure JSON responses)
- [ ] Responses include examples and reference the Resource schema
- [ ] Error responses are documented (422 validation, 401 auth, etc.)

---

---

## Small conventions for this repo ✨
- Put `@OA\Schema` for response objects in the Resource class file.
- Use `ref` to reference the Resource schema (`#/components/schemas/YourResourceName`).
- Use `description` fields for any parameter that could be ambiguous.

---

## Quick example (Controller flow)
1. FormRequest defines `rules()` for `name`, `age`, `body`.
2. Controller method docblock lists `@OA\Parameter` for `age` (query) and `@OA\RequestBody` for `body` if POST/PUT.
3. Controller doc references `PersonResource` for response `200`.

---

*End of skill.*