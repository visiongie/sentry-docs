---
title: The Sentry General
sidebar_order: 7
---

Have you ever wondered,

> "How do large enterprise organizations uses Sentry?"  

>"I did the basics, but What's next? Recommendations? Best Practices?

The Sentry General is a one-size guide fits all guide. It features the same building blocks that the smallest to largest customers use. It's is a set of curated best practices and recommendations (optimizations) so you can maximize your value out of Sentry. The regular documentation shows you all of the possibilities, the Sentry General has parsed through that for you to show what Sentry ends up advising its customers the most.

Assumes some basic familiarity with Sentry already.

(You'll see a division of Basic and Advanced. If you have the Basics down, then try Advanced!)

## Architecture

picture...1 team:1 code base

picture...1 team: 10 code bases

picture...10teams: 1monolithic code base (Issue Owners)

### Organizations
You only need 1. It has many teams.

### Teams
Many teams within an organization. A team gives users access to the Projects associated with it. [Get your User added](https://docs.sentry.io/accounts/membership/#restricting-access) to a Team in order to access the Team's Projects

SSO, Okta.

Teams - [Open Membership](https://docs.sentry.io/accounts/membership/#open-membership) allows members to manage their own teams, without an admin.

### Projects
These should be in a 1:1 ratio with your apps.

Question - How Many Projects Should I create?

#### Projects for Microservices
The recommended way to represent each of their apps in Sentry is with their own Project in Sentry. 1:1 ratio.

If you have multiple back-end microservices, then create a Project for each of those microservices.

#### Projects for the Monolith
If your monolithic app has multiple code bases, then create a Project for each of those.

If your monolithic app is 1 code base but runs discreetly in different ways (e.g. one instance of it powers the API, and another instance powers marketing pages), then create a Project for each instance.

If you're concerned about too many users being responsible for parts of the monolith they don't manage, you can leverage
Alerts + Issue Owners configured based on subdirectories in your source code to route errors to the appropriate teams responsible for those subdirectories of modules/code

#### Project DSN Keys
The only time you should use multiple DSN keys for 1 Project
- you have tenants using your application software and want to keep track of them
- you need to adjust rate-limiting across your environments.

Bear in mind that Rate-Limiting is per Project and per DSN Key. So if you're trying to RL, have to do for Projects * Keys.
**Resources:**

https://docs.sentry.io/guides/getting-started/#-how-many-projects-should-i-create  
https://docs.sentry.io/error-reporting/quickstart/?platform=python#configure-the-sdk  
https://docs.sentry.io/error-reporting/configuration/?platform=python#dsn  

### Releases
A release in Sentry represents a version of your code that is deployed

Releases let you track which errors came from which releaseReleases have commits associated to them, so you can tell which commits may [suspect commits](link) behind the errors

You can make the Release # based on an environment variable that is set during your build process. 
 
You can make the Release # by using sentry-cli, which will generate a new release # if there have been new commits since the previous release. Sentry-cli will verify commit info in the .git subdirectory (or via the integration?) as well as Release numbers via Sentry.io.

Can keep release # in one-to-one correlation to your app's version (think Release in Github). If you trigger a build (and new sentry Release) on every commit in your CI/CD but only some of these get deployed to Production, that's okay because you'll be able to track the Releases in Sentry specifically from Production. 

If you're unable to use an Integration with Github, Bitbucket, Gitlab, you can use sentry-cli to specify commits yourself, as you issue new releases. Cross-repo commits would allow you to tell that it is coupled to N projects.

```
Sentry.init({
      release: process.env.release,
      ...
})
```
### Environments
Do not forget to specify an environment where you initialize Sentry
```
Sentry.init({
      environment: "production",
})
```
So you can leverage them in top level filters and Discover.
(photo1)  
(photo2)  

## Define Your Data
Sentry already provides you with a lot of context regarding the state of the code environment, device, part of your app, that it happened with.

#### Tags
Sentry SDK's set roughly 10 depending on the platform.

You can add more context. Some quick ideas:
- feature flags
```
# keep track of a user experience, if it errors up
Sentry.configureScope(scope => {
      scope.setTag("studioEditorEnabled", true);
});
```
- customer segments
```
# after loading in the user object
Sentry.configureScope(scope => {
      scope.setTag("customerType", "elite-status"); // custom-tag
});
```

component: 'scanner'

component: commentbox

scanner: true 

commentbox: true

Mission-critical

Significant user events (signup, checkout)

Lifecycle Methods, Mounting, Connections

View them from Project Settings > Tags

Bear in mind you may be able to identify the errors from the above simply by the URL, which is already captured. as well as `stack.module`, `stack.filename`. (photo)

#### Tags Source Maps
Because the unminified Stacktrace is Your Data too

Stacktrace is code, and caused exception

#### Tags Front End
Component (React/Angular) Initialization

#### Tags Back End
Middleware, for sessionId, transactionId, user
modelId, entityId, decoders.js

#### Breadcrumbs
In front-end, Redux actions are a great [example](https://blog.sentry.io/2016/08/24/redux-middleware-error-logging)

If you're not familiar with what Breadcrumbs already capture, see [here](https://docs.sentry.io/enriching-error-data/breadcrumbs/?platform=python)

#### Additional the Extra Data
...

## Filtering
### SDK Side
Return null

Precaution - no idea if it's significant, less indicatino how many users. spikes small and large same

Blacklist URL's


based on event.exception test, event.info level

based on event.tag key

based on event.tag key, value

Also opportunity for Data Scrubbing (link to that section)



### Server Side
per project.

Inbound Filters

Rate Limiting

By Release

Grouping (is not filtering, but)

Inbound Data Filter - allowed domains (whitelist, blacklist)

## Sensitive Data
If you don't send it, Sentry can't persist it.

## Workflow & Integrations
Advanced - If you have multiple dev teams working on 1 monolithic app, then configure your Alerts & Issue Owners based on the 'file path' that the error is coming from. This way, errors from ./src/* notify the 1 team who manages src/* while errors from something else like ./middleware/* go to the team that manages the middlewares. Order of precedence, so if you specify a fallback condition of ./*

## Understanding Your Volume
Photo? Guide on event quota management.
Viewing it as client vs server side.

## Hosted vs Self-Hosted
Thoughts, words. Guide on the migration.