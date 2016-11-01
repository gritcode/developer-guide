[🏠](README.md) › AWS

# AWS

- [DynamoDB](aws-dynamodb.md)
- [Serverless](aws-serverless.md)


## naming convention

Type | Name
--- | ---
Repository | `import-read`
S3 | `import-read-production`
Lambda | `import-read-production-*`
SQS | `import-read-production-basic`
Dynamodb | `storage-production-basic`

## cli

**View all named profiles**  
`cat ~/.aws/credentials`


## Account setup

### sandbox account
**Default region**  
Oregon is the cheapest + closest region.  
`us-west-2`

```javascript
{
  "Statement": [{
    "Sid": "Stmt1375943389569",
    "Action": "ec2:*",
    "Effect": "Allow",
    "Resource": "*",
    "Condition": {
      "StringEquals": {
        "ec2:Region": "us-west-2"
      }
    }
  }]
}
```

**resources**  
- [How to Separate Your AWS Production and Development Accounts](https://blog.codeship.com/separate-aws-production-and-development-accounts)
