### 📅 The Daily Ritual (First 45 Minutes)
**Do not skip this.** The JD demands "Excellent problem solving."
* **Platform:** LeetCode or NeetCode.io.
* **Focus:** Do 2 Medium problems daily. Do not waste time on Hard problems yet; you need volume and pattern recognition.
* **Key Patterns to Cover:** Sliding Window, Two Pointers, BFS/DFS (Trees/Graphs), Hash Maps.

---

### 📅 The Curriculum

#### Day 1: The V8 Engine & Advanced JavaScript
*Goal: Prove you understand the "Why" behind the code, not just the syntax.*

* **Theory (1.5 hrs):**
    * **The Event Loop:** Microtasks vs. Macrotasks. Explain `process.nextTick()` vs `setImmediate()`.
    * **Scope & Closures:** Deep dive into lexical scoping and memory leaks.
    * **Prototypes:** Prototypal inheritance vs. Class-based inheritance (ES6 classes).
    * **Async Patterns:** Promises, `async/await`, and error handling strategies in async flows.
* **Practice (1.5 hrs):**
    * Implement a simplified `Promise.all` or `debounce/throttle` function from scratch.
    * Refactor a nested callback hell snippet into clean `async/await` code.

#### Day 2: Modern React Architecture (Not just "How-to")
*Goal: "Complex and modern React/Vue applications." Move beyond `setState`.*

* **Theory (1.5 hrs):**
    * **Reconciliation:** How the Virtual DOM diffing algorithm works. Keys and why index keys are bad.
    * **Hooks Under the Hood:** Rules of Hooks, `useMemo` vs. `useCallback` (and when *not* to use them), Custom Hooks.
    * **State Management:** Context API performance pitfalls vs. Redux Toolkit/Zustand.
    * **Server-Side Rendering (SSR):** Conceptually understand Next.js vs. Client-Side Rendering (CSR).
* **Practice (1.5 hrs):**
    * Build a "Data Grid" component: Fetch data, render a table, implement client-side sorting/filtering using Hooks. Focus on preventing unnecessary re-renders (use `React.memo`).

#### Day 3: Node.js at Scale & API Design
*Goal: "Highly scalable projects."*

* **Theory (1.5 hrs):**
    * **Node.js Architecture:** Single-threaded nature, Worker Threads, Clustering (scaling Node across cores).
    * **Streams & Buffers:** Handling large files/data without blowing up RAM.
    * **API Design:** REST maturity levels vs. GraphQL. Status codes (200, 201, 400, 401, 403, 500).
* **Practice (1.5 hrs):**
    * Create a simple Express server. Implement a stream that reads a large file and pipes it to the response (simulate a CSV download).
    * Implement basic Middleware for error handling and logging.

#### Day 4: Web Security & The Ecosystem
*Goal: "Excellent understanding of various web security methods."*

* **Theory (1.5 hrs):**
    * **OWASP Top 10:** Specifically XSS (Cross-Site Scripting), CSRF (Cross-Site Request Forgery), and SQL Injection.
    * **Authentication:** Session-based vs. JWT. Where to store tokens (HttpOnly Cookies vs. LocalStorage).
    * **CORS:** How the browser pre-flight check works.
    * **HTTPS:** TLS handshake basics.
* **Practice (1.5 hrs):**
    * Secure the Express server from Day 3: Add `helmet`, implement Rate Limiting, and sanitize inputs.
    * Simulate a CSRF attack conceptually and write down how you'd prevent it (SameSite cookies).

#### Day 5: System Design & AWS Basics
*Goal: "Modern cloud technologies."*

* **Theory (1.5 hrs):**
    * **The Scalability Cube:** Horizontal vs. Vertical scaling. Load Balancers (L4 vs L7).
    * **AWS Core:** EC2 (Compute), S3 (Storage), RDS/DynamoDB (Database), Lambda (Serverless), VPC (Networking basics).
    * **Caching:** Redis/Memcached strategies (Cache-aside, Write-through).
    * **Microservices:** Pros/Cons vs. Monolith. Communication (HTTP vs. Message Queues like RabbitMQ/SQS).
* **Practice (1.5 hrs):**
    * **Whiteboard Challenge:** Design a URL Shortener (Bit.ly). Draw the diagram. Where does the Load Balancer go? Which DB do you use? How do you handle 1M writes/sec?

#### Day 6: Design Patterns & Clean Code
*Goal: "Good understanding of OOP and Design Patterns."*

* **Theory (1.5 hrs):**
    * **SOLID Principles:** Map these to JavaScript/TypeScript scenarios.
    * **Design Patterns:** Singleton (DB connections), Factory (creating components), Observer (Event Emitters), Strategy (auth strategies).
    * **Code Quality:** Dependency Injection in Node.js (Inversion of Control).
* **Practice (1.5 hrs):**
    * Refactor a "God Object" (a generic messy class) into smaller classes using the Factory or Strategy pattern.

#### Day 7: Integration & The "Manager to Maker" Pivot
*Goal: Prepare your narrative.*

* **Full Stack Mini-App (3 hours):**
    * Build a simple "Task Manager" with a React Frontend and Node Backend.
    * *Constraint:* It must be secure (JWT), scalable (code structure), and use a database (MongoDB or Postgres).
    * *Why?* To get the "rust" off your fingers. You need to verify you can configure Webpack/Vite and Express without Googling every single line.
* **The Narrative (1 hour):**
    * Prepare answers for: "Why are you moving back to individual contribution?"
    * *Answer strategy:* "I love architecture and building. I’ve enjoyed leading, but I miss the hands-on impact of shipping scalable code."