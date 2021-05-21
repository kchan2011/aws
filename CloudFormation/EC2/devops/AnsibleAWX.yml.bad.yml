AWSTemplateFormatVersion: '2010-09-09'
Description: 'Create an EC2 instances running a Ubuntu AMI to host AWX containers. 
One EBS volume will be created and mounted to the active EC2 instance with running containers.  The AMI is chosen based on the region in which the stack is run.'
Metadata:
  AWS::CloudFormation::Interface: 
    ParameterGroups: 
      - Label: 
          default: "Network Configuration"
        Parameters: 
          - Vpc
          - PrivateSubnet1a
          - PrivateSubnet1b
          - SecurityGroupID
          - SSHLocation
      - Label: 
          default: "Amazon EC2 Configuration"
        Parameters:
          - Env
          - KeyName
          - LatestAmiId
    ParameterLabels: 
      Vpc: 
        default: "Which VPC should this be deployed to?"

Parameters:
  Env:
    Description: Application Envirnoment Type
    Type: String
    ConstraintDescription: must specify POC, Test, Dev, QA, UAT, PreProd, Staging, Prod1a, Prod1b.
    #Default: Dev
    AllowedValues: [Dev, UAT, Prod1a, Prod1b]
  KeyName:
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instance
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: must be the name of an existing EC2 KeyPair.
  Vpc:
    Type: "List<AWS::EC2::VPC::Id>"
    Default: vpc-07c9b85d62b6f4a7f
    Description: VpcId of your existing Virtual Private Cloud (VPC)
    ConstraintDescription: must be the VPC Id of an existing Virtual Private Cloud.
  PrivateSubnet1a:
    # Type: AWS::EC2::Subnet::Id
    Type: String
    Default: subnet-0c6790c3ecaffce32 # us-east-1a
    Description: SubnetId of an existing subnet in 1 in your VPC
    ConstraintDescription: must be an existing subnet in the selected Virtual Private Cloud.
  PrivateSubnet1b:
    # Type: AWS::EC2::Subnet::Id
    Type: String
    Default: subnet-0d5faa025dde79b24 # us-east-1b
    Description: SubnetId of an existing subnet in AZ2 in your VPC
    ConstraintDescription: must be an existing subnet in the selected Virtual Private Cloud.
  SSHLocation:
    Description: The IP address range that can be used to SSH to the EC2 instances
    Type: String
    MinLength: 9
    MaxLength: 18
    # Default: 0.0.0.0/0
    Default: 67.82.52.94/32
    AllowedPattern: (\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/(\d{1,2})
    ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x.
  EC2Image:
    Type: String
    Default: 'Ubuntu Pro 20.04 LTS'
    ConstraintDescription: Select EC2 Instance AMI
    AllowedValues: ['CIS Ubuntu Linux 20.04 LTS Benchmark', 'Ubuntu Pro 20.04 LTS']
  # ImageId:
  #   Type: String
  #   Default: ami-0ed28656d62ce20d6
  #   Description: AMI ID
  #   # ImageId: ami-0de8ae0e6d86b93c8  # CIS Ubuntu Linux 20.04 LTS Benchmark
  #   # ImageId: ami-0ed28656d62ce20d6  # Ubuntu Pro 20.04 LTS
  EC2Iam:
    Type: String
    Default: EC2SSM
    ConstraintDescription: Type in EC2 IAM
  SonarQubePassword:
    Type: String
    Default: Sonar
    ConstraintDescription: Type in EC2 IAM
Conditions:
  CreateDevResources:    !Equals [!Ref Env, "Dev"]
  CreateUATResources:    !Equals [!Ref Env, "UAT"]
  CreateProd1aResources: !Equals [!Ref Env, "Prod1a"]
  CreateProd1bResources: !Equals [!Ref Env, "Prod1b"]
  CreateProdResources:   !Or [!Equals [!Ref Env, "Prod1a"], !Equals [!Ref Env, "Prod1b"]]

  CreateEC2_Ubuntu20.04LTS_CIS: !Equals [!Ref EC2Image, "CIS Ubuntu Linux 20.04 LTS Benchmark" ]
  CreateEC2_Ubuntu20.04LTS:     !Equals [!Ref EC2Image, "Ubuntu Pro 20.04 LTS" ]

Resources:
  # https://computingforgeeks.com/configure-aws-application-load-balancer-with-cloudformation/
  # https://stackoverflow.com/questions/63118516/aws-cloudformation-target-group-for-application-load-balancer-is-not-working-fo

  EC2SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: SonarQube SG
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 9000
          ToPort: 9000
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: icmp
          FromPort: -1
          ToPort: -1
          CidrIp: 0.0.0.0/0
        # - IpProtocol: tcp
        #   FromPort: 5432
        #   ToPort: 5432
        #   CidrIp: 0.0.0.0/0
      VpcId: vpc-07c9b85d62b6f4a7f

  EC2Instance1:
    Type: AWS::EC2::Instance
    # https://linuxtut.com/en/8ff718cd1e66778f88b5/
    # https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-init.html
    # https://aws.amazon.com/premiumsupport/knowledge-center/cloudformation-helper-scripts-windows/
    # https://stackoverflow.com/questions/39760158/ubuntu-could-not-enable-service-cfn-hup
    Metadata:
      AWS::CloudFormation::Init:
        configSets:
          full_install: [ "install_apps", "cfn-hup" ]
        install_apps:          
          packages: 
            apt:
              net-tools: []
              inetutils-traceroute: []
              apache2: []
              tmux: []
              mc: []
              nvme-cli: []
              docker.io: []
              docker-compose: []
          files: 
            /var/www/html/index.html: 
              content: !Sub |
                Hello World ${Env} - ${EC2Instance1}
            /home/root/docker-compose.yml:
              content: !Sub |
                version: "3"
                services:
                  sonarqube:
                    container_name: sonar-sonarqube
                    image: sonarqube:8.5.1-enterprise
                    # restart: always
                    depends_on:
                      - db
                    environment:
                      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
                      SONAR_JDBC_USERNAME: sonar
                      SONAR_JDBC_PASSWORD: ${SonarQubePassword}
                    volumes:
                      - /apps/sonarqube/conf:/opt/sonarqube/conf
                      - /apps/sonarqube/data:/opt/sonarqube/data
                      - /apps/sonarqube/extensions:/opt/sonarqube/extensions
                      - /apps/sonarqube/logs:/opt/sonarqube/logs
                    ports:
                      - "9000:9000"
                  db:
                    container_name: sonar-postgres
                    image: postgres:12
                    # restart: always
                    environment:
                      POSTGRES_USER: sonar
                      POSTGRES_PASSWORD: ${SonarQubePassword}
                    ports:
                      - "5432:5432"
                    volumes:
                      - /apps/postgres/psql:/var/lib/postgresql
                      - /apps/postgres/data:/var/lib/postgresql/data
        cfn-hup:
          files:
            /etc/cfn/cfn-hup.conf:
              content: !Sub |
                [main]
                stack=${AWS::StackId}
                region=${AWS::Region}
                interval=1
              mode: 000400
              owner: root
              group: root
            # https://github.com/villasv/aws-airflow-stack/issues/72
            # https://stackoverflow.com/questions/39760158/ubuntu-could-not-enable-service-cfn-hup
            # https://www.atcomputing.nl/blog/aws-cloudformation-updating-ec2-instance-with-cfn-hup/
            # https://cfn101.workshop.aws/30-workshop-part-01/30-launching-ec2/400-lab-09-helper-scripts.html
            # https://gist.github.com/kixorz/10194688
            # https://gist.github.com/mmasko/66d34b651642525c63cd39251e0c2a8b
            /etc/cfn/hooks.d/cfn-auto-reloader.conf:
              content: !Sub |
                [cfn-auto-reloader-hook]
                triggers=post.update
                path=Resources.EC2Instance1.Metadata.AWS::CloudFormation::Init
                action=/opt/aws/bin/cfn-init --stack ${AWS::StackName} --resource EC2Instance1 --region ${AWS::Region}
                runas=root
            #systemd service 
            /etc/systemd/system/cfn-hup.service: 
              content: |
                [Unit]
                Description=CloudFormation helper daemon

                [Service]
                ExecStart=/opt/aws/bin/cfn-hup
                Restart=always
                Type=simple

                [Install]
                WantedBy=multi-user.target
              mode: 000400
              owner: root
              group: root

          commands:
            01_update_rc:
              command: "update-rc.d cfn-hup defaults" 
        
          sysvinit:
            docker:
              enabled: true
              ensureRunning: true
            cfn-hup:
              enabled: true
              ensureRunning: true
              files:
                - /etc/cfn/cfn-hup.conf
                - /etc/cfn/hooks.d/cfn-auto-reloader.conf

          commands:
            02_start_cfn-hup:
              command: "systemctl start cfn-hup.service"
            03_mkdir_apps:
              command: mkdir /apps
            04_format_apps_vol:
              #command: mkfs.ext4 /dev/xvdf
              command: mkfs.ext4 /dev/nvme1n1
            05_mount_apps_vol:
              #command: mount /dev/xvdf /apps  # t3mirco
              command: mount /dev/nvme1n1 /apps  # m3
            # 06_docker_compose_check:
            #   command: docker-compose --version >/dev/null 2>&1; (( $? != 0 ))
            06_vm_max_map_count:
              command: sysctl -w vm.max_map_count=262144
            07_docker_compose_up:
              command: cd /home/root && docker-compose up -d

    # ImageId: !Ref 'LatestAmiId' # https://octopus.com/blog/aws-cloudformation-ec2-examples
    # CIS Ubuntu Linux 20.04 LTS Benchmark - Level 1 - ami-0de8ae0e6d86b93c8
    # https://aws.amazon.com/marketplace/server/configuration?productId=96863db2-ecac-4ad4-8d50-e63c424d6245&ref_=psb_cfg_continue
    #
    # Ubuntu Pro 20.04 LTS - ami-0ed28656d62ce20d6
    # https://aws.amazon.com/marketplace/server/configuration?productId=ae7ed378-8838-4fcf-842d-d1d09b34f116&ref_=psb_cfg_continue
    # DisableApiTermination: true
    Properties:
      ImageId: !If [CreateEC2_Ubuntu20.04LTS_CIS, ami-0de8ae0e6d86b93c8, !If [CreateEC2_Ubuntu20.04LTS, ami-0ed28656d62ce20d6, ami-0ed28656d62ce20d6] ]  

      # InstanceType: !Ref 'InstanceType'
      InstanceType: !If [CreateProdResources, m5.large,  !If [CreateDevResources, t2.micro, t3.large] ]  

      KeyName: !Ref 'KeyName'
      IamInstanceProfile: !Ref 'EC2Iam'
      # Monitoring: true
      # Monitoring: false 
      # SubnetId: subnet-0d5faa025dde79b24
      SubnetId: !If [CreateProd1bResources, !Ref PrivateSubnet1b, !Ref PrivateSubnet1a ]

      SecurityGroupIds:
        - !Ref EC2SecurityGroup
      # https://aws.amazon.com/blogs/infrastructure-and-automation/amazon-s3-authenticated-bootstrapping-in-aws-cloudformation/
      # https://linuxtut.com/en/8ff718cd1e66778f88b5/
      # Check /var/log/cloud-init-output.log to troubleshoot UserData 

      UserData:
        'Fn::Base64':
          !Sub |
            #!/bin/bash -xe
            echo start to run userdata... > /var/log/userdata.log            
            apt-get update
            apt -y install python3-pip wget git curl
            python3 -m easy_install --script-dir /opt/aws/bin https://s3.amazonaws.com/cloudformation-examples/aws-cfn-bootstrap-py3-latest.tar.gz
            chmod +x /opt/aws/bin/cfn-init
            echo start running cfn-init >> /var/log/userdata.log

            # on Ubuntu server it expects it in /etc/systemd/system, /etc/init.d/ is for legacy files.
            # ln -s /opt/aws/bin/cfn-hup /etc/init.d/cfn-hup
            ln -s /usr/local/lib/python3.8/dist-packages/aws_cfn_bootstrap-2.0-py3.8.egg/init/ubuntu/cfn-hup /etc/init.d/cfn-hup
            # Call cfn-init script to install files and packages
            /opt/aws/bin/cfn-init -v --stack ${AWS::StackName} --resource EC2Instance1 --configsets full_install --region ${AWS::Region} || error_exit 'Failed to run cfn-init\n'
            echo start running cfn-signal >> /var/log/userdata.log
            # Call cfn-signal script to send a signal with exit code
            /opt/aws/bin/cfn-signal --exit-code $? --stack ${AWS::StackName} --resource EC2Instance1 --region ${AWS::Region}

      # To list volume, lsblk
      # attached volume, https://docs.aws.amazon.com/cli/latest/reference/ec2/attach-volume.html
      # aws ec2 attach-volume --volume-id vol-01ce5d247ed3a0524 --instance-id i-03b8e13dd609897bd --device /dev/xvdf
      # https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/device_naming.html
      # https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/nvme-ebs-volumes.html
      # https://github.com/transferwise/ansible-ebs-automatic-nvme-mapping
      # https://stackoverflow.com/questions/41073906/how-to-attach-and-mount-volumes-to-an-ec2-instance-using-cloudformation
      # https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html
      # https://hangarau.space/automatically-mounting-an-ebs-volume-using-ansible/
      # lsblk -f / blkid
      # mkdir /apps
      # mkfs.ext4 /dev/xvdf, mkfs.ext4 /dev/nvme1n1    
      # mount /dev/xvdf /apps, mount /dev/nvme1n1 /apps
      # mount /dev/sdf /apps
      BlockDeviceMappings:
        - DeviceName: "/dev/sda1"
          Ebs:
            VolumeSize: 20
            VolumeType: gp3
            Encrypted: true
        - DeviceName: "/dev/sdf"
          Ebs:
            VolumeSize: 8
            VolumeType: gp3
            Encrypted: true
            DeleteOnTermination: true
            # DeleteOnTermination: false
            # AutoEnableIO: true
            # MultiAttachEnabled: false

      Tags:
        - Key: Name
          Value: !Join ['-', [ !Ref 'AWS::StackName', 'cfn' ] ]
          # Value: !Ref 'AWS::StackName'
        - Key: Env
          Value: !Ref 'Env'
        - Key: OwnerContact
          Value: "nydevops@smbc-cm.com"

      # CreationPolicy:
      #   ResourceSignal:
      #     Timeout: PT7M

Outputs:
  Env:
    Description: EC2 instance Environment
    Value: !Ref 'Env'
  InstanceId1:
    Description: InstanceId of the newly created EC2 instance
    Value: !Ref 'EC2Instance1'
  SecurityGroupId:
    Description: Group ID of the newly created EC2 security group
    Value: EC2SecurityGroup
  AZ1:
    Description: Availability Zone of the newly created EC2 instance
    Value: !GetAtt [EC2Instance1, AvailabilityZone]
  PrivateDnsName1:
    Description: Private DNSName of the newly created EC2 instance
    Value: !GetAtt [EC2Instance1, PrivateDnsName]
  PrivateIp1:
    Description: Private IP address of the newly created EC2 instance
    Value: !GetAtt [EC2Instance1, PrivateIp]
  # PublicDNS:
  #   Description: Public DNSName of the newly created EC2 instance
  #   Value: !GetAtt [EC2Instance1, PublicDnsName]
  # PublicIP:
  #   Description: Public IP address of the newly created EC2 instance
  #   Value: !GetAtt [EC2Instance1, PublicIp]
