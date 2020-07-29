# Metaflow Service Migration Guide

## Metaflow 2.1.+

In Metaflow 2.1.0, we introduced a new AWS service integration with AWS Step Functions. Now, users of Metaflow can easily deploy their flows to AWS Step Functions. If this is a functionality that you would like to use, depending on if/when you deployed the [metaflow service](https://app.gitbook.com/@hawkins/s/metaflow-admin/~/drafts/-MDQ9c_b9eEtHKMgoQni/metaflow-on-aws/metaflow-on-aws#metadata), you might have to take some actions to upgrade your service. If while trying to [schedule your flows](www.google.com) on AWS Step Functions via :

```text
python myflow.py step-functions create
```

you ran into the following error :

![](../../.gitbook/assets/screenshot-2020-07-29-at-8.27.22-am.png)

then you would need to upgrade the deployed version of your metaflow service. This upgrade requires migration of the backing RDS instance.

In this situation, the administrator should decide if and when they want to run the migration, which will incur some downtime - up to a few minutes. As a best practice, it is advisable to take a backup of the database prior to the migration which allows you to roll back the migration in case something goes wrong.

{% hint style="info" %}
We highly recommend [taking a backup of your RDS instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_CommonTasks.BackupRestore.html) before attempting this upgrade. This will allow you to restore the service from the backup in case there are any issues with the migration.
{% endhint %}

{% hint style="info" %}
This migration will result in a short downtime of the metaflow service. Metaflow users can guard against this downtime by using the [@retry](https://docs.metaflow.org/metaflow/failures) decorator in their flows.
{% endhint %}

To make this database migration easy, metaflow service comes with a built-in migration service. When you deploy or restart the [latest version of the metaflow service image](https://hub.docker.com/repository/docker/netflixoss/metaflow_metadata_service), the migration service will detect the schema version of the backing database, and launch the latest version of the metaflow service that is compatible with the database schema. The migration service provides hooks to upgrade the database schema to the latest version so that you can upgrade the metaflow service to the latest version.

There are two paths to upgrading your service, depending on how you first deployed the service - using our [AWS CloudFormation template](metaflow-service-migration-guide.md#aws-cloudformation-deployment) or [manually through the AWS console](metaflow-service-migration-guide.md#manual-deployment).

In addition to migrating the service, if you intend to use AWS Step Functions, you would need to update a few IAM roles and set up an Amazon DynamoDB table. The instructions below will walk you through those as well.

### AWS CloudFormation Deployment

If you originally deployed the AWS resources needed for Metaflow using our [AWS CloudFormation template](../deployment-guide/aws-cloudformation-deployment.md), then you can use AWS CloudFormation to spin up the necessary resources for this service migration. The latest version of the AWS CloudFormation template pulls the [latest version of the metaflow service image](https://hub.docker.com/repository/docker/netflixoss/metaflow_metadata_service) which comes bundled with a migration service as well as an AWS Lambda function which you can execute manually to trigger the database migration.

{% hint style="info" %}
We highly recommend [taking a backup of your RDS instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_CommonTasks.BackupRestore.html) before attempting this upgrade. This will allow you to restore the service from the backup in case there are any issues with the migration
{% endhint %}

1. Open the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation) and choose the stack corresponding to your existing deployment.
2. Choose _Update_ and choose _Replace current template_ under _Prerequisite - Prepare template._ 
3. Choose _Upload a template file_ under _Specify template_.
4. Choose _Choose file_ and upload [this template](https://github.com/Netflix/metaflow-tools/blob/master/aws/cloudformation/metaflow-cfn-template.yml). You will have to copy the template to your laptop before you can upload it. Choose _Next._
5. Select your parameters for your deployment under _Parameters_ and choose _Next._
6. Feel free to tag your stack in whatever way best fits your organization. When finished, choose _Next._
7. The _Change set preview_ will log all the changes that this update to your CloudFormation stack will cause. If you were not already on the latest version, you will notice that there are a few additions that this update will result in - 
   1. _StepFunctionsRole -_ IAM role for AWS Step Functions
   2. _EventBridgeRole -_ IAM role for Amazon EventBridge
   3. _StepFunctionsStateDDB -_ Amazon DynamoDB table
   4. _ExecuteDBMigration_ - AWS Lamdba function for upgrading the RDS schema
   5. Updates to _BatchS3TaskRole_ and _ECSFargateService_ to allow for migration and AWS Step Functions integration.
8. Choose _I acknowledge that AWS CloudFormation might create IAM resources_ and choose _Update stack._
9. Wait for the stack to finish updating itself. This might take ~10 minutes.
10. Once the stack has updated, you would notice a new key _MigrationFunctionName_ which points to the AWS Lambda function that will upgrade your database schema. Note the name of this function.
11. Using either the [AWS Lambda console](https://console.aws.amazon.com/lambda) or AWS CLI, trigger the lambda function - 
    1. AWS Lambda console
       1. Choose the function that you just deployed in Step 10.
       2. In the dropdown for _Select a test event_, choose _Configure test events._
       3. In the resulting dialog, give a name to your event in _Event name_. The actual contents don't matter in this case. Choose _Create._
       4. Make sure you have taken a backup of your RDS instance before proceeding with the next step.
       5. Choose _Test._
       6. Check the execution result. In the resulting JSON blob, you should see `upgrade-result` set to `upgrade success` and `is_up_to_date` in `final-status` set to `true`. Congratulations! You have upgraded your database schema successfully. You can skip Step 7. and now let's upgrade the version of the metaflow service.
       7. If you saw a failure, [restore your RDS instance using the backup](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_RestoreFromSnapshot.html) that you had generated before. Please [get in touch](../../overview/getting-in-touch.md) with us so that we can figure out what went wrong.
    2. AWS CLI
       1. Make sure you have appropriate credentials to execute the AWS Lambda function on your laptop.
       2. Make sure you have taken a backup of your RDS instance before proceeding with the next step.
       3. Execute the command `aws lambda invoke --function-name <lambda-function-name> output.log` 
       4. Check the execution result. In the resulting JSON blob, you should see `upgrade-result` set to `upgrade success` and `is_up_to_date` in `final-status` set to `true`. Congratulations! You have upgraded your database schema successfully. You can skip Step 5. and now let's upgrade the version of the metaflow service.
       5. If you saw a failure, [restore your RDS instance using the backup](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_RestoreFromSnapshot.html) that you had generated before. Please [get in touch](../../overview/getting-in-touch.md) with us so that we can figure out what went wrong.
12. 


### Manual Deployment
