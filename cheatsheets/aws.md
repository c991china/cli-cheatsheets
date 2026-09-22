# AWS CLI 速查

```bash
# 配置
aws configure

# S3 列桶 / 上传 / 下载
aws s3 ls
aws s3 cp ./file s3://bucket/path/
aws s3 sync ./dir s3://bucket/dir/

# EC2 查实例
aws ec2 describe-instances --query "Reservations[].Instances[].InstanceId"

# IAM 列用户
aws iam list-users
```
