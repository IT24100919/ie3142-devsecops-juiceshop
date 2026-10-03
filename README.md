# IE3142 DevOps Security — DevSecOps Pipeline for OWASP Juice Shop
 
Group 08 | IE3142 — DevOps Security | SLIIT
 
A containerised deployment of OWASP Juice Shop (v20.2.0) behind an Nginx reverse proxy, with four demonstrated vulnerabilities fixed and a GitHub Actions CI/CD pipeline enforcing four security gates (SAST, dependency scanning, secrets scanning, container image scanning).
 
## Team
 
| Student ID | Name | Role |
|---|---|---|
| IT24100919 | Senevirathne G.G.J.D. (Jalitha) | AppSec Lead |
| IT24102945 | Madusara P.H.G.S. (Sahan) | Architecture & Container Lead |
| IT24103424 | Weerasinghe I.K.V.K. (Venura) | DevOps / CI-CD Lead |
| IT24102070 | Pihara H.G.T. (Teena) | Secrets & Compliance Lead |
 
## Architecture
 
```
Internet --> Nginx reverse proxy (port 8080, public) --> Juice Shop app (internal Docker network only)
```
 
Nginx is the only service exposed to the host. Juice Shop is reachable only across the internal Docker network, forming the project's trust boundary. See `docs/architecture-diagram.png` for the full diagram.
 
## Prerequisites
 
- Docker Desktop (or Docker Engine) installed and running
- Git
Check both are installed:
```
docker --version
docker compose version
git --version
```
 
## Setup and Run
 
**1. Clone the repository.**
```
git clone https://github.com/IT24100919/ie3142-devsecops-juiceshop.git
cd ie3142-devsecops-juiceshop
```
 
**2. Create a local `.env` file** in the project root (same level as `docker-compose.yml`) with the required secret:
```
JWT_SECRET=<your-local-dev-value>
```
This file is excluded from git via `.gitignore` and is never committed. In CI, the equivalent value is stored as a GitHub Actions encrypted secret.
 
**3. Start the full stack with one command.**
```
docker compose up -d --build
```
This builds the Juice Shop image from source (`juice-shop-src/`) and starts both the Juice Shop and Nginx containers. The first build takes several minutes; subsequent builds are faster due to Docker layer caching.
 
**4. Confirm both containers are running.**
```
docker compose ps
```
You should see `juice-shop` and `nginx-proxy` both listed as running.
 
**5. Open the app.**
```
http://localhost:8080
```
The Juice Shop storefront should load. Juice Shop itself is never directly reachable on its internal port; all traffic goes through Nginx.
 
**6. Stop the stack when done.**
```
docker compose down
```
 
## Project Structure
 
```
.
├── docker-compose.yml          # Brings up the full stack with one command
├── nginx.conf                  # Reverse proxy configuration
├── .env                        # Local secrets (not committed)
├── .trivyignore                # Documented accepted-risk CVEs (see report, Section 4)
├── .gitignore
├── juice-shop-src/             # Juice Shop application source (built from source, not the prebuilt image)
│   ├── routes/login.ts         # Fixed: SQL injection (login bypass)
│   ├── routes/search.ts        # Fixed: SQL injection (UNION-based data exfiltration)
│   ├── routes/basket.ts        # Fixed: IDOR on basket endpoint
│   └── server.ts               # Fixed: public FTP directory exposure
├── .github/workflows/
│   └── pipeline.yml            # CI/CD pipeline: build, SAST, SCA, secrets scan, image scan
├── docs/
│   ├── architecture-diagram.png
│   ├── secrets-management.md
│   ├── fix-locations-notes.md
│   └── IE3142_Ethical_Clearance_Form.pdf
├── evidence/
│   ├── before/                 # Screenshots: each vulnerability exploited on unmodified app
│   ├── after/                  # Screenshots: same exploit blocked after the fix
│   └── pipeline/                # Screenshots: CI/CD gates blocking real findings
└── IE3142_Technical_Report.pdf
```
 
## Security Gates (CI/CD)
 
The pipeline runs automatically on every push to `main`/`cicd-pipeline` and on every pull request into `main`.
 
| Gate | Tool | Enforcement |
|---|---|---|
| SAST | Semgrep | Reports findings |
| Dependency / SCA | npm audit | Reports findings |
| Secrets scanning | Gitleaks | Reports findings |
| Container image scanning | Trivy (`--severity CRITICAL`) | **Enforced — blocks the build** |
 
Trivy is the designated enforced blocking gate. See `IE3142_Technical_Report.pdf`, Section 4, for evidence of it genuinely blocking a build on real findings.
 
## Reproducing the Four Fixed Vulnerabilities
 
Full before/after exploit steps, payloads, and screenshots are documented in `IE3142_Technical_Report.pdf`, Section 3. Summary:
 
1. **Login SQL injection** — `routes/login.ts`, fixed via Sequelize parameterised query
2. **Search SQL injection** — `routes/search.ts`, fixed via Sequelize parameterised query
3. **Basket IDOR** — `routes/basket.ts`, fixed via server-side ownership check
4. **Public FTP exposure** — `server.ts`, fixed via `security.isAuthorized()` middleware
## Secrets Management
 
See `docs/secrets-management.md` for the full approach. In short: no credentials are hardcoded in the application source; secrets are provided via a local `.env` file (excluded from git) for development and via GitHub Actions encrypted secrets for CI.
 
## Ethical Clearance
 
This project uses OWASP Juice Shop, an intentionally vulnerable open-source application built for security training. All exploit demonstrations were performed locally, offline, against this authorised training target only. See `docs/IE3142_Ethical_Clearance_Form.pdf`.
 
