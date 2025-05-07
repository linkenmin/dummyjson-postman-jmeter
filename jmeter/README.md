# dummyjson-jmeter-demo

This JMeter test plan demonstrates a **pipeline-style API test flow** against the public [https://dummyjson.com/docs/auth](https://dummyjson.com/docs/auth) API, orchestrated in a single `.jmx` file. It automatically:

1. **Fetches** all users (`GET /users`) and writes them to a CSV file
2. **Iterates** over each user credential via **CSV Data Set Config**
3. **Logs in** (`POST /auth/login`), extracts and stores tokens
4. **Validates** the login (`GET /auth/me`)
5. **Refreshes** the access token (`POST /auth/refresh`)
6. **Loops** until all users have been processed

> Thread count is dynamically set to the number of lines in the generated CSV; loop count is configurable via the `-JloopCount=` property. An HTML report (`index.html`) can be generated via JMeter’s CLI.

---

## Design Philosophy

1. **Single JMX, All-in-One**
   The entire flow—data fetching, credential iteration, authentication, and validation—is encapsulated in one JMeter test plan. No external orchestration required.

2. **Dynamic Thread Count**
   A preliminary HTTP Request (`GET /users`) writes the user list to `users.csv`. The **Thread Group** uses `${__P(threadCount)}` so that thread count matches the number of CSV entries.

3. **Configurable Loop Count**
   Each thread repeats its sequence based on the `${__P(loopCount)}` property, passed via `-JloopCount=` at runtime—allowing flexible test depths.

4. **Automated Reporting**
   Leverages JMeter’s non-GUI mode and built-in report generator to output an `index.html` for easy analysis.

---

## Prerequisites

* **Apache JMeter 5.4+** installed and on your `PATH`
* **Java 8+** runtime environment
* Internet access to **[https://dummyjson.com](https://dummyjson.com)**

---

## Import & Run

1. **Place Test Plan**
   Save `dummyjson-auth.jmx` and the included `user.properties` (if any) in your working directory.

2. **Run in Non-GUI Mode**
   Use JMeter’s CLI to execute and generate results + HTML report:

   ```bash
   jmeter \
     -n \
     -t dummyjson-auth.jmx \
     -JloopCount=5 \
     -l results.jtl \
     -e \
     -o report-output
   ```

   * `-JloopCount=5` sets each thread to loop 5 times (override as needed)
   * `results.jtl` is the raw XML/CSV result file
   * `report-output/index.html` is the interactive dashboard

3. **Generate Standalone Report**
   If you already have `results.jtl`, regenerate the HTML report separately:

   ```bash
   jmeter \
     -g results.jtl \
     -o report-output
   ```

---

## JMX Structure

```
dummyjson-auth.jmx
└── Test Plan (dummyjson-auth)
    ├── HTTP Request Defaults (base_url = https://dummyjson.com)
    ├── setUp Thread Group
    │   └── HTTP Request - GET /users
    │       └── JSR223 PostProcessor (write users.csv)
    └── Thread Group (Load Test - User Pipeline)
        ├── Once Only Controller
        │   ├── CSV Data Set Config - users.csv
        │   └── JSR223 Sampler (Set user for each thread)
        ├── Loop Controller (loops = ${__P(loopCount)})
        │   ├── HTTP Request - POST /auth/login
        │   │   └── JSON Extractor (access_token, refresh_token)
        │   ├── HTTP Request - GET /auth/me
        │   └── HTTP Request - POST /auth/refresh
        └── Listeners
            ├── View Results Tree
            ├── Summary Report
            ├── Aggregate Report
            └── Simple Data Writer
```

* **setUp Thread Group** runs once to fetch all users and outputs `users.csv` for the main test.
* **Thread Group** is configured with dynamic **threadCount** matching CSV lines and runs the auth sequence per user.
* **Once Only Controller** ensures CSV and variable initialization only occur once per thread.
* **Loop Controller** repeats the login→me→refresh steps based on `-JloopCount=`.

## Troubleshooting

* **users.csv not found?** Make sure the Setup Thread Group completes successfully before the main Thread Group starts. You can verify this in the CLI logs or results.
* **Thread group doesn't start?** Ensure `threadCount` is correctly passed in via properties or auto-calculated in your JSR223 script.
* **Loop count not working?** Double check you’ve added `-JloopCount=N` when running from CLI.
* **Missing tokens or login failures?** Open the `results.jtl` file or view the generated `index.html` to inspect failed samplers and extract response bodies.
* **Script errors in JSR223?** Enable debug logging in JMeter or check for missing libraries or incorrect variable syntax.
* **HTML report not generated?** Confirm that JMeter was executed with `-e -o` flags and that the `results.jtl` file has valid entries.
