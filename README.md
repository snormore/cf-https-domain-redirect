# Problem

You need to redirect a domain using SSL to another domain, for example https://example.com to https://app.foo.com. 


# Solution

Without the constraint of the source domain requiring HTTPS/SSL, this would be simple to do using [S3 bucket redirection](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/redirect-website-requests.html). Instead we are going to use CloudFront, S3, and Certificate Manager to accomplish this, with the setup encapsulated in a CloudFormation template. This solution is based on [these instructions from Aptible](https://www.aptible.com/support/topics/enclave/how-do-i-setup-an-apex-redirect-using-aws/) for redirecting from an apex domain using a similar technique.


## Instructions

### Phase 1 - Create your CloudFormation stack

Start by creating a CloudFormation stack using the template provided in this post.

 1. Navigate to the [CloudFormation Dashboard](https://console.aws.amazon.com/cloudformation/home) and click "Create Stack"
 2. Choose "Specify an Amazon S3 template URL", and use this URL: https://s3.amazonaws.com/snormore-cloudformation/redirect-https-domain.json
 3. Click "Next" and fill out the following parameters:
   - `Stack name` can be any identifier, I would choose something like `example-com-redirect`
   - `FromDomain` is the source domain, and would be `example.com` in our example
   - `ToDomain` is the destination domain, and would be `app.foo.com` in our example
   - `ViewerBucketName` is the S3 bucket name for redirection, this cannot contain dots. I would normally make this the same as your stack name, `example-com-redirect`
 4. Then click "Next" through the next setup screens, you'll see `CREATE_IN_PROGRESS` on your newly created stack.

### Phase 2 - Domain ownership confirmation for AWS Certificate Manager

From here, CloudFormation will use AWS Certificate Manager to provision and renew a SSL certificate to use for the source domain, for free. This process involves validating domain ownership, so you will receive an email (if you own the domain) from AWS asking for confirmation. Once you receive this, follow the instruction to confirm ownership.

### Phase 3 - Wait some time for CloudFormation to create your stack

Your CloudFormation stack will process for about an hour on various stages of `CREATE_IN_PROGRESS`, assuming nothing went wrong. If there is a failure, you can click into the stack and take a look at the event log for details.

If your stack creation failed, and you see a rollback notification and/or a failure status, check that there are no conflicting S3 buckets with the same name as your `ViewerBucketName`.


### Phase 4 - Configure your DNS provider

When your stack has been created, you'll see a status of `CREATE_COMPLETE`. Grab the `DistributionHostname` output from the Outputs tab, and update your DNS records to include either a CNAME to this value, or if you're using Route53 in AWS, you can specify it as an internal Alias.


# End

That's it. The CloudFormation template is [on Github](https://github.com/snormore/cf-https-domain-redirect) if you'd like to fork and extend.
