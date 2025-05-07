# dummyjson-postman-demo

This Postman project demonstrates a **pipeline-style API test flow** against the public [https://dummyjson.com/docs/auth](https://dummyjson.com/docs/auth) API, orchestrated in a single `.json` file.. It automatically:

1. **Fetches** all users (`GET /users`)  
2. **Iterates** over each user’s credentials  
3. **Logs in** (`POST /auth/login`), extracts and stores tokens  
4. **Validates** the login (`GET /auth/me`)  
5. **Refreshes** the access token (`POST /auth/refresh`)  
6. **Loops** until all users have been processed  

> Designed as a fully **self-contained** Collection using **Collection Variables** only—no external Environments required.

---

## Design Philosophy

1. **Pipeline Control with `pm.execution.setNextRequest`**  
   Uses Postman’s flow-control API in each request’s **Tests** script to dynamically drive the next step, rather than a static runner sequence.

2. **Single Source of Truth via Collection Variables**  
   All state (current index, user credentials array, tokens) lives in Collection-level variables.  You can import this Collection anywhere without extra setup.

3. **Built-in Assertions for Quality**  
   Each step includes `pm.test` assertions on status codes, JSON structure and required fields.  Failures surface in the Runner report without silent errors.

4. **Clear Separation of Concerns**  
   - **Pre-request Scripts** – prepare request payloads (login, refresh)  
   - **Tests Scripts** – assert responses, store tokens, control flow  

---

## Prerequisites

- **Postman v11+** (Collection Runner must support `pm.execution.setNextRequest`)  
- Internet access to **[https://dummyjson.com](https://dummyjson.com)**

---

## Import & Run

1. **Import Collection**  
   In Postman → **Import** → Select the provided `dummyjson.postman_collection.json`.

2. **Run the Pipeline**  
   - Select **dummyjson** in the sidebar  
   - Click **Run** (Collection Runner)  
   - Ensure **Iterations: 1**, **Keep variable values** and **Stop run if an error occurs** are checked  
   - Press **Run dummyjson**  

   The Runner will execute:
   ```
   GET /users
   → POST /auth/login
   → GET /auth/me
   → POST /auth/refresh
   → (repeat login→me→refresh until all users done)
   ```

---

## Run via Newman (CLI)

To run this collection via CLI and generate a **visual HTML report**, use [Newman](https://github.com/postmanlabs/newman), the official Postman runner:

### 1. Install Newman and HTML Reporter

```bash
npm install -g newman
npm install -g newman-reporter-html
````

### 2. Run the Collection with HTML Output

```bash
newman run dummyjson.postman_collection.json -r html
```

This will generate a file `newman/newman-run-report.html` in the current directory.

---

## Collection Structure

```
dummyjson
├── Variables
│   ├── base_url = https://dummyjson.com
│   ├── user_credentials = []
│   ├── current_index = 0
│   ├── access_token = ""
│   └── refresh_token = ""
├── GET /users            # fetches all users, initializes credentials array
├── POST /auth/login      # logs in one user based on current_index
├── GET /auth/me          # validates login, asserts user info
└── POST /auth/refresh    # refreshes token, increments index or ends run
```

Each request has:

- **Pre-request Script** → builds request body or initializes variables  
- **Tests** → `pm.test` assertions + `pm.execution.setNextRequest(...)`  

---

## Troubleshooting

- **No flow jumps?** Ensure you run **via Runner**, not individual Send buttons.  
- **Variable stale?** The Collection’s first request (`GET /users`) resets `current_index` and clears previous arrays.  
- **Assertion failures?** Check Postman Console (`Ctrl+Alt+C` / `⌘+⌥+C`) for detailed logs and test results.
