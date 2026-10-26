# Deploy the full app to Vercel

Vercel Services deploy the Next.js frontend and Express API from this repository as one project. PostgreSQL remains hosted separately with Supabase or Neon.

```text
Browser -> Vercel frontend (/) -> Vercel Express API (/api/*) -> Supabase or Neon PostgreSQL
```

Vercel Services are in beta and available on all plans. The free Hobby plan has usage limits, and the database provider has its own limits; check both providers' current plans. This app has no authentication or authorization. Use disposable demo data only until write routes are protected.

## 1. Push the repository to GitHub

Push the whole repository, including `frontend`, `backend`, `database`, the root `package.json`, `package-lock.json`, and `vercel.json`. Do not commit `.env` files or credentials. If Vercel says it cannot find `vercel.json`, check that the file is at the repository root on the branch you are importing.

## 2. Create a PostgreSQL database

Create a Supabase or Neon PostgreSQL database and copy its connection URI. URL-encode special characters in the password. For Supabase, use a connection type that supports external serverless clients and follow its current TLS instructions.

The migration creates the `users` table. Existing local database records are not copied to the hosted database.

## 3. Import the Vercel project

1. In Vercel, import the GitHub repository.
2. Keep the project root at the repository root (`.`), where `vercel.json` is located. Do not set the project root to `frontend` or `backend`.
3. Vercel should detect two services: `backend` (Express) and `frontend` (Next.js).
4. The root `vercel.json` routes `/api/*` to `backend` and all other paths to `frontend`.
5. No service binding is needed: the browser calls the public same-domain `/api` route.

The Vercel configuration installs dependencies from the root npm workspace for each service. The backend build compiles `database` before `backend`, so TypeORM's compiled files are available when the API starts.

## 4. Set Vercel environment variables

Add these variables in the Vercel project settings. For `NEXT_PUBLIC_API_URL`, select Production and Preview. For the database variables, select each environment you intend to deploy; Preview deployments need database variables too if you want to test them.

| Key | Value |
| --- | --- |
| `NEXT_PUBLIC_API_URL` | `/api` |
| `DATABASE_URL` | Your hosted PostgreSQL connection URI |
| `DATABASE_SSL_MODE` | `verify-full` |
| `DATABASE_SSL_CA` | The provider CA certificate in PEM format, if the provider requires its own CA |

Do not set `DATABASE_URL` or a database password in any `NEXT_PUBLIC_` variable. `DATABASE_SSL_CA` is server-side configuration and is not a credential. For Neon, leave the custom CA unset unless its connection instructions require one. If Vercel has already started a deployment, redeploy after changing environment variables.

## 5. Run the initial database migration

Run the migration once before using the deployed API. Put the hosted `DATABASE_URL`, `DATABASE_SSL_MODE=verify-full`, and—if required—the provider CA file path in your ignored local `database/.env`. Then run from the repository root:

```powershell
npm run build -w database
npm run migration:run:prod -w database
```

Alternatively, if the Vercel backend environment has `DATABASE_SSL_CA` set, you can use the same PEM value for the local migration environment. Never commit database credentials. Future schema changes must be made through TypeORM migrations, not automatic synchronization.

## 6. Verify the deployment

1. Open the Vercel production URL. The frontend should load.
2. Open `<your-vercel-url>/api/users`. It should return an empty array before any users have been added.
3. Add a disposable user in the UI, refresh the page, and verify the row persists.
4. Check the `users` table in Supabase or Neon.

## Troubleshooting

| Problem | Check |
| --- | --- |
| Vercel says `vercel.json` is missing | Ensure it is pushed at the repository root on the selected branch and the Vercel project root is `.`. |
| Build cannot resolve `@users/database` | Keep the root npm workspace and lockfile; do not change either service root to the repository root or run an install from inside only one package. |
| API returns a database connection error | Check `DATABASE_URL`, the provider network/TLS settings, and `DATABASE_SSL_MODE`. Supply `DATABASE_SSL_CA` when the provider requires a custom root certificate. |
| API says the table does not exist | Run `npm run build -w database` and `npm run migration:run:prod -w database` against the same database URI used by Vercel. |
| Frontend calls the wrong API host | Set `NEXT_PUBLIC_API_URL=/api` and redeploy. |
| Requests return 404 | Check that `/api/(.*)` routes to the backend and that Express routes retain their `/api` prefix. |

For local installation, development, and database commands, see [CONFIGURATION.md](CONFIGURATION.md).
