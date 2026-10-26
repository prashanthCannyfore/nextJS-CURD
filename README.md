# Users CRUD

A simple user directory built with Next.js, Express, PostgreSQL, and TypeORM migrations.

| Part | Technology | Hosting |
| --- | --- | --- |
| Frontend | Next.js | Vercel |
| API | Express | Vercel Services |
| Database | PostgreSQL with TypeORM migrations | Supabase or Neon |

The Vercel project serves the frontend at `/` and the API at `/api`. The database is hosted separately. See [DEPLOYMENT.md](DEPLOYMENT.md) for deployment instructions and [CONFIGURATION.md](CONFIGURATION.md) for local setup.

This starter has no authentication or authorization. Use disposable demo data until the write API is protected; CORS and user roles do not restrict access.
