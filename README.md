##Cloud Insight AWS Integrations    
In addition to the AWS environment exposure assessment Cloud Insight provides, we provide an open source project that allows users to extend available Cloud Insight results.

```ci_lambda_checks``` is a 'node js'-based AWS Lambda project that evaluates changes to your environment, and then publishes exposures to the Cloud Insight product. 
The ```ci_lambda_checks``` checks are evaluated in response to events received by the lambda function and supports the following modes:
- configurationItem - check is executed to evaluate a single change reported by AWS Config service.
- snapshotEvent - check is executed to evaluate the entire snapshot generated by AWS Config service.
- scheduledEvent - check is executed based on the AWS Lambda 'Scheduled Event'
- configRule - check is executed when an AWS Config Rule evaluation is reported by AWS Config service.
- inspectorEvent - check is executed when Amazon Inspector reports assessment completion.

Currently this project enables integrations with 'Amazon Inspector', 'AWS Config Rules', 'EC2' and 'VPC' services.

### Amazon Inspector Integration
```awsInspector``` is a check executed periodically, based on the AWS Lambda 'Scheduled Event' notifications. This check enumerates all Amazon Inspector findings generated by the Amazon Inspector service, converts the findings to Cloud Insight exposures, and then publishes the exposures, for the specified assets, to Cloud Insight.
    
**Note:** The check publishes exposures as a set. Each subsequent run of the check replaces the set of exposures published during the previous run of the check.
    
### AWS Config Rules Integration
```awsConfigRules``` is a check executed when a new AWS Config snapshot is generated, a single AWS environment change is reported by the AWS Config service, or when an AWS Config Rule evaluation is completed for an AWS resource. The check converts reported evaluation results to Cloud Insight exposures, based on the map specified in the ```awsConfigRules``` check's configuration within the 'config.js' file. The check then publishes the exposures, for the specified assets, to Cloud Insight.


### Custom Checks with Examples 
```ci_lambda_checks``` contains the following set of sample custom checks for users to extend the functionality of Cloud Insight with their own custom ```ci_lambda_checks```: 
 
- ```sg``` - This check evaluates the 'Security Group' configuration, and then publishes an exposure to Cloud Insight if a Security Group configuration does not match the specified criteria.
- ```namingConvention``` - This check evaluates the 'Name' tag value of an AWS Resource, and then publishes an exposure to Cloud Insight if a 'Name' tag value does not match the specified criteria.
- ```requiredTags``` - This check evaluates whether an AWS asset includes all specified tags key:value pairs, and then publishes an exposure to Cloud Insight if an AWS asset does not the match specified criteria.
- ```enableVpcScanning``` - This check evaluates whether the Cloud Insight appliance is able to scan an AWS instance, then and adds Alert Logic Security Protection Group to instances to allow Cloud Insight appliances to scan AWS instances.

## Setup
### Mac OS X  Installation Requirements
*~ You must install XCode and accept the licensing agreement before you continue with this document ~*  

Install [Homebrew](http://brew.sh/), which allows us to easily install and manage packages with dependencies.  
```$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"```  

Use [Homebrew](http://brew.sh/) to install [Node](http://nodejs.org/)  
```$ brew install node```  

**Note:** To run Lambda cli tools, you must install Javascript runtime.  

###Linux Installation Requirements  

Install the latest distribution of [Node](http://nodejs.org/) from [Distributions](https://nodejs.org/dist/v4.1.1/)  

**Note:** To run Lambda cli tools, you must install Javascript runtime.  

###Create Your Development Environment

1. To create your environment, clone this repository somewhere within your home directory. We recommend ~/workspace.   
```$ git clone git@github.com:alertlogic/ci_lambda_checks.git ci_lambda_checks```  
```$ cd ci_lambda_checks```  

2. Execute the Lambda development environment installation script.  
```$ build/install.sh```  

###Work in your environment   

The [NPM](https://www.npmjs.org/) install process that you ran earlier installed some [Node](http://nodejs.org/) modules that make the Lambda framework much more helpful than simple code checkouts.  Starting the framework will enable real-time linting, as well as the artifact build system.

##Build and deploy ```ci_lambda_checks``` to AWS  
You must have a valid account in Cloud Insight and have already set up a valid environment. In addition, you must correctly set up your your AWS Credentials for use with the AWS SDKs. Refer to [http://docs.aws.amazon.com/AWSSdkDocsJava/latest/DeveloperGuide/set-up-creds.html](http://docs.aws.amazon.com/AWSSdkDocsJava/latest/DeveloperGuide/set-up-creds.html).

1. Run ```npm run build``` to create a versioned, distributable zipped artifacts and guides you through deployment process.  
You will need to provide your Cloud Insight user name and password.  
2. (Optional) If you wish to update the version numbers.  
Run ```npm run release``` to update the version.

#### Changes made to your environment during deployment
- Creates an AWS IAM Role that grants the following permissions to AWS Lambda functions created by ```ci_lambda_checks```:
    - AWS Config read permissions
    - Amazon Inspector read permissions
    - AWS Lambda execution permissions
    - Add/Remove/Modify Security Groups and modify instance attributes
    - Read S3 bucket where AWS Config snapshots are stored
- Creates two lambda functions within each region supported by AWS Lambda.
    - driver - This function subscribes to AWS Config Service notifications, and contains the Cloud Insight account information used to create new exposures in Cloud Insight.
    - worker - The driver function calls the worker function to evaluate each change reported by AWS Config Service.  
**Note:** When a new AWS Config snapshot is generated, the worker function is called for each reported change.
- Configures AWS Config Service and makes sure that AWS Config service is configured to publish notifications and call AWS Lambda service  
**Note:** that if AWS Config service is already configured, the only deployment will only add the a permission to an IAM Role configured for AWS Service to call AWS Lambda to enable ```driver``` function to be successfully called when AWS Config service detects changes or generates a snapshot.
- Subscribes ```driver``` AWS Lambda function to the AWS Config SNS topic.
- Initiates AWS Config snapshot delivery.

#### Disable checks
To disable checks:

1. Set  the ```enable``` attribute to ```false``` in the ```config.js``` file.
2. Deploy ```ci_lambda_checks``` again.

#### Configure Amazon Inspector integration
1. You must configure Amazon Inspector ```Assessment Template``` to publish ```Run finished``` events to SNS topics.  
**Note:** For information on how to properly configure ```Assessment Template``` to send notifications to an SNS topic, see [https://docs.aws.amazon.com/inspector/latest/userguide/inspector_assessments.html](https://docs.aws.amazon.com/inspector/latest/userguide/inspector_assessments.html).
2. Add the SNS topic that ```Assessment Template``` uses as an ```Event Source``` to the  ```ci_checks_driver_XXXXXXXX``` AWS Lambda function, ```Event Sources```.  
**Note:** For information on how to set up SNS event sources for the AWS Lambda function, see [http://docs.aws.amazon.com/lambda/latest/dg/intro-core-components.html#intro-core-components-event-sources](http://docs.aws.amazon.com/lambda/latest/dg/intro-core-components.html#intro-core-components-event-sources).