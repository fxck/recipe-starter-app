# Remix + Postgres Starter — App

The application code behind the [`recipe-starter-template`](https://github.com/fxck/recipe-starter-template) Zerops recipe. A [Remix](https://remix.run) app with a [PostgreSQL](https://www.postgresql.org/) database, ready to deploy on [Zerops](https://zerops.io).

<!-- #ZEROPS_EXTRACT_START:intro# -->
A minimal Remix + Postgres app: server-rendered routes, a Prisma data layer, and session-cookie auth. The whole thing builds with Vite and runs as a single Node process.
<!-- #ZEROPS_EXTRACT_END:intro# -->

## Integration guide

<!-- #ZEROPS_EXTRACT_START:integration-guide# -->

Already have a Remix app? Here's how to run it on Zerops without adopting this template wholesale.

### 1. Add a `zerops.yml`
Drop a `zerops.yml` at your repo root describing how to build and run the app:

```yaml
zerops:
  - setup: app
    build:
      base: nodejs@22
      buildCommands:
        - npm ci
        - npm run build
      deployFiles:
        - build
        - public
        - package.json
        - node_modules
    run:
      base: nodejs@22
      ports:
        - port: 3000
          httpSupport: true
      start: npm run start
```

### 2. Read the database connection from the environment
Zerops injects the database connection details as environment variables named after the service hostname (`db`). Build your `DATABASE_URL` from them rather than hardcoding:

```ts
// db.server.ts
const url = process.env.DATABASE_URL
  ?? `postgresql://${process.env.db_user}:${process.env.db_password}@${process.env.db_hostname}:5432/${process.env.db_dbName}`;
```

### 3. Wire `SESSION_SECRET`
Remix's cookie session storage needs a secret. The recipe sets `SESSION_SECRET` at the project level (machine-generated), so just read `process.env.SESSION_SECRET` in `createCookieSessionStorage`.

### 4. Add the service to your import yaml
Point a service at your repo with `buildFromGit: https://github.com/<you>/<your-app>` and you're done — push to build.

<!-- #ZEROPS_EXTRACT_END:integration-guide# -->

## Knowledge Base

<!-- #ZEROPS_EXTRACT_START:knowledge-base# -->

### Prisma migrations on deploy
Run `prisma migrate deploy` from the `run.initCommands` hook (not at build time) so migrations execute against the live database after the container starts, not against a build-time sandbox.

<!-- #ZEROPS_EXTRACT_END:knowledge-base# -->

---

Need help? Join the [Zerops Discord community](https://discord.gg/zeropsio).
