# Create_private_certificate
# https://cloudaffaire.com/create-an-internal-network-load-balancer-with-static-private-ip-address/

SET Env=Prod
SET Apps=SonarQube
SET R53DomainName=www.devops123.com

aws cloudformation deploy --template-file C:\downloads\github\aws\CloudFormation\devops\ACM\create_private_certificate.yml --stack-name CM-%Apps%-PCA-%Env% --parameter-overrides Env=%Env% R53DomainName=%R53DomainName% --profile admin            
