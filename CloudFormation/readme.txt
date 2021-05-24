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

SET Env=Prod
SET KeyName=kevinchanSMBC
aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EC2\devops\AnsibleAWX.yml --stack-name CM-SonarQube-%Env% --parameter-overrides KeyName=%KeyName% Env=%Env%

http://sonarqube-baf7250a6c5958fb.elb.us-east-1.amazonaws.com/about

#####
?aws cloudformation validate-template --template-body C:\downloads\github\aws\CloudFormation\EC2\devops\SonarQube.yml

SET Env=Prod1a
SET KeyName=kevinchanSMBC
SET sonarqubepassword=test12345

aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EC2\devops\SonarQube.yml --stack-name CM-SonarQube-%Env% --parameter-overrides KeyName=%KeyName% Env=%Env% SonarQubePassword=%sonarqubepassword%

SET Env=Prod1b
SET KeyName=kevinchanSMBC
SET sonarqubepassword=test12345
aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EC2\devops\AnsibleAWX.yml --stack-name CM-SonarQube-%Env% --parameter-overrides KeyName=%KeyName% Env=%Env% SonarQubePassword=%sonarqubepassword%

# NLB stack
# https://cloudaffaire.com/create-an-internal-network-load-balancer-with-static-private-ip-address/
SET Env=Prod
aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EC2\devops\NLB.yml --stack-name CM-SonarQube-NLB-%Env% --parameter-overrides Env=%Env%            

=======================
# SonarQube Cross Stack

SET Env=Dev
SET KeyName=kevinchanSMBC
SET sonarqubepassword=test12345
aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EC2\devops\SonarQube-CrossStack.yml --stack-name CM-SonarQube-%Env% --parameter-overrides KeyName=%KeyName% Env=%Env% SonarQubePassword=%sonarqubepassword%

###
SET Env=Prod1a
SET KeyName=kevinchanSMBC
SET sonarqubepassword=test12345
aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EC2\devops\SonarQube-CrossStack.yml --stack-name CM-SonarQube-%Env% --parameter-overrides KeyName=%KeyName% Env=%Env% SonarQubePassword=%sonarqubepassword%

SET Env=Prod1b
SET KeyName=kevinchanSMBC
SET sonarqubepassword=test12345
aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EC2\devops\SonarQube-CrossStack.yml --stack-name CM-SonarQube-%Env% --parameter-overrides KeyName=%KeyName% Env=%Env% SonarQubePassword=%sonarqubepassword%

SET Env=Prod
aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EC2\devops\NLB-CrossStack.yml --stack-name CM-SonarQube-NLB-CrossStack-%Env% --parameter-overrides Env=%Env%   

aws cloudformation describe-stack-resources --stack-name CM-SonarQube-Prod1a
aws cloudformation describe-stack-resources --stack-name CM-SonarQube-Prod1b

# Take EBS snapshot
aws ec2 create-snapshot --volume-id vol-1234567890abcdef0 --description "CM-SonarQube-Prod1a apps volume snapshot"
===============

###
# Create S3 bucket
set S3url=kchan-bucket-2021
aws s3api create-bucket --bucket %bkt% --region us-east-1
aws s3 cp SonarQube-Stack.cfn s3://%bkt%/devops/cloudformation/SonarQube-Stack.cfn
aws s3 cp SonarQube.yml s3://%bkt%/devops/cloudformation/SonarQube.yml

cd C:\downloads\github\aws\CloudFormation\EC2\devops\
SET Env=Prod
SET S3url=https://kchan-bucket-2021.s3.amazonaws.com/devops/cloudformation
echo %S3url%/SonarQube-Stack.cfn 
echo SonarQube-Stack-%Env%

aws cloudformation deploy --template-file %S3url%/SonarQube-Stack.cfn --stack-name SonarQube-Stack-%Env%

######

SET Env=Dev
SET KeyName=kevinchanSMBC
aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EC2\devops\AWSCDK.yml --stack-name CM-AWSCDK-%Env% --parameter-overrides KeyName=%KeyName% Env=%Env%

=================
Create a Chnage Set

aws cloudformation create-change-set --template-body file://changeset.yaml --stack-name CM-AWSCDK --capabilites CAPABILITY_NAMED_IAM

aws cloudformation describe-change-set --change-set-name changeset --stack-name CM-AWSCDK

aws cloudformation execute-change-set --change-set-name changeset --stack-name CM-AWSCDK
