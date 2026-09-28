# Secrets Management

No credentials, API keys, or connection strings are hardcoded in the
current repository source.

## Local development

Secrets are stored in a local .env file, excluded from git via
.gitignore, and injected into the Juice Shop container at runtime via
docker-compose.yml's env_file directive.

## CI/CD pipeline

The same values are stored as GitHub Actions encrypted secrets under
the repository's Settings, referenced in workflow steps via the
secrets context (e.g. `${{ secrets.JWT_SECRET }}`), never appearing as
plain text in any workflow file or log output.