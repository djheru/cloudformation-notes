# AWS CloudFormation Master Class

## Introducing CloudFormation

### What is CloudFormation

- A service to declaratively define AWS infrastructure using JSON or YAML templates
- CloudFormation creates it for you, in the correct order, with the exact configuration you specify

### Benefits of CloudFormation

- Infrastructure as code
    - No resources are created manually
    - Code is version controlled
    - Infrastructure changes are managed
- Cost
    - Each resource within the stack is tagged with an identifier, so you can calculate how much a stack costs
    - You can estimate the cost of resources using CloudFormation templates
    - You can automate infrastructure changes to save money by, for example, deleting dev stacks when not in use
- Productivity
    - Can make infrastructure changes quickly
    - Can generate diagrams for templates
    - Declarative programming, no need to figure out ordering and orchestration
- Separation of concerns
- Don't reinvent the wheel
    - Use existing templates
    
### Cost Estimation

- Can upload a template
- Use the "Simple Monthly Calculator"

### CloudFormation vs Ansible/Terraform

- AWS Native
- CloudFormation is state based, AWS figures out how to reach that state
- Ansible/Terraform are instruction based, need to manually orchestrate stacks

### CloudFormation Building Blocks

- Template Components
    1. Resources: AWS resources declared in the template
    2. Parameters: Inputs for the template
    3. Mappings: The static variables for the template
    4. Outputs: References to resources used by other templates
    5. Conditionals: List of conditions to consider when creating resources
    6. Metadata
- Template helpers
    1. References
    2. Functions

### Introductory Example 

- EC2 instance
- Add Elastic IP
- Add 2 security groups

1. Go to CloudFormation in the AWS Console
2. Select "Create Stack"
3. Select template from "Upload a Template"
4. Upload `0-just-ec2.yaml`
5. Select a "stack name", e.g. "Introduction"
    - Stacks are identified by the name
6. Add a tag, e.g "course: CloudFormation" - Will be used later
7. Review, then create
8. View the different tabs, Events, Resources, OUtputs, Parameters, Template
9. Edit the template, by selecting "Update Stack" and upload a new file, `1-ec2-with-sg-eip.yaml`
10. Preview your changes, click "Update"
11. See the new resources are created
12. See the events are listed
13. See the EC2 instance was terminated and replaced with a new one
14. Clean up
    - Go back to cloudFormation and delete the stack
    - It deletes everything in the correct sequence
    
## CloudFormation Examples with S3

- Check docs here for available fields: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html
- See 2-creating-s3-bucket.yaml for minimal example yaml
    - We define resources in `Resources` top level key
    - The only required field here is `Type: "AWS::S3::Bucket"`
    - Upload it in the CloudFormation AWS console as before
- We can update it - Two different types
    - Updates with no interruption - Adding AccessControl
    - Replacement updates, such as updating the name of the bucket
- Update `AccessControl` 
    - `AccessControl: PublicRead` added under `Properties` (`3-adding-access-control.yaml`)
    - Go to CloudFormation in the console
    - Select the stack and then select the "Update Stack" action from the menu
    - `Replace current template` and upload the new one
    - Click next through the screens, see the "Change set preview" 
        - `Replacement: false`
- Update `BucketName`
    - `BucketName: some-unique-bucket-name`
    - Update Stack as before
    - See "Change set preview"
    - `Replacement: true` - **CloudFormation will not back up the data in your bucket on replacement!**
- Delete the stack

## Template Options

Parameters that are common to any CloudFormation template:

- Tags
    - Every resource in the stack will have these tags
- Permissions
- Notification Options
    - Emails/SNS Topics
- Timeouts
- Rollback on Failure
 - Can select "No" to debug failures
- Stack Policy
    - Protect certain resources so they are not deleted

## Using CloudFormation Designer

- From CloudFormation console homepage, click "Designer"
- Can do drag and drop
    - Can right-click dragged element, representing a resource, select info
    - Mouse over dots to see the connections that are available/required for a resource  
- Can paste a found template into the template to view it visually in designer
- Click the "Validate Template" button (Top menu, checkbox icon) 
- Click the "Cloud Upload" button to continue on to build the stack from the template

## CloudFormation Parameters

- https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html
- Allow you to provide inputs to CloudFormation templates
- Allow you to reuse templates
- Are typed
- Use if Resource configuration is likely to change
- Defined using the following:
    - Type
        - String, Number, CommaDelimitedList
        - List<Type>
        - AWS Parameters, e.g. reference to existing resource
    - Description
    - Constraints
    - ConstraintDescription
    - Min/MaxLength
    - Min/MaxValue
    - Defaults
    - Allowed Values (array)
    - AllowedPattern (regex)
    - NoEcho - Boolean
- See `6-parameters-hands-on.yaml` for example
- To reference a parameter: `!Ref ParameterName` or `!Ref ResourceName`
- To reference an element from a list: `!Select [0, !Ref ParameterName]`
- `!Ref` is a shorthand for `Fn::Ref`, a function

### Pseudo Parameters

- `AWS:AccountId`
- `AWS::NotificationARNs`
- `AWS::NoValue`
- `AWS::Region`
- `AWS::StackId`
- `AWS::StackName`

## Resources

- https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html
- The core of the CloudFormation template
- Example see `7-ec2-with-sg-eip.yaml`
- Represent the various AWS Components that will be created/configured
- Resources are declared and can reference each other
- The properties of each type of resource are defined in the docs
- Resource types are of the form `AWS::aws-product-name::data-type-name` 
    - e.g. `AWS::EC2::Instance`
- Additional attributes 
    - DependsOn
    - DeletionPolicy
    - CreationPolicy
    - Metadata

## Mappings

- Fixed variables within your CloudFormation template
- Can be used to differentiate between different environments, regions, etc
- To access Mapping values, we use `!FindInMap [MapName, TopLevelKey, SecondLevelKey]`
- Example see `8-mappings-ec2.yaml`

## Outputs

- Enables cross-stack references
- Declares optional output values that can be imported into other stacks
- Example see `09-create-ssh-security-group.yaml` for a template for a Security Group resource
    - Exposes SG as such: 
```yaml
Outputs:
  StackSSHSecurityGroup:
    Description: The SSH Security Group for our Company
    Value: !Ref MyCompanyWideSSHSecurityGroup
    Export:
      Name: SSHSecurityGroup
```
- Example see `10-reference-ssh-security-group.yaml` for template that references the above template
    - Imports SG as such:
```yaml
Resources:
  MySecureInstance:
    # http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-a4c7edb2
      InstanceType: t2.micro
      SecurityGroups:
        # we reference the output here, using the Fn::ImportValue function
        - !ImportValue SSHSecurityGroup
```
- CloudFormation will prevent deletion of stacks that are imported by other stacks

## Conditionals

- Control flow for creation of resources or outputs
- Common conditions
    - ENV
    - AWS Region
    - Any parameter value
- Defining a condition
```yaml
Conditions:
    LogicalID:
      Intrinsic function

```
- Intrinsic functions (Conditions):
    - Fn::And
    - Fn::Equals
    - Fn::If
    - Fn::Not
    - Fn::Or
    - Docs: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-conditions.html
- Other intrinsic functions
    - https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html
    - `Fn::GetAtt` - Get the attribute attached to any resource
        - Long Form: `Fn::GetAtt: [logicalNameOfResource, attributeName]`
        - Short Form: `!GetAtt logicalNameOfResource.attributeName`
    - `Fn::Sub` - Substitutes variable from text like `${VariableName}`
- See example: 11-conditions.yaml
    - Define condition
    ```yaml
    Conditions:
      CreateProdResources: !Equals [ !Ref EnvType, prod ]
    ```

    - Use condition value
    
    ```yaml
      MountPoint:
        Type: "AWS::EC2::VolumeAttachment"
        Condition: CreateProdResources
        Properties:
          InstanceId:
            !Ref EC2Instance
          VolumeId:
            !Ref NewVolume
          Device: /dev/sdh
    
      NewVolume:
        Type: "AWS::EC2::Volume"
        Condition: CreateProdResources
        Properties:
          Size: 100
          AvailabilityZone:
            !GetAtt EC2Instance.AvailabilityZone
    ```

## Metadata

- Optional
- Example:
```yaml
Metadata:
    Instances:
      Description: "Information about the instances"
    Databases:
      Description: "Information about the databases"
``` 
- Used by CloudFormation Designer 
    - Describes how resources are laid out in the template canvas, position/size of icons
- Used by the AWS CloudFormation Console
    - Defines the grouping and ordering of input parameters in the console
- See example: `13-interface.yaml`

## CFN-Init and EC2-user data

### EC2 User Data

- A script that gets executed at first boot of an instance
- Example PHP install script: `14-ec2-user-data.sh`
- Example of script within a CloudFormation template: `15-ec2-user-data.yaml`
    - Notice `UserData` under `WebServer`
- Logs go to /var/log/cloud-init-output.log

### CloudFormation Helper Scripts

- 4 Python scripts preinstalled on the Amazon AMI or can be installed using Yum
    - `cfn-init` - Retrieve and interpret resource metadata, install packages, create files, start services
    - `cfn-signal` - Signals to CloudFormation CreationPolicy or WaitCondition to enable orchestration
    - `cfn-get-metadata` - Wrapper script to help retrieve all or some of the resource metadata
    - `cfn-hup` - Daemon to check for updates to metadata and execute hooks when changes are detected

#### AWS::CloudFormation::Init

- And such as...

```yaml
Resources:
    MyInstance:
      Type: "AWS::EC2::Instance"
      Metadata:
        AWS::CloudFormation::Init:
          config:
            packages: # Packages to install
            groups: # User groups
            users: # Users
            sources: # Download files to the instance
            files: # Create files on the instance
            commands: # Commands to run
            services: # launch a list of sysvinit
    Properties:
```
- Can use multiple config blocks

##### Packages

- can install from any of the following repositories
    - apt
    - msi
    - python
    - rpm
    - rubygems
    - yum
- Processed in the following order:
    - rpm, yum/apt, rubygems, python
    - Specify an older version if you like

- Example config packages section

```yaml
Resources:
    MyInstance:
      Type: "AWS::EC2::Instance"
      Metadata:
        AWS::CloudFormation::Init:
          config:
            packages:
              rpm: 
                epel: "http://download.fedoraproject.com/pub/epl/"
              yum:
                httpd: [] # [] = latest version
                php: []
                wordpress: []
              rubygems:
                chef:
                  - "0.10.2"
```

##### Groups and Users

```yaml
Resources:
    MyInstance:
      Type: "AWS::EC2::Instance"
      Metadata:
        AWS::CloudFormation::Init:
          config:
            groups:
              groupOne: {} # Uses any groupId
              groupTwo:
                gid: "45"
            users:
              myUser:
                groups: 
                  - "groupOne"
                  - "groupTwo"
                uid: "50"
                homeDir: "/tmp"
```

##### Sources 

- Download scripts from the interwebz

```yaml
Resources:
    MyInstance:
      Type: "AWS::EC2::Instance"
      Metadata:
        AWS::CloudFormation::Init:
          config:
            sources:
              "/home/ec2-user/aws-cli": "https://github.com/aws/aws-cli/tarball/master":
```

##### Files

- Files are powerful
- Can be drawn from a URL or be written inline

```yaml
Resources:
    MyInstance:
      Type: "AWS::EC2::Instance"
      Metadata:
        AWS::CloudFormation::Init:
          config:
            files:
              "/tmp/cwlogs/apacheaccess.conf":
                content: !Sub |
                  [general]
                  state_file= /var/awslogs/agent-state
                  [/var/log/httpd/access_log]
                  file = /var/log/httpd/access_log
                  log_group_name = ${AWS::StackName}
                  log_stream_name = {instance_id}/apache.log
                  datetime_format = %d/%b/%Y:%H:%M:%S
                mode: '000400'
                owner: apache
                group: apache
              "/var/www/html/index.php":
                content: !Sub |
                  <?php echo '<h1>AWS CloudFormation PHP Sample for ${AWS::StackName}</h1>'; ?>
                mode: '000644'
                owner: apache
```

##### Commands

- Runs `command`s one at a time in **alphabetical order**
- Can set `env` vars
- Can set a directory `cwd` from which the commands are run
- Can provide a `test` to control whether or not the command is executed
- Has an `ignoreErrors` attribute

##### Services

- Launch services at instance launch
    - Or make sure a default service doesn't run
- Ensure services are restarted when files change or packages are updated by `cfn-init`

#### Calling the helpers

- See example: `16-cfn-init.yaml`
- `cfn-signal` - Use `cfn-init` first to launch the configuration
    - when configuration has been completed, lets CloudFormation know the resource was created successfully
    - Used in conjunction with CreationPolicy
- `cfn-hup` - Listens for metadata changes
- Logs go to /var/log/cfn-init.log

## More Advanced Techniques

https://github.com/awslabs/aws-cloudformation-templates

### Using AWS CLI

- Use it ot create, update, delete CloudFormation templates
- Use to manage parameters
- To install - `pip install awscli --upgrade --user`
- Configure profile
    - Generate security credentials
        - Create new Access Key
        - Download Access Key and Secret
        - Run `aws configure --profile profileName`
            - Enter Access Key ID
            - Enter Secret Access Key
            - Region Default
            - Output Default

#### Deploy CloudFormation Stack

- https://docs.aws.amazon.com/cli/latest/reference/cloudformation/index.html#cli-aws-cloudformation
- Run `aws cloudformation` subcommand
- Create new stack

```bash
aws cloudformation create-stack \
    --stack-name example-cli-stack \
    --template-body file://18-sample-template.yaml \
    --parameters file://19-parameters.json \
    --region us-east-1 \
    --profile profileName
```
- Returns the `StackId` in the cmd output

- Update stack

```bash
aws cloudformation update-stack \
    --stack-name example-cli-stack \
    --template-body file://18-sample-template.yaml \
    --parameters file://19-parameters.json

```

### DeletionPolicy

- Values:
    - `Delete` - Deletes the resource and all of its content if applicable during stack deltion
    - `Retain` - AWS CloudFormation keeps the resource without deleting the resource or its contents when the stack is deleted
    - `Snapshot` - Some resources support snapshots
        - AWS::EC2::Volume
        - AWS::ElastiCache::CacheCluster
        - AWS::RDS::DBInstance
        - AWS::RDS::DBCluster
        - A few others
    
### Custom Resources using AWS Lambda

- Allows you to write custom provisioning logic in templates that AWS CloudFormation run anytime create/update/delete stacks
- Custom resources can be provisioned using an AWS Lambda function
- https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources.html
- https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/walkthrough-custom-resources-lambda-lookup-amiids.html

### Best Practices for CloudFormation

- Organizing templates
    - Layered architecture (horizontal layers)
        - Network layer, instance layer, application layer
    - Service-oriented architecture (vertical layers)
        - Network, instance, application layers for a single application
- Use cross-stack references to enable access to resources in other templates
- Make sure the template is environment agnostic so you can do dev/test/prod
- Do not embed credentials
    - Use `NoEcho` with sensitive parameters
- Use specific parameter types and constraints where possible
- Use CFN init
- Validate templates before use
- Don't do manual operations on stack resources
- Verify changes with changesets 
- Use stack policies and deletion policies
    
