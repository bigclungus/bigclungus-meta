## Parallel Development Approach

Per koole__ (2026-03-25): build the new TypeScript server **concurrently** with serve.py, not as an inline replacement. This allows behavioral comparison throughout development.

**Setup:**
- `hello-world-ts/` runs on port 8081
- `website.service` keeps serving serve.py on :8080 throughout development
- New `hello-world-ts.service` systemd unit for the TypeScript server
- Both services run simultaneously; no cutover until TS server is verified

**Verification approach:**
- Hit the same endpoint on both servers, diff the responses
- Flag behavioral differences as bugs or intentional improvements before cutover
- Cutover = swap which port Cloudflare tunnel routes to (one-line config change)

**Rollback:**
- If the TS server has issues after cutover, flip the Cloudflare route back to :8080
- serve.py stays intact until both framers confirm TS server is stable
