# Code services

There are a bunch of CodeX services:

CodeCommit - source control service. By itself, does just the git stuff, so no pipelines etc. You can manage access/clone/push rights/etc with IAM
CodeBuild - Continuous Integration, does things like compiling, testing, building, and producing packages.
CodeDeploy - Continuous Deployment: can deploy to EC2, Fargate, Lambda.

CodePipeline - Continuous Delivery - This is basically the pipeline service that combines all the above. You can use any of the above seperately, or you can create a pipeline that gets code from codecommit, builds it, deploys it.

