# Remix + Postgres Starter — App

The application code behind the [`recipe-sim-starter-template`](https://github.com/fxck/recipe-sim-starter-template) Zerops recipe. A [Remix](https://remix.run) app with a [PostgreSQL](https://www.postgresql.org/) database, ready to deploy on [Zerops](https://zerops.io).

<!-- #ZEROPS_EXTRACT_START:intro# -->
A minimal Remix + Postgres app: server-rendered routes, a Prisma data layer, and session-cookie auth. The whole thing builds with Vite and runs as a single Node process.
<!-- #ZEROPS_EXTRACT_END:intro# -->

## Integration guide

<!-- #ZEROPS_EXTRACT_START:integration-guide# -->
Already have a Remix app? You don't have to adopt this template wholesale — here's how to run *your* app on Zerops by adding the same pieces.

### 1. Add a `zerops.yaml`
Drop a `zerops.yaml` at your repo root describing how to build and run the app:

```yaml
zerops:
  - setup: app
    build:
      base: nodejs@22
      buildCommands:
        - npm ci
        - npm run build
      deployFiles: [ build, public, package.json, node_modules ]
    run:
      base: nodejs@22
      ports:
        - port: 3000
          httpSupport: true
      start: npm run start
```

### 2. Read the database connection from the environment
Zerops injects the database connection as variables named after the service hostname (`db`). Build `DATABASE_URL` from them instead of hardcoding:

```ts
// db.server.ts
const url = process.env.DATABASE_URL
  ?? `postgresql://${process.env.db_user}:${process.env.db_password}@${process.env.db_hostname}:5432/${process.env.db_dbName}`;
```

> [!TIP]
> Reference services by hostname, never by IP. Containers move; the hostname is stable across restarts and scaling.

### 3. Wire `SESSION_SECRET`
The recipe sets `SESSION_SECRET` at the project level (machine-generated), so just read `process.env.SESSION_SECRET` in `createCookieSessionStorage`.

### 4. Point a service at your repo
In the import yaml, add `buildFromGit: https://github.com/<you>/<your-app>` and a `zeropsSetup` that matches a setup in your `zerops.yaml`. Push to build.

> [!NOTE]
> That's the whole integration: a build/run descriptor, env-derived connections, and a service entry. Everything else — domains, scaling, backups — is configured in the Zerops GUI, not in your code.
<!-- #ZEROPS_EXTRACT_END:integration-guide# -->

## Knowledge Base

<!-- #ZEROPS_EXTRACT_START:knowledge-base# -->
### Prisma migrations on deploy
Run `prisma migrate deploy` from the `run.initCommands` hook (not at build time) so migrations execute against the live database after the container starts, not against a build-time sandbox.

### Vite build output
The Remix Vite build emits a server bundle plus static assets. Deploy both (`build` + `public`); Zerops serves the assets and runs the server bundle as the app process.
<!-- #ZEROPS_EXTRACT_END:knowledge-base# -->

---

Need help? Join the [Zerops Discord community](https://discord.gg/zeropsio).
