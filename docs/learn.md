# Concepts to brush up on

These are the ideas behind opening the loan application form to external applicants, and the ones the form already depends on. Each section says what the idea is, where it shows up in this project, and a question to check yourself. Work through Part 1 first: it covers the public-page plan.

---

## Part 1: Security for a public form

### 1. Authentication vs. authorization

**Authentication** answers "who are you?" (a login, a one-time code, a secret link). **Authorization** answers "what are you allowed to do?" (read this record, not that one).

**In this project:** staff authenticate with a Saturn login, and Saturn's roles authorize them. Applicants won't log in, so a **link token** authenticates them. Authorization then comes from the Path's checks: "this token belongs to this application, and this record belongs to this application".

**Check yourself:** an applicant has a valid link. What stops them reading someone else's Party record?

Read: [MDN, HTTP authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)

### 2. Bearer tokens and API keys

A **bearer token** is a secret string sent in the `Authorization: Bearer <token>` header. Whoever holds ("bears") it gets access, with no other proof. An **API key** is a kind of bearer token issued to a program rather than a person.

**In this project:** the saved Requests use `Bearer {{secrets.…}}`. In Saturn, an API key belongs to a **User Group**, so its permissions can be limited.

**Watch out:** a bearer token **identifies the page to Saturn**, not the applicant. It doesn't give applicants access by itself, and it doesn't stop anyone using the page.

**Check yourself:** why must the token never appear in the Vue component's code?

Read: [MDN, Authorization header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization)

### 3. The server-side proxy (Secrets, Requests, `pageHttp`)

The browser can't keep a secret: anything sent to it can be read in the developer tools. So the page asks the server to make the call for it. The browser sends only a **request name** and **inputs**. The server adds the secret and calls a **fixed** URL.

**In this project:**
- **Secrets**: the token, stored encrypted on the server.
- **Requests**: an allow-list of named calls, each with a fixed URL, method and headers. `{{secrets.x}}` is filled in on the server. `{{inputs.x}}` comes from the page.
- **`pageHttp('name', inputs)`**: how the component runs one. It returns `{ status, data }`.
- Inputs can only be strings, numbers, booleans or null. Nested data has to go as a JSON string (`JSON.stringify`).
- The URL is fixed, so a record ID can't go in the URL path. That's why Saturn's own `/api/applications/<id>` endpoints can't be called directly and Paths are needed.

**Check yourself:** on a public page, who can run a saved Request? (Answer: anyone, with any inputs.)

### 4. Never trust the browser

Anything the page does can be skipped or faked. Anyone can open the developer console and call `pageHttp` or `/api/_paths/...` with their own values. Browser checks are for a smooth experience only. **Every real rule has to be checked again on the server.**

**In this project:** required fields, amounts, document rules, "status is Draft", and "this ID belongs to this application" all have to be checked inside the Path. Fields like `status`, `application_number`, `submitted_at` and `interest_rate` must be stripped on the server.

**Check yourself:** the form hides the interest-rate field. Is that enough to stop an applicant setting it?

### 5. Guessable IDs (IDOR)

**IDOR** (Insecure Direct Object Reference) is when a system takes a record ID from the user and doesn't check they're allowed to touch it. Changing `id=123` to `id=124` then shows someone else's data. It's one of the most common web security bugs.

**In this project:** this is why the gateway Path keeps an **own-records list** (`intake_record_ids`) on each application and rejects any ID not on it. It's also why the **application number** must never be used in a link: it's sequential and easy to guess.

**Check yourself:** why is checking "the ID exists" not the same as checking "the ID belongs to this applicant"?

Read: [OWASP, IDOR prevention](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html) · [OWASP API Security Top 10](https://owasp.org/API-Security/)

### 6. Secret links (capability URLs)

A link that grants access on its own, like `?application=…&token=…`, is a **capability URL**. Rules for making them safe:
- **Long and random.** `crypto.randomBytes(32).toString("hex")` gives 64 hex characters, far too many to guess.
- **Expire.** Set `access_token_expires_at` and check it on every call.
- **Stop working when done.** Clear the token on submit, as MediPal clears its token after a decision.
- **Know where it leaks.** URLs end up in browser history, server logs, and sometimes other sites through the `Referer` header. A forwarded email gives the link away too. A one-time code (Saturn's `Challenge`) can be added later for extra protection.

**Check yourself:** an applicant forwards their link to a friend. What can the friend do, and what would limit that?

Read: [W3C, Good practices for capability URLs](https://www.w3.org/TR/capability-urls/)

### 7. Hashing secrets and timing-safe comparison

**Hashing** turns a value into a fixed fingerprint that can't be reversed (SHA-256). Store the **hash** of the link token, not the token itself. If the database leaks, the stored values can't be used as links. To check a token, hash what arrived and compare the hashes. **Encryption** is different: it can be reversed with a key.

A plain `===` comparison can stop at the first different character, which in theory leaks timing information. `crypto.timingSafeEqual` always takes the same time.

**In this project:** MediPal stores its token in plain text. The loan form should store `sha256(token)`, using `_util.crypto.createHash("sha256")`, and compare with `_util.crypto.timingSafeEqual`.

**Check yourself:** why is a hash enough to check a token, even though it can't be turned back into the token?

Read: [Node.js crypto: randomBytes, createHash, timingSafeEqual](https://nodejs.org/api/crypto.html)

### 8. Least privilege

Give every key, user and process **only the permissions it needs**. Then a mistake or a leak does less damage.

**In this project:** create a "Public loan intake" User Group with only the needed resources, and issue the Path's key to that group, not an admin key. Reference data (loan types, lookups) is read-only in the gateway.

### 9. Abuse of public endpoints

Anything public will eventually be scripted: spam, guessing, oversized uploads.
- **Rate limiting**: cap requests per IP or per token.
- **Bot checks that don't slow people down**: anyone can start an application on the public "apply" page, so bots can too. A *honeypot* is a hidden field people never see but bots fill in; the server drops anything that fills it. A CAPTCHA or one-time code would block more, but was turned down to keep the form easy for anyone.
- **Cleanup**: abandoned drafts pile up; expire them and remove or archive them on a schedule.
- **Upload limits**: check file type and size on the server.
- **Kill switch**: a WorkflowTrigger's `status` can be set to inactive to switch a Path off at once.

### 10. Personal data on a public page

The form holds names, NIS numbers, IDs, income and bank-related details. Mistakes here can mean data-protection breaches, not just bugs. Keep the data sent back to the page to the minimum, log failures without logging personal data, and get sign-off from whoever is responsible for data protection before going live.

---

## Part 2: How the web and Saturn work

### 11. HTTP basics

- **Method**: `GET` (read), `POST` (create or run something), `PUT` (update), `DELETE`. GET and HEAD can't have a body.
- **Where data goes**: in the **path** (`/applications/123`), the **query** (`?status=open`), the **headers** (`Authorization`), or the **body** (JSON).
- **Status codes**: 200 OK, 400 bad request, 401 not authenticated, 403 not allowed, 404 not found.
- **Saturn quirk**: errors often come back as **HTTP 200** with `{"status":"FAILURE", "message": …}`. Always check `data.status`, not only whether the call threw. The `_paths` test returned exactly this for an unknown Path.

Read: [MDN, an overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)

### 12. JSON, multipart uploads and base64

- **JSON** is the normal request body.
- **multipart/form-data** is how browsers upload files. Saturn's upload endpoint uses it with the fields `file`, `name`, `saturn_file_type`, `tags`, `meta_data`.
- **base64** turns a file into text so it can go inside JSON. It makes the file **about 33% bigger**, and servers have a **maximum body size**. That's why base64 uploads through a Path need testing with a real scanned document.

Read: [MDN, Base64](https://developer.mozilla.org/en-US/docs/Glossary/Base64) · [MDN, FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData)

### 13. REST and CRUD

**CRUD** means Create, Read, Update, Delete. **REST** APIs map these to URLs and methods: `POST /api/assets` creates, `GET /api/assets/{id}` reads, `PUT …/{id}` updates, `DELETE …/{id}` deletes.

**In this project:** `Resource.get/list/create/update/delete` are wrappers around these. The gateway Path (Option A) keeps the same five operations so the form's save logic barely changes.

**Check yourself:** why can't a saved Request call `PUT /api/applications/{id}` for a different `id` each time?

### 14. Reading API documentation (OpenAPI / Swagger)

`https://loanapi.saturn.gd/docs` is a **Swagger UI** page. The real description is the JSON it loads, `/docs/spec`. It lists every endpoint, its parameters, and the auth scheme (`Bearer` in the `Authorization` header). Learn to find an endpoint, its method, its path parameters, and its body fields.

Read: [Swagger, what is OpenAPI?](https://swagger.io/docs/specification/about/)

### 15. Saturn Paths and workflows

A **Path** is configuration, not backend code:
- **WorkflowTrigger** with `trigger_provider: "http"`, `configuration.path` (the name in `/api/_paths/<name>`), `actions` (which workflow action runs), and `status` (active or not).
- **Workflow action**: a `task` that runs **commands** in `series`.
- **Reusable command**: called from an action with a `reference` task. Shared logic (like submit) belongs here.
- **Code operator**: `async function (flow, { _util, _store }) { … }`.
  - `flow` / `_flow`: the request body and the values passed along.
  - `_store`: values to pass to later commands.
  - `_util.spice.models.<Resource>`: read and create records (`new Application().get({ id })`).
  - `_util.crypto`, `_util.moment`, `_util.axios`.
  - The last operator's return value is the response.
- **Built-in operators**: `property_updater` (update a record), `send_email`, `upload_to_cloud`/`save_to_cloud`.
- **Errors**: `throw new Error("…")` stops the action. Check how the page receives it.

**Check yourself:** Saturn needs the bearer token to run a Path, but the public page lets any visitor use it. So where must the applicant's token check live?

**Reading Saturn's replies:** without a bearer token, Saturn answers `Route not found` for every route, even real ones like `/api/applications`. So "Route not found" without a token doesn't prove the route is missing.

### 16. Document databases (Couchbase)

Saturn's backend (Spice.js, built on Koa) stores records in **Couchbase**, a document database. Each record is a JSON document with an ID, not a row in a table. Relationships are stored as **lists of IDs** (`Application.parties`, `asset_ids`, `Party.ids`) rather than database joins.

**In this project:** that's why the **save order** matters (create the Party before the ID that points to it), and why restoring reads the ID lists. It's also how `intake_record_ids` works: one more list of IDs on the Application.

Read: [Couchbase, data modelling](https://docs.couchbase.com/server/current/learn/data/document-data-model.html)

---

## Part 3: JavaScript and Vue used in the form

### 17. Async JavaScript

- `async` / `await`: wait for a request without freezing the page.
- `try` / `catch` / `finally`: handle failures and always reset `loading`.
- `Promise.all`: run independent saves side by side. The form uses it for people, assets and liabilities.
- Error objects come in many shapes (`error.response.data.message`, `error.message`). Log the whole object before guessing.

Read: [MDN, async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) · [MDN, Promise.all](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)

### 18. Vue component basics used here

- **Props**: values passed into a component. `pageHttp` and `queryParams` arrive as props on a Saturn custom page.
- **`mounted()`**: runs once the page is shown. It's where the link check or draft load happens. Wait for it before showing the form.
- **`computed`**: values worked out from other values (like `requestId` read from the URL).
- **`URLSearchParams`**: reads `?application=…&token=…` from the address bar.

Read: [Vue, props](https://vuejs.org/guide/components/props.html) · [Vue, lifecycle hooks](https://vuejs.org/guide/essentials/lifecycle.html) · [MDN, URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams)

### 19. localStorage is not security

`localStorage` stays in one browser on one device, and the user can read and change it. It's fine for convenience (remembering a draft or a step) but never for deciding access. MediPal's "link already used" flag only hides the buttons in that one browser. The server's record is what counts.

Keeping a **secret** there (like the applicant's link token) is a different question. The main risks:
- **Shared devices.** The next person on a family phone or a cyber-café computer opens the page and lands in the draft.
- **Any script on the same site can read it**, including code from another custom page on the same Saturn address, a bad library, or a cross-site scripting (XSS) bug.

What keeps the risk acceptable: the token only opens one draft, it expires, it stops working on submit, and a "Finish later" button lets the applicant remove it from the browser.

Read: [MDN, localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)

### 20. Saturn quirks already learned (see `claude.md`, "Lessons learned")

- No spread syntax (`...`) in Saturn component code.
- Saturn rejects `null`, so empty values are left out (`withoutEmptyValues()`).
- `create` can reply with a FAILURE instead of throwing.
- Dates: Grenada is UTC−4, so `toISOString()` gives the wrong "today" after 8 pm.

These apply to the new public-mode code too. The spread rule may also apply inside Code operators: MediPal's operators use `...flow`, so it seems to work there, but test before relying on it.

---

## Suggested order

1. Sections 1–5: what keeps applicants' data safe.
2. Sections 3, 11 and 15: how a click on the page reaches Saturn and comes back.
3. Sections 6–7: the link token design.
4. The rest as you meet them while building.
