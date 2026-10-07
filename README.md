# fem-cicd-service

A small Astro site that exists to carry a GitHub Actions pipeline. The site is a
single page. The pipeline around it is the point.

Built while working through the Frontend Masters CI/CD course.

## The pipeline

Four files, arranged so the build is defined once and used by both pull requests
and deployments.

| File | What it does |
| --- | --- |
| `.github/actions/build-astro/action.yml` | Composite action. Sets up Node with npm caching, runs `npm ci` and `npm run build`, uploads `dist/` as an artifact |
| `.github/workflows/_build.yml` | Reusable workflow that calls the composite action. Takes `node-version` and `artifact-name` as inputs |
| `.github/workflows/ci.yml` | Runs on pull requests to `main`. Calls `_build.yml`. Does not deploy |
| `.github/workflows/deploy.yml` | Runs on push to `main`, or manually. Builds, then syncs `dist/` to S3 |

## Decisions worth explaining

**Actions are pinned to commit SHAs, not tags.** A tag can be moved to point at
different code later. A SHA cannot.

**AWS access uses OIDC instead of stored keys.** The deploy job requests an
identity token and assumes an IAM role. No long-lived AWS credentials are kept
in the repository or in repository secrets.

**Concurrency is set differently for each workflow.** CI groups by pull request
number and cancels runs that are already in progress, because a check against
outdated code is wasted work. Production deploys use one shared group and do not
cancel, so a deploy in progress is never interrupted part way through.

**The build is written once.** Both `ci.yml` and `deploy.yml` call `_build.yml`,
so a pull request is built exactly the way a deploy is built. There is no second
copy of the build steps to drift out of sync.

## Running it locally

```bash
npm ci
npm run dev
```

## Status

The pipeline is finished. The site itself is a placeholder.
