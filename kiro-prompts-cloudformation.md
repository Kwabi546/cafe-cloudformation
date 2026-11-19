# Kiro prompts: CloudFormation assignment (agent-implemented run)

Region us-east-1. Run in Kiro CLI from the folder that holds lab-network.yaml and
lab-application.yaml. Kiro must print each command before running it.
Agent-built resources use the "-agent" suffix: stacks lab-network-agent and
lab-application-agent, tag Project=cafe-cfn-assignment-agent.

## Before starting Kiro (in your terminal)

    cd <repo folder with the two yaml files>
    export CAFE_DB_PASSWORD='YourPasswordHere'   # 8-41 chars; letters, digits and # % ^ * _ + = . , -
    kiro-cli chat

## Prompt 1: Remove the manual (console-built) stacks

Using your shell tool with the AWS CLI in us-east-1, do ONLY this, in order, waiting for each step:
1. Delete the CloudFormation stack lab-application and wait for stack-delete-complete.
2. Force-delete these seven secrets with --force-delete-without-recovery (the stack only schedules them for deletion): /cafe/showServerInfo, /cafe/timeZone, /cafe/currency, /cafe/dbUrl, /cafe/dbName, /cafe/dbUser, /cafe/dbPassword.
3. Delete the stack lab-network and wait for stack-delete-complete.
4. Print aws cloudformation list-stacks for ACTIVE statuses and list-secrets for names starting with /cafe/. Both should be empty.
Do not touch the S3 bucket cafe-cfn-files-<my account id>, cafe-app-role, key pairs, the default VPC or any other resource. Show me each command first. Stop if any step fails.

## Prompt 2: Validate and deploy the network stack

Using your shell tool with the AWS CLI in us-east-1, do ONLY this:
1. Run aws cloudformation validate-template on lab-network.yaml and on lab-application.yaml. Print both results. Stop if either fails.
2. Create the stack lab-network-agent from lab-network.yaml with parameter ProjectName=cafe-cfn-assignment-agent and --tags Key=Project,Value=cafe-cfn-assignment-agent. Use defaults for the other parameters.
3. Wait with aws cloudformation wait stack-create-complete. If it fails, print only the CREATE_FAILED events and stop.
4. Print the stack status and its Outputs as a table.
Show me each command before you run it.

## Prompt 3: Deploy the application stack

Using your shell tool with the AWS CLI in us-east-1, do ONLY this:
1. Check that no secret whose name starts with /cafe/ exists (list-secrets --include-planned-deletion). If one exists, stop and tell me.
2. Find my workstation public IP with: curl -s https://checkip.amazonaws.com
3. Create the stack lab-application-agent from lab-application.yaml with --tags Key=Project,Value=cafe-cfn-assignment-agent and these parameters:
   NetworkStackName=lab-network-agent, ProjectName=cafe-cfn-assignment-agent,
   WorkstationIp=<the IP from step 2>, AsgMinSize=2, AsgMaxSize=4, ShowServerInfo=true,
   AppBucketName=cafe-cfn-files-<my account id>, AppBucketPrefix=cafe-app,
   DBPassword=taken from the environment variable CAFE_DB_PASSWORD.
   Never print the password or echo its value in any command or message; reference the variable only.
4. Wait with aws cloudformation wait stack-create-complete (about 20 to 30 minutes because of Multi-AZ RDS). If it fails, print only the CREATE_FAILED events and stop.
5. Print the stack status and its Outputs as a table.
Show me each command before you run it.

## Prompt 4 (after you verify and submit): cleanup

Using your shell tool with the AWS CLI in us-east-1, delete everything this assignment created, in order, waiting for each step:
1. Delete the stack lab-application-agent and wait for stack-delete-complete.
2. Force-delete the seven /cafe/* secrets with --force-delete-without-recovery.
3. Delete the stack lab-network-agent and wait for stack-delete-complete.
4. Empty and delete the bucket cafe-cfn-files-<my account id>.
5. Print what remains with Project=cafe-cfn-assignment-agent: stacks, VPCs, NAT gateways, Elastic IPs, load balancers, RDS instances, secrets starting with /cafe/, and the bucket. Everything should be empty.
Do not touch cafe-app-role, key pairs, the default VPC or any other resource. Show me each command first.
