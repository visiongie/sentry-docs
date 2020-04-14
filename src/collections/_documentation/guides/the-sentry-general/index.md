---
title: The Sentry General
sidebar_order: 7
---

Have you ever wondered,

> "How do large enterprise organizations uses Sentry?"  

>"I did the basics, but what do you recommend next?" 

> "What are some Best Practices?"

The Sentry General is a one-size fits all guide based on curated best practices and recommendations that Sentry recommends to its customers the most so they can maximize the value they get out of Sentry. It assumes some basic familiarity with Sentry. It identifies *what* you need to do and the regular documentation (docs.sentry.io) will have step-by-step instructions on how to do that. (what Sentry advises the most to customers)

## Architecture

picture...1 team: 10 code bases

picture...10teams: 1monolithic code base

### Projects
**General Rule** - Sentry Projects should be in a 1:1 ratio with your apps, with an "app" here meaning a repository.

#### Microservices
If you have multiple back-end microservices, then create a Project for each of those microservices.

#### Monolith

If your monolithic app is 1 code base but runs discreetly in different ways (e.g. one instance of it powers an API, and another instance powers web pages), then create a Project for each of those instances.

If your monolithic app has multiple code bases, then create a Project for each of those code bases.

You can leverage Alerts and Issue Owners to make sure developers are only getting notified about the parts of the monolith that they are responsible for maintaining.


#### DSN Keys
**General Rule** 1 DSN Key for 1 Project

The only time you should use multiple DSN keys for 1 Project
- you have tenants using your application software and want to keep track of them. However you could also do this using Tags (link)
- you want to control the errors you accept per environment (dev, staging, prod) and so you create a DSN key for each environment, and you can control a separate rate-limit per DSN key.

Bear in mind that Rate-Limiting is per DSN Key. (So you'd have to do it for every DSN Key for every project)

**Resources:**  
[how many projects should I create](https://docs.sentry.io/guides/getting-started/#-how-many-projects-should-i-create  )


### Releases
A Release in Sentry represents a version of your code that is deployed.

```
Sentry.init({
      release: process.env.release,
})
```

You can specify the release number yourself or you can use sentry-cli which will do it based on whether there have been new commits or not.

#### Why Releases?

Releases allow you to track the health of your applications across releases. You'll see New Issues vs Resolved issues as well as all *Files Changed* and *Commits* and *Authors* for that Release. This is possible if you've setup your [Integration]({% link _documentation/guides/the-sentry-general/index.md %}#workflow--integrations) with your version control software.

![Releases View]({% asset guides/the-sentry-general/releases-view.png @path %})

**Releases** enable you to use [Suspect Commits](). On a given error's page, Sentry will show you which commits updated files that are in the error's stacktrace.

#### CI/CD and Releases
It's important to create the Release using `sentry-cli` in your CI/CD before deploying your code, so if a Release has no errors, then you can see that 0 count in the Releases Dashboard. Otherwise you have to wait for an Error to occur.

If you trigger a build *and* new sentry Release on every commit in your CI/CD but only some of these get deployed to Production, that's okay because you'll be able to track the Releases in Sentry specifically from Production.

If you're unable to use an Integration with our supported version control systems, you can specify/send the commit metadata yourself using `sentry-cli`, as you issue new releases. Cross-repo commits would allow you to tell that it is coupled to N projects.

[Releases Documentation](https://docs.sentry.io/workflow/releases/?platform=python#configure-sdk)

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
If you have multiple dev teams working on 1 monolithic app, then configure your Alerts & Issue Owners based on the 'file path' that the error is coming from. This way, errors from ./src/* notify the 1 team who manages src/* while errors from something else like ./middleware/* go to the team that manages the middlewares. Order of precedence, so if you specify a fallback condition of ./*

#### Alerts & Issue Owners
...configure your Alerts and Issue Owners based on the file / sub-directory or URL in the repo that's causing the error. This way they're routed to the appropriate teams responsible for those subdirectories of modules/code


#### Suspect Commits
where Sentry matches files in the error's stacktrace with commits that updated those files. which are commits that updated files that are in the error's stacktrace.

## Understanding Your Volume
Photo? Guide on event quota management.
Viewing it as client vs server side.

#### Discover
3 Types of Workflow
1. wide angle, operations. management. ALL. tallies, statistics.
2. problem areas. that you're concerned about. volume.
3. problem, but not sure what area yet.

3 Levels of Analysis  
1. Projects (row items)
2. Issues
3. Events

## Hosted vs Self-Hosted
Thoughts, words. Guide on the migration.