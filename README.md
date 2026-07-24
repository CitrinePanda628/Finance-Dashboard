# Personal Finance Dashboard

A web app I built for tracking money. You sign in, add your accounts (like a current account or a card), add categories (rent, groceries, salary), and log transactions against them. It shows all of it back to you in a dashboard with a chart on top and a table of recent transactions below, and you can filter by account and date range to see the numbers change.

If you already have your data in a spreadsheet you can import it as a CSV instead of typing everything in by hand. On the transactions page you can search, edit rows, or select a bunch and delete them in one go. The forms have proper checks so you can't accidentally save something broken.

## How it's built

The app runs on Next.js 15 with the App Router and Turbopack, React 19, and Tailwind for the styles. Most of the UI is shadcn/ui, which uses Radix under the hood, so all the dialogs, dropdowns and selects behave properly with the keyboard and with screen readers.

The database is Postgres running on Neon (their free tier is enough for this), and I use Drizzle as the ORM plus drizzle kit for the migrations. Zod handles the schema checks, and drizzle zod lets me write the shape once so I get the same types on the database, the API and the forms. That saved me a lot of headaches.

The API is written with Hono and lives inside the Next.js app under `app/api`. On the client, Tanstack React Query looks after fetching and caching the data so I never had to write loading and error states by hand, and Tanstack Table drives the transactions table.

Auth is Clerk. It comes with the middleware bit and its own sign in and sign up pages, so all I had to do was drop it in and point at those routes.

The rest is smaller stuff, react hook form with Zod for the forms, react papaparse for the CSV import, recharts for the charts, react countup for the animated numbers on the dashboard, sonner for toast messages, and next themes for light and dark mode.

## How the code is laid out

The `app` folder has the pages and the API. The `features` folder groups things by feature so accounts, categories and transactions each have their own hooks, forms and dialogs sitting together, which made it a lot easier to change one part without breaking another. `components` has the shared UI, `lib` has small helpers and the database client, `migrations` holds the SQL that Drizzle generates, and `providers` wraps everything in the Query client.

## What you need to run it

You need a Clerk account for auth and a Neon Postgres database for storage. Both have free tiers so this costs nothing to run locally.

Make a `.env.local` file at the root of the project and put these in it:

```dotenv
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up

DATABASE_URL='postgresql://<user>:<password>@<host>/<db>?sslmode=require'
NEXT_PUBLIC_API_URL=http://localhost:3000
```

The Clerk keys come from your Clerk dashboard once you make an app there, and the Postgres URL comes from Neon after you make a project.

Then install and run:

```bash
npm install
# or if you prefer:
bun install

# Push the schema to the database
npx drizzle-kit push

# Start the app
npm run dev
```

It opens at `http://localhost:3000`.


