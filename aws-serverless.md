[🏠](README.md) › [AWS.js](aws.md) › Serverless

# serverless

We currently use serverless as it is a lightweight abstraction with little investment (minimal dsl) that handles the heavy lifting of:  

1. Developing cloudformation templates (provisioning AWS resources in code).  
2. Deploying to AWS.  

This may change if we hit a ceiling (finer grained permissions), or if Amazon Flourish is a superior offering when it is released.  


## resources

A typical Serverless "service" will touch the following AWS services.  
- IAM roles  
- S3  
- lambda  
**Optional (events)**  
- API Gateway
- S3
- ...


### gotchas  

**Can't change path variable in endpoint**  
> "An error occurred while provisioning your stack: ApiGatewayResourceApigName.
A sibling ({id}) of this resource already has a variable path part -- only one is allowed."

Manually delete the endpoint in AWS API Gateway.

**Bucket doesn't exist error**  
> "The specified bucket does not exist"

If you manually remove resources in AWS, ensure you also remove the corresponding cloudformation stacks uploaded by serverless.
