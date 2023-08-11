# Azure DevOps

This is not going over DevOps as a whole, but more of a collection of stuff I learned that is Azure DevOps related.

## Analyics widgets

<https://learn.microsoft.com/en-us/azure/devops/report/dashboards/analytics-widgets>

There are a bunch of analyics DevOps offers you, that you can make widgets out of.
You must have 'View Analyics' on the project-level set to *Allow*. Users must be *Stakeholder* to see the widgets.

- Burndown: Trend of remaining work across multiple sprints. e.g.: we have 100 items left, we do on average 30 per sprint, we will complete in ~3 sprints.
- Burnup: Similar to a burndown, except instead of going down to 0, it goes up to a certain target.
- Sprint Burndown: trend of remaining work within a sprint. e.g.: we have 10 items left, we should do 1 a day. we are ahead of/behind schedule.
- Cumulative Flow Diagram (CFD): Count of work items over time, grouped by each column in kanban. E.g. around monday-wednesday, a lot of work items were stuck in review/security.
- Cycle Time: Time it takes for team to complete work *once they start actively working on them*
- Lead Time: Time it takes from *creation of work item to completion*
- Velocity: For each sprint: show planned and completed story points, and calculate average velocity.
- Test Result trend: track test quality of pipelines. Not really for unit tests, but rather e.g. checking pipeline health (which can be unit tests).

## Access levels

There are two places where access levels are being done: on a project basis, and on *Microsoft* basis. The first has to do with security, the second is more to do with pricing.

For the second one, your 'access level' is set on your Azure/Microsoft account, and is the same regardless of what project you are a part. The levels are:

- Basic: Provides access to most features. Meant for developers. Cost $$$, so you generally are not going to give this to every employee in your organization.
- Basic + Test Plans: Gives Basic, as well as Azure Test Plans.
- Stakeholder: Gives limited access to private projects, full access to public projects. Is completely free. For private projects, you have partial access to boards/pipelines/dashboards, no access to repos. This is also all you need if you are a *test participant* in the *Test & Feedback extension*.
- Visual Studio Subscriber: Meant for users who already have a Visual Studio subscription. You will get Basic, or sometimes Basic+Test Plan automatically, depending on what subscription that user has.

For Test plans: Stakeholders can only give feedback, Basic can execute tests/mark test outcomes, Basic + Test Plan can manage/create test plans/assign testers.

For the permissions within devops, there are 'project collection' roles and 'project' roles. Project collection is across all projects in an organization

With a project, there is a special *project administrator* role that allows you to do everything within a project. There are other groups, e.g. readers, that have limited access. You can also make your own groups. For repository security, you can manage it per repository, e.g. make it so Readers are set to *Deny* view rights on a specific repository, and even some rights (like contribute) can be done on a branch basis.

## Processes

<https://learn.microsoft.com/en-us/azure/devops/boards/work-items/guidance/choose-process>

There are 4 processes you can pick for your work.

- Basic: Simply has an Epic->Issue->Task hierarchy. Used when you simply want to track remaining work

- Agile: Use when you use agile, and want to track user stories and bugs.
- Scrum: Use when you use Scrum, and want to track product backlog items (PBI's)

It seems the main difference between agile and scrum is terminology. Also, with Agile, you can track original estimate/completed work, with Scrum you can only esitmate remaining work.

- CMMI (Capability Maturity Model Integration): More formal process. Use when you want to manage change requests/issues/reviews/risks. Also, with this you can also track original estimate/completed work.

## Service Hooks

<https://learn.microsoft.com/en-us/azure/devops/service-hooks/>

A service hook, in a nutshell, is running a task when an event happens in your project. There are many 'tasks' you can run. Some examples (not an exhaustive list!)

- Post a message in Slack/Teams
- Deploy App to App Service
- Post to a HTTP webhook
- Trigger Bamboo/Jenkins build

You can also add other apps/services via the Visual Studio Marketplace.
In fact, we use a service hook for our slack integration. (except we kind of set it up differently: instead of creating a service hook in devops directly, we installed a slack app, which then created a service hook for us)

Some things you can subscribe to are:
- Build completed/failed
- Code pushed
- PR created/updated/merged
- Release created
- Release deployment started/completed
- Work item created/updated/deleted

By default, only project administrators have permissions to create a service hook, but other users can be granted the 'edit service hook subscriptions' privilege.

## Branching strategy

<https://learn.microsoft.com/en-us/azure/devops/repos/git/git-branching-guidance>

Generally, microsoft recommends using a simple branching strategy with feature/bugfix/hotfix branches, and to keep the main/master branch up to date. This is called 'short lived branching strategy'.

You can also use release branches to manage releases. Potential bugfixes in the release branches should be *cherry-picked* back to the main branch.
(this still sorta fits with short lived, since the feature branches are short lived, even if the release branches are long lived)

Other strategies are:

- Master-based: everyone works on master, and everyone pushes directly to master (and locally merges if needed). This is fine only for the simplest of projects, but should not really be used for anything with any complexity. Also you cannot do PRs which you should be doing.

- Integration based: Here, there is some 'integration test' branch or branches, which are used to merge multiple feature branches. You have to pass some (rigorous) tests before being merged to master. This makes the workflow more complicated, so should probably only be used if you want to be really sure that the master branch is always functional.