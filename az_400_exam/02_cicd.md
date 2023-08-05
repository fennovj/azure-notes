# CI/CD / DevOps / Workflows

## DevOps

<https://learn.microsoft.com/en-us/devops/what-is-devops>

### DevSecOps

DevSecOps in a nutshell is integrating security into your workflow. The opposite of DevSecOps is doing a whole project, and either not caring about security, or only having security people take a look at it when the project is done, and not anymore after that. DevSecOps means already running security scans in every step of the process (pre-commit, ci, cd, production, ops), and even doing regular 'war-game' exercises where you actively try to compromise a product, even through social engineering. Although, I have to admit this seems like it goes a bit too far for the teams I am in contact with, who do mainly backend data pipelines and api's. But I'm sure a Microsoft security person would disagree.

## CI vs CD

It seems embarassing to have to write this down, but I realize that many resources simply say 'CI/CD' without splitting them up, or describing what they are. CI/CD is described as 'a development approach where developers work together on the same shared repository, and changes are automatically built and deployed'. Another way to look at it is: DevOps consists of multiple 'stages' of work, of which CI and CD are two of them.

- Pre-commit: before making a commit. Azure Devops can be used e.g. in specifying work items, e.g. demanding work items have certain information in them for the developer. Also pre-commit is IDE plugins, static code analysis (e.g. type checking in the IDE), and pre-commit hooks.
- CI: Happens after a commit. The code is built, and certain checks can be run. Generally, this also refers to pipelines that run on a Pull Request. E.g. unit tests, or code quality checks such as static code analysis. However, the code is not deployed yet, not even to a test environment.
- CD: Code is deployed to a (test or acceptance) environment. This also means testing of the cloud configuration, automated security scans (e.g. scanning which ports are open), and also automated load/security tests. Acceptance is also a good place for pen testers to 'play around' and find issues before they get deployed to production.
- Production: in production, you can check configurations: did some configuration differences between acc/prod break stuff? So perhaps this also involves automated security checks
- Operations: All the stuff after development, what makes it 'DevOps'. You do monitoring for threats, and can also do 'real' penetration testing where you get your security team to devote time to testing your app.

![Image from whizlabs](https://s3.amazonaws.com/media.whizlabs.com/learn/2020/06/26/ckeditor_6.png)
