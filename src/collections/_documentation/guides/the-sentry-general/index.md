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

[how many projects should I create](https://docs.sentry.io/guides/getting-started/#-how-many-projects-should-i-create  )

#### DSN Keys
**General Rule** - 1 DSN Key for 1 Project

The only time you should use multiple DSN keys for a Project
- you have multiple tenants accessing dedicated instances of your software/product. You could also do this using Tags (link)
- you want to control the errors you accept per environment (dev, staging, prod) and so you create a DSN key for each environment, and you can control a separate rate-limit per DSN key. (beforeSend?)

Bear in mind that Rate-Limiting is per DSN Key. (So you'd have to do it for every DSN Key for every project)


### Releases
A Release in Sentry represents a version of your code that is deployed.

```
Sentry.init({
      release: process.env.release,
})
```

You can specify the release number yourself or you can use sentry-cli which will do it based on whether there have been new commits or not.

#### Why Releases?

Releases allow you to track the health of your applications across releases. You'll see *New Issues vs Resolved issues* for the Release. This is possible if you've setup your [Integration]({% link _documentation/guides/the-sentry-general/index.md %}#workflow--integrations) with your version control software. **Releases** enable you to use [Suspect Commits](). On a given error's page, Sentry will show you which commits updated files that are in the error's stacktrace. Also upload your [Source Maps](https://docs.sentry.io/platforms/javascript/sourcemaps/).

![Releases View]({% asset guides/the-sentry-general/releases-view.png @path %})

You'll also see the *Files Changed*, *Commits* and *Authors* per Release.  
![Releases Commits]({% asset guides/the-sentry-general/releases-commits.png @path %})


#### Releases in CI/CD
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
![Environment Top Level Filter]({% asset guides/the-sentry-general/environment-top-level-filter.png @path %})

Do not create a separate Project for each of your environments.

[Environments](https://docs.sentry.io/enriching-error-data/environments/?platform=python)

## Define Your Event Context

### Tags
The Sentry SDKs sets roughly 10 Tag values for you depending on the platform.

Anything mission-critical. Significant user events / business processes (signup, checkout). Lifecycle Methods, Mounting, Connections

You can add more context. Some quick ideas
- feature flags or user experience
- product / user experience
- customer segments and tenants
- component/area of your app

#### Tag Examples
```
Sentry.configureScope(scope => {
      # Feature Flag
      scope.setTag("studioEditorEnabled", true);
      
      # Product of yours that your customers are using
      scope.setTag("product", 'shipping');
      # or
      scope.setTag("shipping", true)
        
      # Know which customer / tenant of your software
      scope.setTag("customer", 12345)
      
      # Component / area of app
      scope.setTag("component", 'dashboard') # commentBox, commentBox: true
});
```

You can view what custom tags users have set in Project Settings > Tags. Create custom Discover queries based around them.

{% capture __alert_content -%}
You don't have to use a Tag for everything. Good for reporting. The 'URL' tag on the event might already indicate what part of your app. The stack.filename if you wish to report via that. Breadcrumbs may reveal the info too, but those are not indexable/searchable.
{%- endcapture -%}
{%- include components/alert.html
    title="Note"
    content=__alert_content
    level="warning"
%}

### Stack Trace
The stack trace is part of the event. **Javascript** make sure you upload your **Source Maps**. Below is an example but reference [Source Maps](https://docs.sentry.io/platforms/javascript/sourcemaps/) for complete documentation. Don't forget your [Integration]({% link _documentation/guides/the-sentry-general/index.md %}#workflow--integrations) so can associate to a commit / Suspect Commits.

```
# 1 Create Release
sentry-cli releases -o <YOUR_ORG> new -p <YOUR_PROJECT> <RELEASE_NUMBER>

#2 Associate Commits
sentry-cli releases -o <YOUR_ORG> -p <YOUR_PROJECT> set-commits --auto <RELEASE_NUMBER>

#3 Upload SourceMaps
sentry-cli releases -o <YOUR_ORG> -p <YOUR_PROJECT> files <RELEASE_NUMBER> \
      upload-sourcemaps --url-prefix "~/<PREFIX> --validate build/<PREFIX>
```

The stack trace appears if it was sent with the native exception, by the language/framework run-time. Some languages by nature of design do not provide this, but can see Packge/Class and Function name as well as line number, like in Java.

**Common Reasons for No Stacktrace**
- Unhandled promise rejections.
- Failed mounts/integrations.
- 3rd party javascript, no source maps for these

#### Front End Tags
Component (React/Angular) Initialization

#### Back End Tags
Middleware, for sessionId, transactionId, user
modelId, entityId, decoders.js

### Breadcrumbs
In front-end, Redux actions are a great [example](https://blog.sentry.io/2016/08/24/redux-middleware-error-logging)

If you're not familiar with what Breadcrumbs already capture, see [here](https://docs.sentry.io/enriching-error-data/breadcrumbs/?platform=python)

### Extra Context
Good for object/json data, key value pairs. What a user sees in the GUI could be different than what's in javascript memory or what's in the database.

#### Extra Context Examples
- Parts of Redux but do not recommend putting the entire redux tree, for sensitive information and size limitation concerns.
- 'Shopping Cart' something user might Add/Remove CRUD a lot, this is prone to error.



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

Advanced Data Scrubbing

## Workflow & Integrations
If you have multiple dev teams working on 1 monolithic app, then configure your Alerts & Issue Owners based on the 'file path' that the error is coming from. This way, errors from ./src/* notify the 1 team who manages src/* while errors from something else like ./middleware/* go to the team that manages the middlewares. Order of precedence, so if you specify a fallback condition of ./*

### Alerts & Issue Owners
...configure your Alerts and Issue Owners based on the file / sub-directory or URL in the repo that's causing the error. This way they're routed to the appropriate teams responsible for those subdirectories of modules/code


### Suspect Commits
where Sentry matches files in the error's stacktrace with commits that updated those files. which are commits that updated files that are in the error's stacktrace.

## Understanding Your Volume
Photo? Guide on event quota management.
Viewing it as client vs server side.

### Discover
3 Types of Workflow
1. wide angle, operations. management. ALL. tallies, statistics.
2. problem areas. that you're concerned about. volume.
3. problem, but not sure what area yet.

3 Levels of Analysis  
1. Projects (row items)
2. Issues
3. Events

### Other
Stats / Subscription / Issues page do's don'ts limitations

## Hosted vs Self-Hosted
Thoughts, words. Guide on the migration.