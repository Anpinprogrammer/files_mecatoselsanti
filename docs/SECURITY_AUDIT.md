### TASK: Comprehensive Cybersecurity Audit & Hardening

Please perform a thorough security audit across the `backend`, `admin-frontend`, and `print-server` packages to identify and remediate potential security vulnerabilities.

Focus areas:

1. Rate Limiting & Brute Force Protection:
   - Verify rate limiting middleware on `/api/auth/login`, `/api/auth/pin-login`, and `/api/auth/forgot-password`.
   - Ensure `trust proxy` is configured if deployed behind a reverse proxy (e.g., Render).

2. NoSQL Injection & Input Sanitization:
   - Audit all MongoDB/Mongoose queries to ensure user input (`req.body`, `req.query`, `req.params`) is validated and sanitized (e.g., using `Zod` or `express-mongo-sanitize`).

3. Auth, Cookies & CORS:
   - Verify JWT cookies use `httpOnly: true`, `secure: true`, and proper `sameSite` policies.
   - Ensure CORS settings reject wildcard origins and match production domain variables.

4. HTTP Security Headers:
   - Ensure `helmet` middleware is registered in the main Express application entry points.

5. Print Server Isolation:
   - Confirm `mecatos-print-server` binds strictly to `127.0.0.1` and validates print payload structures.

6. Update Documentation:
   - Record security measures and configuration flags in `CLAUDE.md`.