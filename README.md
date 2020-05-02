## ec2-image-builder
### What is EC2 Image Builder?
EC2 Image Builder is a fully managed AWS service that makes it easier to automate the creation, management, and deployment of customized, secure, and up-to-date “golden” server images that are pre-installed and pre-configured with software and settings to meet specific IT standards. 

### Features of EC2 Image Builder 
- Increase productivity and reduce operations for building compliant and up-to-date images
- Increase service uptime
- Raise the security bar for deployments
- Centralized enforcement and lineage tracking

### Supported operating systems
Image Builder supports the following operating systems:
- Amazon Linux 2
- Windows Server 2019/2016/2012 R2
- Windows Server version 1909
- Red Hat Enterprise Linux (RHEL) 8 and 7
- CentOS 8 and 7
- Ubuntu 18 and 16
- SUSE Linux Enterprise Server (SUSE) 15

### How EC2 Image Builder works
When you use the EC2 Image Builder console to create a golden image, a wizard guides you through the following steps.
1. **Select source image.** You select a source OS image, for example, an existing AMI.
2. **Create image recipe.** You add components to create an image recipe for your image pipeline. Components are the building blocks that are consumed by an image recipe, for example, packages for installation, security hardening steps, and tests. The selected OS and components make up an image recipe. Components are installed in the order in which they are specified and cannot be reordered after selection. 
3. **Output.** Image Builder creates an OS image in the selected output format. 
4. **Distribute.** You distribute your image to selected AWS Regions after it passes tests in the image pipeline.

### Prerequisites
- **EC2 Image Builder service-linked role**
    - EC2 Image Builder uses a service-linked role to grant permissions to other AWS services on your behalf. You don't need to manually create a service-linked role. When you create your first Image Builder resource Image Builder creates the service-linked role for you.
    
- **Auto Scaling groups**
    - EC2 Image Builder uses Auto Scaling groups to launch instances during the build and test phases of the image pipeline. When you use Amazon EC2 Auto Scaling, a required service-linked role is created in your account. If this role is not present in your account when you use Image Builder, the Image Builder service-linked role will create if for you.
    
- **Configuration requirements**
    - EC2 Image Builder does not support encrypted AMIs as the source for or output image of a pipeline.
    - You must specify a VPC in the infrastructure configuration. Image Builder does not support EC2-Classic. 
    - Image Builder does not support Amazon VPC endpoints (PrivateLink).
    - Instances used to build images and run tests using Image Builder must have access to the Systems Manager service. All build activity is orchestrated by SSM Automation. The SSM Agent will be installed on the source image if it is not already present, and it will be removed before the image is created.
    
- **AWS Identity and Access Management (IAM)**
    - The IAM role that you associate with your instance profile must have permissions to run the build and test components included in your image.
    - The following IAM role policies must be attached to the IAM role that is associated with the instance profile:
      - `EC2InstanceProfileForImageBuilder`
      - `AmazonSSMManagedInstanceCore`
    - If you configure logging, the instance profile specified in your infrastructure configuration must have `s3:PutObject` permissions for the target bucket (`arn:aws:s3:::<BucketName>/*`).

### Document Sections
The sections of a document are as follows.
- **Phases.** Phases are a logical grouping of steps.
  - Each phase name must be unique within a document.
  - You can define many phases in a document.
  - Image Builder executes phases called build, validate, and test in the image build pipeline.
- **Steps.** Steps are individual units of work that comprise the workflow for each phase.

  - Each step must define the action to take.
  - Each step must have a unique name per phase.
  - Steps are executed sequentially.
  - Both the input and output of a step can be used as inputs for a subsequent step (“chaining”).
  - Each step uses an action module that returns an exit code.
  
- **Supported Actions.** Supported actions are the actions that each step must contain in a document. Each supported action correlates to an action module. Below is the complete list of supported action modules:
  - `ExecuteBinary`
  - `ExecuteBash`
  - `ExecutePowerShell`
  - `Reboot`
  - `UpdateOS`
  - `S3Upload`
  - `S3Download`
  - `SetRegistry`

### Input and Output Chaining
The configuration management application provides a feature for chaining inputs and outputs by writing references in the following formats:<br>
`{{ phase_name.step_name.inputs/outputs.variable }}`<br>
or<br>
`{{ phase_name.step_name.inputs/outputs[index].variable }}`<br>
The chaining feature allows you to recycle code and improve the maintainability of the document.

**The usage requirements of chaining are as follows:**
- Chaining expressions can be used only in the inputs section of each step.
- Statements with chaining expressions must be enclosed in quotes. For example:
  - **Invalid expression:** `echo {{ phase.step.inputs.variable }}`
  - **Valid expression:** `"echo {{ phase.step.inputs.variable }}"`
  - **Valid expression:** `'echo {{ phase.step.inputs.variable }}'`
- Chaining expressions can reference variables from other steps and phases in the same document. 
- Indexes in chaining expressions follow 0-based indexing (first index is 0).

### Security Best Practices for EC2 Image Builder
EC2 Image Builder provides a number of security features to consider as you develop and implement your own security policies. The following best practices are general guidelines and don’t represent a complete security solution. Because these best practices might not be appropriate or sufficient for your environment, treat them as helpful considerations rather than prescriptions. 
- Do not use overly-permissive security groups in Image Builder recipes.
- Do not share images with accounts that you do not trust.
- Do not make images public that have private or sensitive data.
- Apply all available Windows or Linux security patches during image builds.
