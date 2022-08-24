# Code services

There are a bunch of CodeX services:

CodeCommit - source control service. By itself, does just the git stuff, so no pipelines etc. You can manage access/clone/push rights/etc with IAM
CodeBuild - Continuous Integration, does things like compiling, testing, building, and producing packages.
CodeDeploy - Continuous Deployment: can deploy to EC2, Fargate, Lambda.

CodePipeline - Continuous Delivery - This is basically the pipeline service that combines all the above. You can use any of the above seperately, or you can create a pipeline that gets code from codecommit, builds it, deploys it.

CodeArtifact - Fully managed artifact repository. Basically like S3 but with some more metadata like domains/repositories that makes it easier to use. Mostly for use with CodeBuild.
  However, you can also do stuff like host Pip repositories, so it's not just S3.

## CodeDeploy Deployment policies

### Intro

Code Deploy provides two types of deployment:

- In-place: stop an instance, install newest version, restart the instance. Only EC2/on-premise supports this.
- Blue/green: start a new instance, and test it. All platforms support this, except on-prem

A revision is a YAML/Json file (AppSpecFile) containing the configurration. For EC2, you must also contain the source code.
CodeDeploy takes the revision, as well as a configuration (inplace, canary, rolling, etc), and a deployment group (where to deploy to).

## Deployment policies, cont.

There are different services that use deployment policies, mainly Beanstalk and API Gateway. The goal is that you have some EC2 instances that run some app, and you want to 'switch over' to EC2 instances with the new version, possibly without interrupting service, possiby while minimizing costs.

- Blue/Green Deployment - Deploy an ENTIRE new environment. Especially in Beanstalk. You now have 2 environments, now switch over the DNS CNAME record to point to the new environment. Then after a few minutes, shut down the old environment (when user sessions have ended)
- All-at-once - Shut down/restart environment. Causes minor downtime.
- Rolling - Deploy new version in batches, by restarting one batch at a time. Reduced capacity while each batch is out of service.
- Rolling with additional batch - Deploy new batch, then do rolling update. No reduced capacity, but slightly additional EC2 costs.
- Immutable - AWS Creates a second load balancer behind the first, deploys behind there, and after health checks, move instances to go to the main load balancer. Then terminate old instances in the end.
- Traffic Splitting - Deploy new instances, split new traffic 50/50 for a while to health check, then shut down old instances.

Canary is basically any slow rollout, so something like Immutable or Rolling could be considered Canary. The important thing is you can rollback if the health check fails.

### Codedeploy configurations

#### EC2

In-place shuts down, then redeploys. Blue/green deploys, then shuts down the old after rerouting.

- All at once
  - In-place or Blue/green
- Half at a time
  - In-place or Blue/green
- One at a time
  - In-place or Blue/green

#### ECS

Canary means deploy 10% right now, then the other 90% x minutes later.
All at once is done in a blue/green fashion, so no downtime.

- Linear 10 percent every minute
- Linear 10 percent every 3 minutes

- Canary 10 percent 5 minutes
- Canary 10 percent 15 minutes

- All at once

#### Lambda

Canary means deploy 10% right now, then the other 90% x minutes later
All at once is done in a blue/green fashion, so no downtime.

- Linear 10 percent every minute
- Linear 10 percent every 2 minutes
- Linear 10 percent every 3 minutes
- Linear 10 percent every 10 minutes

- Canary 10 percent 5 minutes
- Canary 10 percent 10 minutes
- Canary 10 percent 15 minutes
- Canary 10 percent 30 minutes

- All at once
