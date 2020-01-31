# Deploy a website with Azure App Service

An app service does autoscaling , load balancing, and more automatically.

## Host a web app

First, we will create a web app in the portal, then deploy to it from vscode.

An app service web app has deployment slots, which means you can 'swap' the two slots, with one being the production version and the other the testing version. Once the testing version works, you swap it to production. This is nice for CI/CD.

### Automatic deployment

CI/CD works with the Deployment center, you can automatically deploy from Azure DevOps, Github, Bitbucket, local Git, FTP, OneDrive, Dropbox.

### Manual deployment

- Push to Git URL in the web app
- `az webapp up` packages your repository and deploys it
- Zipdeploy - uses curl to deploy a ZIP of the application file
- Visual studio (code) have extensions to deploy web apps
- FTP/S can be used to push code to an App service

## Visual studio

Link hier: <https://docs.microsoft.com/en-us/learn/modules/publish-azure-web-app-with-visual-studio/index> Heb ik overgeslagen want ik heb geen visual studio.

## Stage deployment and rollback with Deployment slots

Run web app in isolated environment, and deploy quickly without affecting users. In the Free/Basic Plan there are no deployment slots, the Standard plan can use 5, Premium can use 20.

Also, avoid cold start, azure will send a request to root, thereby warming up the slot before deploying.

Also, they share vms and deployment plans, only the hostname is different really.

### Autoswap

You can configure autoswap for a certain slot, when you update it, it will automatically swap it to another slot.

## Scaling out/up an App service web app

Scaling has advantages:

- Helps meet high demand
- Saves cost by downscaling when demand drops

However, at least in this section, this is all manual, and not about autoscaling. Still, it seems reasonable that you can look at performance, and when it drops, add an extra instance, or when it is underused, remove some instances.

App service plans have tiers which generally have an effect on scaling

- Free tier - supports 10 apps, but no SLA, single shared instance, and 60 minutes per app per day
- Shared tier - supports 100 apps, no SLA, single shared instance, 240 minutes per app per day. Not available for linux.
- Basic tier - unlimited apps, scale out to three instances, varying compute/storage/disk. 99.95% SLA
- Standard - Scale out to 10 instances
- Premium - Scale out to 20 instances
- Isolated - Runs in dedicated Azure Virtual Network. Can scale out to 100 instances.

You can also scale up an app service plan. During a scaleup, the hardware is increased. This can mean an interruption to service, like request in process being dropped, users need to refresh the page, etc.

## Deploy a web app with docker

### Azure Container Registry

It's a private docker registry. Better than Docker Hub because it's actually private and encrypted at rest. Also it's near where the image will probably be deployed.

`az acr create`

You can also use the acr to build images. You send the Dockerfile to the acr, then the acr builds it. Example: `az acr build .`. (you need to add params to which acr you want to use etcetera)

### Deploying from an image in the ACR

Create a web app, chooise 'Publish: Docker image'. Then, in the options, select which image you want.

### Continuous deployment

You can configure webhooks in the app service, so that the webapp 'subscribes' to the ACR and receives when there is an update.

You can also configure a webhook in the acr, so that it looks for changes in the git repository, and automatically pulls/rebuilds any changes. (or of course automate it in Devops or something)
