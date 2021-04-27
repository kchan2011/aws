https://github.com/awslabs/aws-cloudformation-templates
https://aws.amazon.com/blogs/devops/passing-parameters-to-cloudformation-stacks-with-the-aws-cli-and-powershell/
https://docs.aws.amazon.com/cli/latest/reference/cloudformation/index.html

aws cloudformation deploy --template-file <value> --stack-name <value> 

SET Env=POC
SET KeyName=kevinchanSMBC
aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EC2\devops\SonarQube.yml --stack-name CM-SonarQube --parameter-overrides KeyName=%KeyName% Env=%Env%

InstanceType=t2.micro
======

SET Env=POC
SET KeyName=kevinchanSMBC
aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\EFS\devops\SonarQube.yml --stack-name CM-SonarQube-EFS --parameter-overrides Env=%Env%

aws cloudformation describe-stack-events --stack-name CM-SonarQube 
======
