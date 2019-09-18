---
title: "Cleanup"
weight: 90
---

If you want to keep your application running, you can do so, but this is the end
of this section. In order to clean up and tear down all the resources created in
this section, navigate to [AWS CloudFormation Console](https://console.aws.amazon.com/cloudformation)
and delete the 2 CDK stacks that was created.

Select **CdkWorkshopStack** then click **Delete**.
![cleanup](/images/infrastructure/cleanup.png)

For the **CDKToolkit** stack, you will need to empty the S3 bucket that was
created by the stack before deleting it. Go to [Amazon S3 Console](https://s3.console.aws.amazon.com/s3)
and find navigate into the bucket name starting with **cdktoolkit-stagingbucket**,
select all the folders then select **Action** and then **Delete**. After that
you can go ahead and delete the stack.

