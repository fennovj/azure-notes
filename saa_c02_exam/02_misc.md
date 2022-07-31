# Misc Topics, Useful Links

- https://docs.aws.amazon.com/datasync/latest/userguide/datasync-in-vpc.html

## Storage gateway more info

Some more info about Storage Gateway: You have 'File Gateway' that stores files directly on S3, or 'Volume Gateway' that uses EBS volumes. For the latter, there is:
  - Gateway Cached Volume - Stores the full volume in S3. Keep recent data in a local cache on the gateway, and read/write to S3 when needed.
  - Gateway Stored Volume - Store the volume on the gateway, while maintaining an asynchronous copy in S3. (I'm not sure how this works exactly, is it just using S3 for backup purposes?)

##  CloudFront OAI vs Signed URL's vs Signed Cookies

<https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-choosing-signed-urls-cookies.html>
<https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html>

These three have the same purpose: They are used to control who can access content

- Signed URL: inserts query string parameters `Expires=&Policy=&Signature=&Key-Pair-Id=`. It works by changing your webapp so that when a user requests a file, you instead verify the user, and return the URL. The browser will automatically request the new URL. Cloudfront will verify that the Signature is valid. The Policy can describe which resources you have access to.
- Signed Cookie: You give the cookie to the user when they log in. The cookies are signed and automatically sent with the request, and again, CloudFront verifies the signature.

The Signed URL only works for one file at a time, so is potentially more overhead, e.g. when a user downloads some large multipart file. Also, Signed URL's override the 4 URL parameters mentioned above. Note that not all http clients support cookies. Other than that, it seems mostly a matter of preference.

- OAI: Origin Access Identity. In this case, you change the bucket policy so that you cannot access the S3 files directly without the relevant identity. An identity with access is then assigned to CloudFront. The main goal is so that users can *only* access certain files through CloudFront - e.g. because you want to take advantage of the caching that CloudFront offers, or you have logging in CloudFront that you don't want to be circumvented.

OAI works well in combination with Signed URL's/cookies: if you want to restrict file access, it doesn't make sense if it's publicly available through cloudfront anyway. When you use both, a file is truly restricted. OAI is also useful if you want certain files to be public and others to be CloudFront-only: you can limit the bucket policy to only certain files.

## Migrating AWS Accounts to a new organization

In AWS, there is a difference between member accounts and management accounts. The management account is basically only for cloudtrail, and can be used to police other accounts. You can 'unlink' a member account and make it standalone. <https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_remove.html#orgs_manage_accounts_remove-from-master>. You must have a (temporary) payment method attached to the account to do this. The migration process is as follows:

- Remove all member accounts from the old organization
- Invite all member accounts from the new organization
- Delete the old organization - this will make the management account into a standalone account
- Invite the old management account as a member account - this may not be neccecary if the new organization's management account is sufficient

## ECS Launch Types

There are 2 launch types for running containers in ECS:

- Fargate - Serverless, automatically provisions and scales hardware as you add/remove containers.
- EC2 - you manage the compute yourself, and run the containers on the EC2 instances. This can have potential cost savings mt requires more management, you have to do autoscaling yourself, etcereta.

## Notes about Glacier

Objects in Glacier can't change tier. You need to restore them first. In fact, you need to do that even if all you want to do is read the file. The process is:
- Request a restoration for a specified duration.
- Wait a few minutes (or a few hours) for the file to be restored.
- If you want a permanent copy of the file, you need to copy the file to another location in S3.
- (Optionally) add a lifetime policy to the new copy to move it to the tier you want it.

There are multiple storage tiers, even within Glacier:

- Glacier Instant Retrieval - Can be retrieved like normal S3 files, but is just expensive to read. GET requests are ~20x more expensive, storage is ~5x cheaper. Also it's (a little) slower than Standard S3.
- Glacier Flexible Retrieval - It takes a few hours to retrieve data by copying it to S3. There is the option for 'expedited' retrieval which takes minutes but is VERY expensive (200x more expensive than the standard retrieval).
- Glacier Deep Archive - Half the storage costs compared to Flexible Retrieval. There is no expedited retrieval option, retrieval takes ~12-48 hours.

Also, there is Intelligent-Tiering, which is based on usage: it places data in IA after 30 days, and Archive Instant after 90 days. There is also the *option* to move to deep archive after 180 days.

## Route 53 Alias vs CNAME (non-alias)

Generally, if you are pointing a domain to an AWS resource (such as CloudFront, ELB or S3), you want to use Alias.

The main difference between CNAME and A is the way it resolves: with CNAME, the DNS server returns the CNAME, then the client makes a second DNS lookup. With A, the DNS record will do the DNS lookup itself, until it reaches an ip adress. CNAME is therefore a little slower, and the DNS will eventually have to be resolved by amazon anyway.

Another thing of importance: Route 53 charges for CNAME queries, but not for Alias queries. Basically, it's probably only worth to use CNAME when you really need CNAME, e.g. when you are redirecting to a different DNS server that is not Route 53.

## Cost savings plans

- Compute Savings Plans - Reduce costs up to 66%. You just commit for compute, you are free to then use any family, size, AZ, region, etcetera.
- EC2 Instance Savings Plans - Reduce costs up to 72%. The best deal. You commit to an instance family, in a region. (e.g. M5 in eu-west-1)
- Reserved Instances - Savings plan where you commit to utilization e.g.: I will run 4 instances of the M5 family at least 75% of the time during the next year. More efficient for consistent applications like databases.
- OnDemand Instances - A normal instance, not Spot.
- Spot Instancs - Significantly cheaper instances that can be terminated at will by AWS. Can be used for things like autoscaling groups or databricks nodes that are fault-tolerant.

## Load balancer types

<https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/introduction.html>

Main difference: Classic can only inspect Ip adress and port, whereas Application can also inspect application-content such as endpoint.