https://github.com/awslabs/aws-cloudformation-templates
https://aws.amazon.com/blogs/devops/passing-parameters-to-cloudformation-stacks-with-the-aws-cli-and-powershell/
https://docs.aws.amazon.com/cli/latest/reference/cloudformation/index.html

aws cloudformation deploy --template-file <value> --stack-name <value> 

aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation.EC2\devops\EC2.yml --stack-name CM-SonarQube --parameter-overrides KeyName=kevinchanSMBC 

InstanceType=t2.micro

======
