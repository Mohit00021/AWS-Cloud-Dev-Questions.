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
The queue contains messages, one or more producers can send a message to a queue and one/multiple consumers poll these messages. it's like a buffer between producers and messages

### What is message visibility timeout? in SQS

Answer: Message visibility timeout in SQS is a set amount of time that a message is hidden from other consumers if any one consumer has read the message,
so the queue has a message one consumer reads, the message will be invisible to the other consumers until the message is processed. If it's not processed in the set time then this message will be visible in the queue and other consumers can read it which will now result in more than one consumer processing the message, The default time a message would be invisible is 30 seconds the consumer can change this by calling **ChangeMessageVisibility** API which will mean that the message needs more time to process and keep it invisible for given time. default value can also be set manually in the SQS settings

### Dead letter queue in SQS?

Answer: Dead letter queues (DLQs) can be called storage for failed messages in SQS (messages that weren't processed successfully). A failed message reappears in the main queue, which can result in an infinite loop if there's a problem with the message itself. It would just keep appearing again and again. To solve this, we can set a threshold value for how many times a message can return to the main queue. With a "MaximumReceive" count set to 3, if the message fails for the 4th time, it would then be sent to the dead letter queue.

Redrive to source feature of DQL: These failed messages can be sent back to the main queue once you fix the root cause of their failure.

### Explain Long Polling in SQS 

Answer: To receive a message present in the queue a consumer polls for messages, and long polling allows a consumer to "wait" for messages for a longer period if there are no messages in the queue. Long Polling can help decrease the application latency by reducing the number of API calls made to SQS. Wait time can be set to 0 - 20 seconds. 

Long polling can be enabled at Queue settings and also by calling ReceiveMessageWaitTimeSeconds API

### Extended Client feature of SQS

Answer: Using an extended client we can send messages that are larger in size say 1Gb through SQS, An S3 bucket is also used here which acts as a message store and messages in the main queue would only contain Metadata.

### What's Traffic splitting? In context with deployments 

Answer: In traffic splitting or canary testing the  new application version is deployed on a temporary auto-scaling group.. this temporary autoscaling group would have the same amount of instances the main ASG has (same capacity) the traffic would be split in both the temp and the main ASG (10% - 90%) once the testing is done temp instances are migrated to the main ASG and old version is terminated. 
This is similar to Blue Green deployment(a better way maybe). 
Here are other methods/policies Elastic beanstalks has for deployment: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html

### What are Elastic Beanstalk extensions? 

Answer: EB extensions are used to customize elastic beanstalk environments, basically a user can customize the requirements of eb env beyond the normal eb settings. All the parameters set in UI can be configured using code files in a JSON/YAML format. We can connect external databases too with this.

### What is AWS Cloud Formation? 

Answer: Cloud formations are templates that are written in JSON or YAML format. basically, create resources with different properties by just using code plus you can reuse these templates.

### What is the type identifier for "Resources" in cloud formation templates

Answer: ***service-provider::service-name::data-type-name***

Example: ![image](https://github.com/Mohit00021/AWS-Cloud-Dev-Questions./assets/53579940/90a09727-af4d-443f-9782-2be1ee3ceb56)

*** Resources example ***
```yaml
Resources:
    MyEC2Instance:
        Type: AWS::EC2::Instance
        Properties:
            InstanceType: t2.micro
            ImageId: ami-blablabla
```

### Cloud formation: What are the parameters and when to use them?

Answer: In CF Parameters are parameters(coding). In Cloud Formation we use parameters are a way to provide input to templates. Parameters should be used when resource configurations are likely to be changed in the future.
Parameters can be accessed by using the *"!Ref"* function of AWS cloud formation.

Note:!Ref is not limited to parameters only.

Example:
```yaml
Parameters:
    InstanceType:
        Description: Choose an instance type.
        Type: String
        AllowedValues:
            -t2.micro
            -t2.small
            -t2.medium
        Default: t2.small
Resources:
    MyEC2Instance:
        Type: AWS::EC2::Instance
        Properties:
            InstanceType: !Ref InstanceType  -----> "!Ref" is used to access the parameter here
            ImageId: ami-blablabla
```

### Cloud Formation: what are mappings? 

Answer: Mappings are hardcoded variables aka fixed variables in cloud formation templates. We can access Mappings anywhere in the template using the fn::FindInMap function shortform !FindInMap 

Example:
```yaml
Mappings:
    RegionMap:
        us-east-1:
            HVM64: ami-blablabla
            hVMG2: ami-blalblbla
        us-west-2:
            HVM64: ami-blablabla
            hVMG2: ami-blalblbla
Resources:
    MyEC2Instance:
        Type: AWS::EC2::Instance
        Properties:
            ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", HVM64]
            ImageId: ami-blablabla
```
### Cloud Formation: Outputs 

Answer: Outputs are optional values that must be exported in order to use them. we can import these values into other stacks! `(To understand the reference of stack here Google how cloud formation works or read this  https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#cfn-concepts-stacks)`, basically a stack is created when a template is used and then the events stored in stack are triggered followed by resources creation.. outputs example -> we can export vpc id from a SSH security stack and ref it in a application stack 

```yaml
Outputs:
    SSHSecurityGroup:
        Description: The ssh security group for our organization
        Value: !Ref MySSHGroup
        Export:
            Name: SSHSecurityGroup
Resources:
    MyEC2Instance:
        Type: AWS::EC2::Instance
        Properties:
            InstanceType: t2.micro
            ImageId: ami-blablabla
            SecurityGroups:
                - !ImportValue SSHSecurityGroup  ------> "fn::ImportValue" is used to import the outputs
```
*Note You can only delete outputs if there are no usages of it*

### Cloud Formation: Conditions 

Answer: Create resources based on conditions, and logical functions to be used in Cloud formation conditions fn::AND/IF/EQUALS/OR/NOT. the short form for these functions looks like this "!Equals" this "!" here does not mean Not so don't get confused! it's a short form of "fn::"

```yaml
Conditions:
    CreateProdResource: !Equals [!Ref EnvType, prod]
```

### Intrinsic Functions in Cloud Formation

Answer: If you read the above examples you already know a few intrinsic functions.. "!Ref" , "!Equals" "!FindInMap" there are many also cf has a fn::ForEach too! read this -> https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html








