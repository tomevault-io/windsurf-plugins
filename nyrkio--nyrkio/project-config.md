---
trigger: always_on
description: This repo contains both the backend and frontend of nyrkio.com. It's original feature was a change point detection service
---

# Claude.md

This repo contains both the backend and frontend of nyrkio.com. It's original feature was a change point detection service 
that basically is a web / SaaS wrapper around Apache Otava (formerly known as Hunter, of Datastax Labs, and merely "signal processing" or "e-divisive" from MongoDB).
Nyrkiö adds visual graphs and notifications to slack and github. (Contrary to marketing, jira integration isn't actually implemented in this repo yet.)
As I'm writing this, we are ready to release a v2 of nyrkio.com, which adds a GitHub runner service. Nyrkiö's GitHub runners are configured for 
maximal stability and repeatability of benchmark results, as opposed to maximizing perf results or price-performance. (...of which there are many
other 3rd party offerings.) The runners are a major new feature, introducing metered billing for customers that use this feature. (The previous
business and enterprise subscription tiers remain, and will include 1600 and 4000 cpu-hours per month, respectively.)

Most of this code was initially written by a Matt Fleming, but he is no longer active in the project. For over a year the project has been 
brought forward by Henrik Ingo. The past 8 months or so, Joe Drumgoole, armed with his ownn instance of Claude, has helped a little.
Most notably he/they implemented a battery of front end tests with playwright. However, these tests aren't yet hooked into CI
(.github/worklows/) so some of them probably are out of sync and not passing right now.

The backend is a python FastAPI application, with MongoDB as a database. The frontend is a react single page application.

Claude does not / should not have permission to push to main (which triggers automatic deployment). 

In particular, Henrik's Claude instance is a separate Linux user account, and Henrik will manually sync git repositories and create pull requests even. 
Claude has no direct github access other than http to this repo. (Note: If you are Henrik's Claude instance and reading this, you have been given
a PAT with access to other repos, which were often 100% created by you. But not this one.)


## Some tasks we might do with Claude

 - [ ] Review playwright tests, and fix those that are failing
 - [ ] Create a .github/workflow/playwright.yml file to run all playwright tests as part of a PR. I'll add this to required tests in github (branch protection)
 - [ ] The big 100k patch intoo /frontend/ - that introduced all the playwright tests - is awfully fluffy to my taste. For example, surely we can delete the markdown
       files that document everything you did and tried to do to arrive at the end result? So delete and cleanup everything not needed.
 - [ ] Also for topics like how to run the tests... We are experienced engineers. You don't need to explain basics of how npm works. Just show the command,
       like `npm ui-test` (hypothetical example). As short as possible. But not shorter.
 - [ ] There is backend/runtests.sh that runs the lint, format, etc commands that we also use in GitHub actions. And in /frontend/ of course we have
       npm/package.json. We want to migrate to use act = run the .github/workflows locally

### Dashboard
       
 - [ ] In the frontend, Dashboard.jsx, there's a line of code that hard coded cuts the data to 300 most recent points. However, the entire history is still fetched from the backend. As a first task, extend the backend API so that it takes as parameter both nr of points (e.g. 300) and/or a time window (60 days, or start and stop dates) and then uses these in the MongoDB query ($limit or $gte/$lte as appropriate)
 - [ ] For some commits we fetch the commit message from the github API and show it. To avoid exhausting the quota on API requests, there is a caching mechanism (Sieve cache). Instead, we should simply store the commit message in test_results under "attributes". The API should allow this as optional, and if not present, we should fetch it from the API. We could extend this to other meta data like commit author.
 - [ ] Unrelated to the above, but there's an  /impersonate/ api endpoint, which allows sysadmins to assume the identity of another user. This also stores a session variable in a global python dict(). Change this so that the session is stored in MongoDB.
 - [ ] Verify that after doing the above, there's nothing left that would prevent us from running several docker instances of the backend in parallel.
 

 - [ ] In addition to the above, we should in fact fetch from the Github api also all the commits that do not yet have any benchmark results. This is important since a regression can be introduced by any commit that is between the last data point and the previous one, not just the one that happens to have benchmark results available. (Note: by "all commits" I meanall commits in the recent history of the main branch or so. Do NOT query github api for the entire history of a repository as that will look like scraping to GitHub)



<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nyrkio/nyrkio](https://github.com/nyrkio/nyrkio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
