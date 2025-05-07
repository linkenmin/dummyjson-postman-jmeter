# dummyjson-postman-jmeter

This is a demonstration of an **end-to-end** API test pipeline using both **Postman** and **JMeter** against [https://dummyjson.com/docs/auth](https://dummyjson.com/docs/auth).

## Overview

This repo contains:

```

dummyjson-postman-jmeter/
├── README.md
├── postman/
│   ├── dummyjson.postman_collection.json
│   └── README.md
└── jmeter/
    ├── dummyjson-auth.jmx
    └── README.md

````

- **Postman** folder holds a self-contained Collection that:
  - Fetches all users (`GET /users`)
  - Iterates each credential (`current_index`)
  - Logs in (`POST /auth/login`) and extracts tokens
  - Validates (`GET /auth/me`)
  - Refreshes (`POST /auth/refresh`)
  - Loops until every user is processed
  - See `postman/README.md` for details.


- **JMeter** folder holds a JMX Test Plan that:
  - Uses a **setUp Thread Group** to fetch users & write `users.csv`
  - Dynamically sets **threadCount** from that CSV
  - Assigns each thread to one user (Once-Only Controller)
  - Loops through login -> me -> refresh with assertions & timers
  - See `jmeter/README.md` for usage and configuration.