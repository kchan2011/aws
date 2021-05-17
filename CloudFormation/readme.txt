https://github.com/awslabs/aws-cloudformation-templates
https://aws.amazon.com/blogs/devops/passing-parameters-to-cloudformation-stacks-with-the-aws-cli-and-powershell/
https://docs.aws.amazon.com/cli/latest/reference/cloudformation/index.html

SET Env=Dev
SET KeyName=kevinchanSMBC
aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EC2\devops\EC2_ubuntu.yml --stack-name CM-EC2-%Env% --parameter-overrides KeyName=%KeyName% Env=%Env%

# Override to use CIS Ubuntu Linux 20.04 LTS Benchmark
SET EC2Image="CIS Ubuntu Linux 20.04 LTS Benchmark"
aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EC2\devops\EC2_ubuntu.yml --stack-name CM-EC2-%Env% --parameter-overrides KeyName=%KeyName% Env=%Env% EC2Image=%EC2Image%

==========
# To install W2019 using Amazon W2019 Base AMI
aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EC2\devops\EC2_Windows.yml --stack-name CM-EC2-%Env% --parameter-overrides KeyName=%KeyName% Env=%Env%


=========================
aws cloudformation deploy --template-file <value> --stack-name <value> 

SET Env=POC
SET KeyName=kevinchanSMBC
aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EC2\devops\SonarQube.yml --stack-name CM-SonarQube-%Env% --parameter-overrides KeyName=%KeyName% Env=%Env%


SET Env=Dev
SET KeyName=kevinchanSMBC
aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EC2\devops\AnsibleAWX.yml --stack-name CM-SonarQube-%Env% --parameter-overrides KeyName=%KeyName% Env=%Env%

======

SET Env=Dev
SET KeyName=kevinchanSMBC
aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EC2\devops\AWSCDK.yml --stack-name CM-AWSCDK-%Env% --parameter-overrides KeyName=%KeyName% Env=%Env%

=================
Create a Chnage Set

aws cloudformation create-change-set --template-body file://changeset.yaml --stack-name CM-AWSCDK --capabilites CAPABILITY_NAMED_IAM

aws cloudformation describe-change-set --change-set-name changeset --stack-name CM-AWSCDK

aws cloudformation execute-change-set --change-set-name changeset --stack-name CM-AWSCDK
