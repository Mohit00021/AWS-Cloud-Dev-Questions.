# AWS-Cloud-Dev-Questions.
AWS dev questions and answers. 

### Use of API stage variable?

Answer: Stage variables act like environment variables for your API's deployment stages. They allow different configurations (URLs, settings) for each stage (dev, test, production) while using the same API setup.  In short, you can use stage variables in:
    API setup: Define different backend URLs based on the stage.
    Mapping templates: Access stage-specific settings to customize requests or responses.
    Lambda functions or HTTP backends: Pass configuration parameters based on the deployment stage.

### What's the use of API Gateway mapping Templates? 

Answer: Mapping templates are used to transform data or you can say "Format data".  A mapping template is a script expressed in Velocity Template Language (VTL) and applied to the payload using JSONPath
https://docs.aws.amazon.com/apigateway/latest/developerguide/models-mappings.html

### What is Amazon SQS?

Answer: Simple queue service... It is used to simplify the integration and separation of distributed software systems using a Queue 

### What's Traffic splitting? In context with deployments 

Answer: In traffic splitting or canary testing the  new application version is deployed on a temporary auto-scaling group.. this temporary autoscaling group would have the same amount of instances the main ASG has (same capacity) the traffic would be split in both the temp and the main ASG (10% - 90%) once the testing is done temp instances are migrated to the main ASG and old version is terminated. 
This is similar to Blue Green deployment(a better way maybe). 
Here are other methods/policies Elastic beanstalks has for deployment: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html

### What are Elastic Beanstalk extensions? 

Answer: EB extensions are used to customize elastic beanstalk environments, basically a user can customize the requirements of eb env beyond the normal eb settings. All the parameters set in UI can be configured using code files in a JSON/YAML format. We can connect external databases too with this.

