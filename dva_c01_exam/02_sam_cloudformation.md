# SAM

At its core, SAM is a way to use Cloudformation. I was really confused about what the actual point was, but <https://stackoverflow.com/a/68155484/1615209> explained it fairly well. It takes Cloudformation, and adds 'transforms', aka functions that expand to a valid Cloudformation template.

Example: the resource type `Type: AWS::Serverless::Function` will expand to a `Lambda`, `Role`, `Policy`, `APIGateway Stage`, `APIGateway Endpoint`. Of course, letting SAM do the transform for you saves a lot of code/effort vs having to write those 5 resources yourself.

Theoretically, you can do everything that SAM does with Terraform and CI/CD, but SAM can make it easier, and it's just a different way to do it.

NOTE: AWS Documentation is very confusing and insistent on referring to SAM as 'serverless' or 'lambda-based application'. As far as I can tell, this is only true insofar as SAM has more transforms to create lambdas and such. Most importantly, the `AWS::Serverless::Function` type (but there is also `API`, `HttpApi`, `SimpleTable`). However, I fail to see how the concept of transforms is somehow specific to serverless.

Generally, you have a `template.yaml` that describes your infra. This has the same format as a Cloudformation Template. (which means you can combine SAM with 'vanilla' Cloudformation)

Then you have extra files/folders with code/data. e.g. if you want a Javascript Lambda, you may provide an `index.js` file with the code.

Template specification: <https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification-template-anatomy.html>

### SAM/Cloudformation Template

- Transform: You can define a list of transformations. These transformations essentially take in a template and spit out a new template. The most 'famous' is `AWS::Serverless-2016-10-31` which is the SAM transform, but you can also make your own. Without this, SAM resources won't work.

- Globals: certain global settings you want to use for multiple resources. E.g. you can set `Function: Runtime: nodejs12.x`, and if you have multiple `Function` resources, they will all use that `Runtime` by default
  - *This is only valid for SAM! You cannot use Globals in a vanilla Cloudformation*

- Description: Just a way to add comments

- Metadata - Kind of a 'free' field that you can add arbitrary data about each resource. Can have some small cosmetic effects, e.g. it affects the order of the resources in the console. But can also just be used for comments.

- Parameters - Kinda like terraform `variables.tf`. You can e.g. define a parameter named `InstanceType`, with a default value of `t2.micro`. You can then refer to it with `Ref: InstanceType`. You can override default values at runtime like in Terraform.

- Mappings - Essentially another way to define a Global, but a map instead. Best example is e.g. if you have different values for each region. Each mapping must be double, a.k.a. you always need 2 keys to get to a value. If you have a mapping named `ValuePerRegion`, you can get the value with `!FindInMap [ValuePerRegion, !Ref "eu-west-1", "KeyName"]` (Also don't worry too much about the yaml syntax, hope they won't ask that on the exam :))

- Conditions - Kind of your `count = var.count ? 0 : 1` alternative. Each `Resource` can have a `Condition` field, and refer to a condition listed here. Example: `CreateProd: !Equals [!Ref Environment "prod"]` will make a condition that only evaluates to true if Environment is prod.

- Resources - The actual list of resources. Each resource has it's own list of parameters.
  - Type: The type of the resource. E.g. `AWS::EC2::Instance`
  - Condition: The name of the condition to check if we should make this (optional)
  - Properties: List of properties. Different for each type.

- Outputs: List of outputs, again similar to Terraform. Example: `Outputs: InstanceName: Value: !GetAtt EC2Instance Name`

- Rules: List of rules to be verified before running. Each rule has a RuleCondition and Assertions. e.g. you can assert that when environment is test, instanceType must be `t2.micro1`

- Format Version: Just a standard (but optional!) section to define the template. Looks like `AWSTemplateFormatVersion: "2010-09-09"`. Not neccecary for SAM.