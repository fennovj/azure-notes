# Misc

## KMS

First of all, most keys you work with are Symmetric (S3, RDS, etcetera). There's a reason for this: Symmetric keys are just more useful if you can make sure the keys don't have to leave AWS (which is usually the case). However, you can also make other types of keys:

- Symmetric - The default.
- Asymmetric - Use if you want a public key that can leave AWS. You can use RSA or elliptic curves.
- HMAC - Symmetric, for generating HMAC codes. HMAC codes are used to 'sign'/authenticate a message - If you receive a message + correct HMAC code, you can be sure the message was created by somebody with access to the key.
- Data key - This is an ad-hoc symmetric key you can use in your own application. AWS returns a plaintext and encrypted key. The encrypted key can be decrypted using an existing symmetric key. The plain key is used to encrypt/decrypt data. The key is not stored in AWS, the idea is you store the encrypted key alongside the data.

Secondly, there is a difference between customer-managed keys and AWS managed keys. The main difference is the access policy - AWS managed keys are by definition accesible by the entire account (of course provided you have the kms:decrypt or such role), while Customer keys can have a key policy. Also, customer managed keys allow you to control the renewal yourself, AWS managed keys are rotated every 3 years.

## Redis vs Memcached

<https://aws.amazon.com/elasticache/redis-vs-memcached/>

Generally, Memcache is simpler to use, Redis has more advanced features.

Memcached is multithreaded

Redis has snapshots, replication, more scripting like lua and geospatial functions. Also Redis is better if you want high availability, e.g. if your cache is really important to not overload the application.

So in summary: use Memcached either if the question specifically asks for multithreaded, or for something as simple to set up as possible. Otherwise, especially if the question asks for availability and performance, use Redis.

## Kinesis - picking the number of shards/consumers

Generally, don't pick more consumers than shards, since you can only really use one consumer per shard. Also, to optimize performance the go-to solution is to spread data across consumers evenly by splitting hot shards (shards with much data) and merging cold shards.

## EC2

If you want to detach an EBS volume, you can simply unmount it from your instance. If you detach without unmounting... stuff could go wrong, so don't do it.
However, if it's a root volume... you can't unmount it either. you need to stop the entire instance, then detach.

## CORS

S3 cors configuration is an xml file:

- AllowedOrigin - domains that are allowed to make cross-domain requests
- AllowedMethod - GET, PUT, etc: allowed types. Warning! PUT/DELETE are of course dangerous permissions to give random users.
- AllowedHeader - The headers allowed. Whitelist, even a single unallowed header will mean a deny. This can be * to allow everything.
- MaxAgeSeconds - The way CORS works in the browser, the browser sends an OPTIONS request to ask if CORS is allowed. This OPTIONS response is normally only cachable for 5 seconds. This lengthens that, so you don't have to re-ask every 5 seconds.
- ExposeHeader - Response Headers that the website will be able to access. By default, none (meaning the website can only read the request body)

Note that for CORS, the resource that is being called is where you manage CORS. E.g. if you have a website that calls an API gateway, you need to configure CORS in the api gateway. If you have a website that gets static files from S3, you need to configure CORS in s3.

## Login page

The signin page for AWS is `https://xxx.signin.aws.amazon.com/console/` where xxx is either your account id or an account alias. Seems really specific, but it was an example exam question I saw online.

## Placement strategies (ECS)

<https://aws.amazon.com/blogs/compute/amazon-ecs-task-placement/>

This is the strategy used to decide which instance a new task is started on. There are three layers:

- Cluster contraints: implicit requirements inherent to the task, like 'a port must be available' or 'needs 512MB memory'. Any instance that doesn't qualify is disqualified.
- Placement constraints: You can write `Cluster Query Language` to add additional, explicit, constraints. Example: `attribute:ecs.instance-type == t2.micro`, but you can get more complex with them
- Placement strategy: how to divide between the available instances. This is more subjective, and requires you to pass one of three strategies. (Default is random)

You can also pass a 'field': instanceId (the most common), or any other attribute like `availability zone`. This means you can also spread/pack across other fields. E.g. when the field is `memory`, binpack will try to minimize memory used.
You can also pass multiple fields. in that case there's a hierarchy: e.g. spread across availability zones, then pack within each zone.

- spread: spreads them as evenly as possible, places on the least used
- random: places randomly (but still honors the cluster/placement constraints)
- binpack: packs tasks together, as to minimize the number of instances

## EC2

The instance metadata, such as local/global IP is available on `http://169.254.169.254/latest/meta-data/`, This is useful when running scripts from the instance. Interestingly enough, this same IP address is used in Azure for the same purpose.
You can only access this from within the VM (and indeed, every VM will give you a different result)

E.g. if you want the instance type, you can do `curl http://169.254.169.254/latest/meta-data/instance-type`

## SWF vs Step functions

Both are sort of low-code, logic-apps like orchestration tools. Step functions is easier to use and more productive, and in doubt, use this.

SWF is a more complex but you have more control. It also allows you to write code using the Flow framework.

## Step functions

The output of a state can be the input, the output, or both. You can configure which, in the 'ResultPath'

In the 'OutputPath', you can filter/select the outputs, e.g. only output a single value instead of everything.

In the 'ItemsPath', you can define maps: e.g. you can say 'this input array, instead of treating it as an array, do a for-each loop and run this step multiple times'

Task states:

Task - run a task (e.g. call a lambda, run AWS CLI command, etc, etc.)
Parallel - split off into branches
Choice - choose a branch
Pass - simple pass input to output (with mapping)
Map - for-each loop (itemspath)
Choice - choose between branches
Fail or Succeed - potentially stop execution of the whole thing
Wait - wait for/until a specified time