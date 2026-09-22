# aws-cli cheatsheet

AWS CLI v2. `--profile <p>` picks a named profile; `--region <r>` overrides.

## Identity / config

```bash
aws sts get-caller-identity            # who am I right now
aws configure list-profiles
aws configure --profile work
aws configure sso --profile work       # SSO login flow
aws sso login --profile work
aws configure get region
aws configure list
```

## S3

```bash
aws s3 ls
aws s3 ls s3://bucket/prefix/
aws s3 ls s3://bucket/ --recursive --human-readable --summarize
aws s3 cp file.txt s3://bucket/
aws s3 cp s3://bucket/file.txt ./ --region us-east-1
aws s3 cp dir/ s3://bucket/dir/ --recursive
aws s3 sync ./build s3://bucket/site --delete     # --delete is destructive
aws s3 sync s3://bucket/ ./local
aws s3 rm s3://bucket/file.txt
aws s3 rm s3://bucket/prefix/ --recursive          # destructive
aws s3 mb s3://new-bucket
aws s3 presign s3://bucket/file.txt --expires-in 3600
aws s3api head-object --bucket b --key k
aws s3api list-buckets --query 'Buckets[].Name' --output text
```

## EC2

```bash
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].{id:InstanceId,ip:PrivateIpAddress,state:State.Name}' \
  --output table
aws ec2 describe-instances --filters Name=instance-state-name,Values=running
aws ec2 start-instances --instance-ids i-123
aws ec2 stop-instances --instance-ids i-123
aws ec2 reboot-instances --instance-ids i-123
aws ec2 describe-security-groups --group-ids sg-123
aws ec2 describe-vpcs
aws ec2 describe-subnets
aws ec2 describe-volumes --filters Name=status,Values=available
```

## IAM

```bash
aws iam list-users
aws iam list-roles --query 'Roles[].RoleName' --output text
aws iam get-user
aws iam list-attached-user-policies --user-name ann
aws iam list-access-keys --user-name ann
aws iam get-account-summary
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123:user/ann \
  --action-names s3:GetObject --resource-arns arn:aws:s3:::bucket/*
```

## CloudWatch Logs

```bash
aws logs describe-log-groups
aws logs describe-log-streams --log-group-name /aws/lambda/fn
aws logs get-log-events --log-group-name /aws/lambda/fn \
  --log-stream-name '2026/09/22/[$LATEST]abc' --limit 50
aws logs tail /aws/lambda/fn --follow
aws logs tail /aws/lambda/fn --since 30m
aws logs filter-log-events --log-group-name /aws/lambda/fn \
  --filter-pattern 'ERROR'
```

## ECS / EKS

```bash
aws ecs list-clusters
aws ecs list-services --cluster mycluster
aws ecs describe-services --cluster c --services s
aws ecs update-service --cluster c --service s --force-new-deployment
aws eks list-clusters
aws eks update-kubeconfig --name mycluster --region us-east-1
```

## Lambda

```bash
aws lambda list-functions --query 'Functions[].FunctionName'
aws lambda invoke --function-name fn out.json
aws lambda invoke --function-name fn --payload '{"a":1}' out.json
aws lambda update-function-code --function-name fn --zip-file fileb://fn.zip
aws lambda get-function-configuration --function-name fn
```

## STS / assume role

```bash
aws sts assume-role \
  --role-arn arn:aws:iam::123:role/deploy \
  --role-session-name cli
aws sts get-session-token --duration-seconds 3600
```

## Output / querying (JMESPath)

```bash
aws ec2 describe-instances --output table
aws ec2 describe-instances --output text
aws s3api list-buckets --query 'Buckets[?contains(Name,`prod`)].Name'
aws ec2 describe-instances \
  --query 'Reservations[].Instances[?State.Name==`running`].InstanceId' \
  --output text
aws --cli-auto-prompt                # interactive builder (v2)
```

## Common flags

```bash
--region us-west-2
--profile work
--output json|yaml|table|text
--no-cli-pager                       # don't page output (v2)
--debug                              # full HTTP trace, when nothing else works
--dry-run                            # some commands support it (ec2)
```

## Notes

- `--query` is JMESPath. Backticks are literals, single quotes are strings.
- `aws s3 sync --delete` removes objects at the destination that aren't in the
  source. Read twice.
- If a call hangs, it's usually the wrong region or a missing `--profile`.
- `aws sts get-caller-identity` first, always. Half of "access denied" is being
  the wrong identity.
