# aws-delete-securitygroups
### AWS Lambda function to delete orphaned SecurityGroups created by the ec2 launch wizard 

### The Lambda gets triggered from a cloudwatch event rule (scheduled) 

#### Python Code

```python
import boto3
def lambda_handler(event, context):
    secgroups = {}
    ec2 = boto3.client('ec2')
    all_instances = ec2.describe_instances()
    all_sg = ec2.describe_security_groups()
    for sg in all_sg['SecurityGroups']:
        if sg['GroupName'].startswith('launch-wizard'):
            secgroups[sg['GroupId']] = True
    for reservation in all_instances['Reservations']:
        for instance in reservation['Instances']:
            for sg in instance['SecurityGroups']:
                if sg['GroupId'] in secgroups:
                    del secgroups[sg['GroupId']]
    for g in secgroups:
        ec2.delete_security_group(GroupId=g)
        print('deleted the security-groups: '+ ' '.join(secgroups))
    return 'done'
```

