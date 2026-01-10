# Module 7: AWS Cloud Fundamentals

## AWS Core Services

### Compute Services

**EC2 (Elastic Compute Cloud)**

Virtual servers in the cloud. Launch instances with various instance types optimized for different workloads. Pay per hour or per second. Key components: AMI (Amazon Machine Image), Instance Types, Security Groups, Key Pairs, EBS volumes.

```bash
# Launch EC2 instance using AWS CLI
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name my-key-pair \
  --security-group-ids sg-12345678 \
  --subnet-id subnet-12345678 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyInstance}]'

# List instances
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,Tags[?Key==`Name`].Value]' --output table

# Stop instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Terminate instance
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0
```

```javascript
// Launch EC2 instance using AWS SDK for Node.js
const AWS = require('aws-sdk');
const ec2 = new AWS.EC2({ region: 'us-east-1' });

async function launchInstance() {
  const params = {
    ImageId: 'ami-0c55b159cbfafe1f0',
    InstanceType: 't2.micro',
    KeyName: 'my-key-pair',
    SecurityGroupIds: ['sg-12345678'],
    SubnetId: 'subnet-12345678',
    MinCount: 1,
    MaxCount: 1,
    TagSpecifications: [{
      ResourceType: 'instance',
      Tags: [{ Key: 'Name', Value: 'MyInstance' }]
    }]
  };

  try {
    const result = await ec2.runInstances(params).promise();
    console.log('Instance launched:', result.Instances[0].InstanceId);
    return result.Instances[0].InstanceId;
  } catch (error) {
    console.error('Error launching instance:', error);
    throw error;
  }
}

// Get instance status
async function getInstanceStatus(instanceId) {
  const params = { InstanceIds: [instanceId] };
  const result = await ec2.describeInstances(params).promise();
  const instance = result.Reservations[0].Instances[0];
  return {
    id: instance.InstanceId,
    state: instance.State.Name,
    publicIp: instance.PublicIpAddress,
    privateIp: instance.PrivateIpAddress
  };
}
```

**EC2 Instance Types**

| Category | Use Case | Examples | vCPU | Memory |
|----------|-----------|-----------|-------|---------|
| General Purpose | Web servers, dev environments | t3.micro, t3.medium, t3.large | 1-8 | 1-32 GB |
| Compute Optimized | High-performance computing, batch processing | c5.large, c5.xlarge, c5.2xlarge | 2-72 | 4-144 GB |
| Memory Optimized | Databases, caching, in-memory analytics | r5.large, r5.xlarge, r5.2xlarge | 2-96 | 16-768 GB |
| Storage Optimized | Data warehouses, file systems | i3.large, i3.xlarge, i3.2xlarge | 2-64 | 15.25-512 GB |
| GPU Instances | Machine learning, graphics rendering | p3.2xlarge, p3.8xlarge, p3.16xlarge | 8-64 | 61-488 GB |

**Lambda (Serverless Compute)**

Run code without provisioning servers. Pay for compute time (milliseconds). Supports Node.js, Python, Java, Go, Ruby, and custom runtimes. Integrates with 200+ AWS services.

```javascript
// Lambda function handler
exports.handler = async (event) => {
  console.log('Event:', JSON.stringify(event, null, 2));
  
  // Process event
  const name = event.queryStringParameters?.name || 'World';
  
  const response = {
    statusCode: 200,
    headers: {
      'Content-Type': 'application/json',
      'Access-Control-Allow-Origin': '*'
    },
    body: JSON.stringify({
      message: `Hello, ${name}!`,
      timestamp: new Date().toISOString()
    })
  };
  
  return response;
};

// Lambda with environment variables
exports.handler = async (event) => {
  const DB_HOST = process.env.DB_HOST;
  const DB_NAME = process.env.DB_NAME;
  
  // Connect to database using env vars
  const connection = await connectToDatabase(DB_HOST, DB_NAME);
  
  const result = await connection.query('SELECT * FROM users');
  
  return {
    statusCode: 200,
    body: JSON.stringify(result)
  };
};
```

```bash
# Deploy Lambda function using AWS CLI
aws lambda create-function \
  --function-name my-function \
  --runtime nodejs18.x \
  --role arn:aws:iam::123456789012:role/lambda-role \
  --handler index.handler \
  --zip-file fileb://function.zip \
  --environment Variables={DB_HOST=localhost,DB_NAME=mydb}

# Invoke Lambda function
aws lambda invoke \
  --function-name my-function \
  --payload '{"queryStringParameters": {"name": "AWS"}}' \
  response.json

# Update function code
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://new-function.zip

# Add environment variables
aws lambda update-function-configuration \
  --function-name my-function \
  --environment Variables={API_KEY=secret123,DEBUG=true}
```

**Elastic Beanstalk**

Platform as a service for deploying applications. Handles provisioning, load balancing, auto-scaling, and health monitoring. Supports multiple languages and frameworks.

```bash
# Initialize Elastic Beanstalk application
eb init my-app \
  --platform node.js \
  --region us-east-1

# Create environment
eb create production-env

# Deploy application
eb deploy

# Set environment variables
eb setenv NODE_ENV=production DB_HOST=production.db.com

# View logs
eb logs --all

# Scale environment
eb scale 4
```

---

### Storage Services

**S3 (Simple Storage Service)**

Object storage with unlimited scalability. Store any amount of data. Features: versioning, lifecycle policies, encryption, static website hosting, event notifications.

```bash
# Create S3 bucket
aws s3 mb s3://my-unique-bucket-name

# Upload file to S3
aws s3 cp local-file.txt s3://my-unique-bucket-name/

# Upload with metadata
aws s3 cp local-file.txt s3://my-bucket/ \
  --metadata "author=John,category=documents"

# List objects in bucket
aws s3 ls s3://my-unique-bucket-name/

# Download file from S3
aws s3 cp s3://my-unique-bucket-name/file.txt ./downloaded-file.txt

# Sync directory to S3
aws s3 sync ./local-dir s3://my-bucket/remote-dir --delete

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled

# Set lifecycle policy
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-bucket \
  --lifecycle-configuration file://lifecycle.json
```

```javascript
// S3 operations using AWS SDK
const AWS = require('aws-sdk');
const s3 = new AWS.S3({ region: 'us-east-1' });

// Upload file to S3
async function uploadFile(bucketName, key, filePath) {
  const fs = require('fs');
  const fileContent = fs.readFileSync(filePath);
  
  const params = {
    Bucket: bucketName,
    Key: key,
    Body: fileContent,
    ContentType: 'text/plain',
    Metadata: {
      'author': 'John',
      'upload-date': new Date().toISOString()
    }
  };
  
  try {
    const result = await s3.upload(params).promise();
    console.log('File uploaded:', result.Location);
    return result;
  } catch (error) {
    console.error('Upload error:', error);
    throw error;
  }
}

// Download file from S3
async function downloadFile(bucketName, key, filePath) {
  const params = {
    Bucket: bucketName,
    Key: key
  };
  
  try {
    const result = await s3.getObject(params).promise();
    const fs = require('fs');
    fs.writeFileSync(filePath, result.Body);
    console.log('File downloaded to:', filePath);
    return result;
  } catch (error) {
    console.error('Download error:', error);
    throw error;
  }
}

// List objects in bucket
async function listObjects(bucketName) {
  const params = { Bucket: bucketName };
  
  try {
    const result = await s3.listObjectsV2(params).promise();
    return result.Contents.map(obj => ({
      key: obj.Key,
      size: obj.Size,
      lastModified: obj.LastModified
    }));
  } catch (error) {
    console.error('List error:', error);
    throw error;
  }
}

// Generate presigned URL (for temporary access)
async function getPresignedUrl(bucketName, key, expiresIn = 3600) {
  const params = {
    Bucket: bucketName,
    Key: key,
    Expires: expiresIn
  };
  
  return s3.getSignedUrlPromise('getObject', params);
}
```

**S3 Storage Classes**

| Class | Use Case | Durability | Availability | Min Storage | Retrieval Time |
|-------|----------|------------|---------------|--------------|----------------|
| Standard | Frequently accessed data | 99.999999999% | 99.99% | None | Immediate |
| Intelligent-Tiering | Unknown access patterns | 99.999999999% | 99.9% | None | Immediate |
| Standard-IA | Infrequently accessed data | 99.999999999% | 99.9% | 30 days | Immediate |
| One Zone-IA | Infrequently accessed, resiliency not needed | 99.999999999% | 99.5% | 30 days | Immediate |
| Glacier | Data archiving | 99.999999999% | 99.99% | 90 days | Minutes (expedited) to Hours (standard) |
| Glacier Deep Archive | Long-term retention | 99.999999999% | 99.99% | 180 days | Hours (expedited) to 12 hours (standard) |

**EBS (Elastic Block Store)**

Block storage for EC2 instances. Persistent storage that persists independently of instance life. Types: gp3 (general purpose SSD), io2 (provisioned IOPS SSD), st1 (throughput optimized HDD), sc1 (cold HDD).

```bash
# Create EBS volume
aws ec2 create-volume \
  --size 20 \
  --volume-type gp3 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=MyVolume}]'

# Attach volume to instance
aws ec2 attach-volume \
  --volume-id vol-1234567890abcdef0 \
  --instance-id i-1234567890abcdef0 \
  --device /dev/sdf

# Detach volume
aws ec2 detach-volume --volume-id vol-1234567890abcdef0

# Create snapshot
aws ec2 create-snapshot \
  --volume-id vol-1234567890abcdef0 \
  --description "Backup before update"

# Delete volume
aws ec2 delete-volume --volume-id vol-1234567890abcdef0
```

**EFS (Elastic File System)**

Network file system for EC2 and Lambda. Scalable, elastic, pay-as-you-go. Supports NFS protocol. Mount on multiple instances simultaneously.

```bash
# Create EFS file system
aws efs create-file-system \
  --creation-token my-fs \
  --tags Key=Name,Value=MyFileSystem

# Create mount target
aws efs create-mount-target \
  --file-system-id fs-12345678 \
  --subnet-id subnet-12345678 \
  --security-group-ids sg-12345678

# Mount EFS on EC2 (Linux)
sudo mkdir /mnt/efs
sudo mount -t efs fs-12345678:/ /mnt/efs

# Add to /etc/fstab for auto-mount
fs-12345678:/ /mnt/efs efs defaults,_netdev 0 0
```

---

### Database Services

**RDS (Relational Database Service)**

Managed relational databases: MySQL, PostgreSQL, Oracle, SQL Server, MariaDB, Aurora. Automated backups, patching, scaling, and high availability.

```bash
# Create RDS instance
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --master-username admin \
  --master-user-password MySecurePassword123! \
  --allocated-storage 20 \
  --vpc-security-group-ids sg-12345678 \
  --db-subnet-group-name my-db-subnet-group

# Create read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier mydb-replica \
  --source-db-instance-identifier mydb

# List databases
aws rds describe-db-instances \
  --query 'DBInstances[*].[DBInstanceIdentifier,DBInstanceStatus,Engine]'

# Create snapshot
aws rds create-db-snapshot \
  --db-instance-identifier mydb \
  --db-snapshot-identifier mydb-snapshot-$(date +%Y%m%d)

# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier mydb-restored \
  --db-snapshot-identifier mydb-snapshot-20240101
```

```javascript
// Connect to RDS from Node.js
const mysql = require('mysql2/promise');

async function connectToRDS() {
  const connection = await mysql.createConnection({
    host: process.env.DB_HOST,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    ssl: {
      rejectUnauthorized: true
    }
  });
  
  console.log('Connected to RDS');
  return connection;
}

async function queryDatabase(connection, sql, params = []) {
  try {
    const [rows] = await connection.execute(sql, params);
    return rows;
  } catch (error) {
    console.error('Query error:', error);
    throw error;
  }
}

// Usage
async function main() {
  const connection = await connectToRDS();
  
  // Create table
  await queryDatabase(connection, `
    CREATE TABLE IF NOT EXISTS users (
      id INT AUTO_INCREMENT PRIMARY KEY,
      name VARCHAR(100),
      email VARCHAR(100),
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )
  `);
  
  // Insert user
  await queryDatabase(connection, `
    INSERT INTO users (name, email) VALUES (?, ?)
  `, ['John Doe', 'john@example.com']);
  
  // Query users
  const users = await queryDatabase(connection, 'SELECT * FROM users');
  console.log('Users:', users);
  
  await connection.end();
}
```

**DynamoDB**

Managed NoSQL database. Key-value and document data model. Single-digit millisecond latency at any scale. Features: auto-scaling, global tables, TTL, streams, ACID transactions.

```bash
# Create DynamoDB table
aws dynamodb create-table \
  --table-name Users \
  --attribute-definitions AttributeName=id,AttributeType=S \
  --key-schema AttributeName=id,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --tags Key=Environment,Value=Production

# Put item
aws dynamodb put-item \
  --table-name Users \
  --item '{"id":{"S":"user1"},"name":{"S":"John"},"age":{"N":"30"}}'

# Get item
aws dynamodb get-item \
  --table-name Users \
  --key '{"id":{"S":"user1"}}'

# Query items
aws dynamodb query \
  --table-name Users \
  --key-condition-expression "id = :id" \
  --expression-attribute-values '{":id":{"S":"user1"}}'

# Update item
aws dynamodb update-item \
  --table-name Users \
  --key '{"id":{"S":"user1"}}' \
  --update-expression "SET #a = :a" \
  --expression-attribute-names '{"#a":"age"}' \
  --expression-attribute-values '{":a":{"N":"31"}}'

# Delete item
aws dynamodb delete-item \
  --table-name Users \
  --key '{"id":{"S":"user1"}}'
```

```javascript
// DynamoDB operations using AWS SDK
const AWS = require('aws-sdk');
const dynamoDB = new AWS.DynamoDB.DocumentClient({ region: 'us-east-1' });

// Put item
async function putItem(tableName, item) {
  const params = {
    TableName: tableName,
    Item: item
  };
  
  try {
    await dynamoDB.put(params).promise();
    console.log('Item added:', item);
  } catch (error) {
    console.error('Put error:', error);
    throw error;
  }
}

// Get item
async function getItem(tableName, key) {
  const params = {
    TableName: tableName,
    Key: key
  };
  
  try {
    const result = await dynamoDB.get(params).promise();
    return result.Item;
  } catch (error) {
    console.error('Get error:', error);
    throw error;
  }
}

// Query items
async function queryItems(tableName, keyCondition, expressionValues) {
  const params = {
    TableName: tableName,
    KeyConditionExpression: keyCondition,
    ExpressionAttributeValues: expressionValues
  };
  
  try {
    const result = await dynamoDB.query(params).promise();
    return result.Items;
  } catch (error) {
    console.error('Query error:', error);
    throw error;
  }
}

// Update item
async function updateItem(tableName, key, updateExpression, expressionAttributeValues) {
  const params = {
    TableName: tableName,
    Key: key,
    UpdateExpression: updateExpression,
    ExpressionAttributeValues: expressionAttributeValues,
    ReturnValues: 'ALL_NEW'
  };
  
  try {
    const result = await dynamoDB.update(params).promise();
    return result.Attributes;
  } catch (error) {
    console.error('Update error:', error);
    throw error;
  }
}

// Batch write
async function batchWrite(tableName, items) {
  const params = {
    RequestItems: {
      [tableName]: items.map(item => ({
        PutRequest: { Item: item }
      }))
    }
  };
  
  try {
    await dynamoDB.batchWrite(params).promise();
    console.log('Batch write completed');
  } catch (error) {
    console.error('Batch write error:', error);
    throw error;
  }
}
```

**DynamoDB vs RDS Comparison**

| Feature | DynamoDB | RDS |
|---------|-----------|-----|
| Data Model | NoSQL (key-value, document) | Relational (SQL) |
| Schema | Flexible schema | Fixed schema |
| Scaling | Automatic horizontal scaling | Vertical scaling, read replicas |
| Latency | Single-digit milliseconds | Variable, depends on instance |
| Consistency | Eventually consistent (default) or strongly consistent | ACID compliant |
| Pricing | Pay per request | Pay per instance hour |
| Use Case | High throughput, simple queries | Complex queries, joins, transactions |

---

### Networking Services

**VPC (Virtual Private Cloud)**

Isolated network environment. Define IP ranges, subnets, route tables, gateways. Control network access with security groups and NACLs.

```
VPC Architecture Diagram:

                    Internet
                       |
                 [Internet Gateway]
                       |
                    [Route Table]
                       |
        +--------------+---------------+
        |              |               |
   [Public Subnet] [Public Subnet] [Public Subnet]
        |              |               |
    [EC2]         [EC2]           [EC2]
        |              |               |
        +--------------+---------------+
                       |
                 [NAT Gateway]
                       |
        +--------------+---------------+
        |              |               |
  [Private Subnet] [Private Subnet] [Private Subnet]
        |              |               |
    [RDS]         [ElastiCache]    [Lambda]
```

```bash
# Create VPC
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=MyVPC}]'

# Create public subnet
aws ec2 create-subnet \
  --vpc-id vpc-12345678 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PublicSubnet}]'

# Create private subnet
aws ec2 create-subnet \
  --vpc-id vpc-12345678 \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PrivateSubnet}]'

# Create internet gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=MyIGW}]'

# Attach internet gateway to VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-12345678 \
  --vpc-id vpc-12345678

# Create route table
aws ec2 create-route-table \
  --vpc-id vpc-12345678 \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=PublicRouteTable}]'

# Add route to internet
aws ec2 create-route \
  --route-table-id rtb-12345678 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-12345678

# Associate route table with subnet
aws ec2 associate-route-table \
  --route-table-id rtb-12345678 \
  --subnet-id subnet-12345678
```

**Security Groups & NACLs**

Security groups: Stateful firewall at instance level. NACLs: Stateless firewall at subnet level.

```bash
# Create security group
aws ec2 create-security-group \
  --group-name my-security-group \
  --description "My security group" \
  --vpc-id vpc-12345678

# Allow inbound HTTP traffic
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow inbound HTTPS traffic
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow inbound SSH from specific IP
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/24

# List security group rules
aws ec2 describe-security-groups \
  --group-ids sg-12345678 \
  --query 'SecurityGroups[0].IpPermissions'

# Create network ACL
aws ec2 create-network-acl \
  --vpc-id vpc-12345678 \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=MyNACL}]'

# Add inbound rule to NACL
aws ec2 create-network-acl-entry \
  --network-acl-id acl-12345678 \
  --rule-number 100 \
  --protocol -1 \
  --rule-action allow \
  --ingress \
  --cidr-block 0.0.0.0/0
```

**Load Balancing**

Distribute traffic across multiple targets. Types: ALB (Application Load Balancer - HTTP/HTTPS), NLB (Network Load Balancer - TCP/UDP), CLB (Classic Load Balancer - legacy).

```bash
# Create Application Load Balancer
aws elbv2 create-load-balancer \
  --name my-alb \
  --subnets subnet-12345678 subnet-87654321 \
  --security-groups sg-12345678 \
  --scheme internet-facing \
  --type application

# Create target group
aws elbv2 create-target-group \
  --name my-targets \
  --protocol HTTP \
  --port 80 \
  --vpc-id vpc-12345678 \
  --target-type instance

# Register targets
aws elbv2 register-targets \
  --target-group-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/my-targets/abc123 \
  --targets Id=i-1234567890abcdef0,Port=80 Id=i-0987654321fedcba0,Port=80

# Create listener
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/my-alb/abc123 \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/my-targets/abc123
```

```javascript
// Configure ALB with Terraform-like structure using AWS SDK
const AWS = require('aws-sdk');
const elbv2 = new AWS.ELBv2({ region: 'us-east-1' });

async function createLoadBalancer() {
  const params = {
    Name: 'my-alb',
    Subnets: ['subnet-12345678', 'subnet-87654321'],
    SecurityGroups: ['sg-12345678'],
    Scheme: 'internet-facing',
    Type: 'application',
    Tags: [
      { Key: 'Environment', Value: 'production' },
      { Key: 'Project', Value: 'web-app' }
    ]
  };
  
  const result = await elbv2.createLoadBalancer(params).promise();
  console.log('Load balancer created:', result.LoadBalancers[0].LoadBalancerArn);
  return result.LoadBalancers[0];
}

async function createTargetGroup(vpcId) {
  const params = {
    Name: 'my-targets',
    Protocol: 'HTTP',
    Port: 80,
    VpcId: vpcId,
    TargetType: 'instance',
    HealthCheckPath: '/health',
    HealthCheckIntervalSeconds: 30,
    HealthCheckTimeoutSeconds: 5,
    HealthyThresholdCount: 3,
    UnhealthyThresholdCount: 2
  };
  
  const result = await elbv2.createTargetGroup(params).promise();
  console.log('Target group created:', result.TargetGroups[0].TargetGroupArn);
  return result.TargetGroups[0];
}
```

---

## AWS Security & IAM

### IAM (Identity and Access Management)

**IAM Users & Groups**

Manage access to AWS resources. Users: individuals, Groups: collections of users, Roles: for services and cross-account access. Principle of least privilege.

```bash
# Create IAM user
aws iam create-user \
  --user-name john.doe \
  --tags Key=Department,Value=Engineering

# Create IAM group
aws iam create-group \
  --group-name developers

# Add user to group
aws iam add-user-to-group \
  --group-name developers \
  --user-name john.doe

# Attach policy to group
aws iam attach-group-policy \
  --group-name developers \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# Create access key for user
aws iam create-access-key \
  --user-name john.doe

# List users in group
aws iam get-group \
  --group-name developers

# List user policies
aws iam list-attached-user-policies \
  --user-name john.doe
```

**IAM Policies**

JSON documents that define permissions. Structure: Version, Statement (Effect, Action, Resource, Condition). Use policy generator for complex policies.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3Read",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    },
    {
      "Sid": "AllowEC2ReadOnly",
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*",
        "ec2:Get*"
      ],
      "Resource": "*"
    },
    {
      "Sid": "DenyProductionAccess",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:ResourceTag/Environment": "production"
        }
      }
    }
  ]
}
```

```bash
# Create custom policy
aws iam create-policy \
  --policy-name S3ReadOnlyPolicy \
  --policy-document file://policy.json

# Attach policy to user
aws iam attach-user-policy \
  --user-name john.doe \
  --policy-arn arn:aws:iam::123456789012:policy/S3ReadOnlyPolicy

# Detach policy from user
aws iam detach-user-policy \
  --user-name john.doe \
  --policy-arn arn:aws:iam::123456789012:policy/S3ReadOnlyPolicy

# Delete policy
aws iam delete-policy \
  --policy-arn arn:aws:iam::123456789012:policy/S3ReadOnlyPolicy
```

**IAM Roles**

Allow AWS services to act on your behalf. Also for cross-account access and federated access (SAML, OIDC).

```bash
# Create IAM role for EC2
aws iam create-role \
  --role-name EC2S3AccessRole \
  --assume-role-policy-document file://trust-policy.json

# trust-policy.json:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

# Attach policy to role
aws iam attach-role-policy \
  --role-name EC2S3AccessRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create instance profile
aws iam create-instance-profile \
  --instance-profile-name EC2S3AccessProfile

# Add role to instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2S3AccessProfile \
  --role-name EC2S3AccessRole

# Associate instance profile with EC2 instance
aws ec2 associate-iam-instance-profile \
  --instance-id i-1234567890abcdef0 \
  --iam-instance-profile Name=EC2S3AccessProfile
```

```javascript
// Assume role using AWS SDK
const AWS = require('aws-sdk');

async function assumeRole(roleArn, sessionName) {
  const sts = new AWS.STS();
  
  const params = {
    RoleArn: roleArn,
    RoleSessionName: sessionName,
    DurationSeconds: 3600
  };
  
  try {
    const result = await sts.assumeRole(params).promise();
    
    // Create new AWS client with assumed role credentials
    const assumedCredentials = {
      accessKeyId: result.Credentials.AccessKeyId,
      secretAccessKey: result.Credentials.SecretAccessKey,
      sessionToken: result.Credentials.SessionToken
    };
    
    const s3 = new AWS.S3({
      credentials: assumedCredentials,
      region: 'us-east-1'
    });
    
    console.log('Assumed role successfully');
    return s3;
  } catch (error) {
    console.error('Error assuming role:', error);
    throw error;
  }
}

// Usage
async function accessS3WithRole() {
  const s3 = await assumeRole(
    'arn:aws:iam::123456789012:role/CrossAccountRole',
    'MySession'
  );
  
  const buckets = await s3.listBuckets().promise();
  console.log('Buckets:', buckets.Buckets);
}
```

**Security Best Practices**

✅ **Secure Practices**
- Use MFA (Multi-Factor Authentication) for root and IAM users
- Rotate access keys regularly (every 90 days)
- Use IAM roles instead of access keys for applications
- Apply least privilege principle
- Enable CloudTrail for logging API calls
- Use AWS Config for compliance monitoring
- Enable encryption at rest and in transit
- Regularly review IAM policies and permissions
- Use IAM Access Analyzer to identify unintended access
- Implement password policies

❌ **Vulnerable Practices**
- Using root account for daily operations
- Sharing access keys in code repositories
- Giving overly permissive policies (*)
- Not rotating credentials
- Disabling CloudTrail logging
- Hardcoding credentials in applications
- Not using MFA
- Ignoring security warnings
- Using outdated IAM policies
- Not reviewing access regularly

---

### KMS (Key Management Service)

**Encryption Keys**

Managed service for creating and controlling encryption keys. Customer Master Keys (CMKs) and Data Keys. Integration with 200+ AWS services.

```bash
# Create KMS key
aws kms create-key \
  --description "My encryption key" \
  --tags Tag=Environment,Value=Production

# Create alias for key
aws kms create-alias \
  --alias-name alias/my-key \
  --target-key-id 1234abcd-12ab-34cd-56ef-1234567890ab

# Encrypt data
aws kms encrypt \
  --key-id alias/my-key \
  --plaintext "My secret data" \
  --output text \
  --query CiphertextBlob | base64 --decode > encrypted.data

# Decrypt data
aws kms decrypt \
  --ciphertext-blob fileb://encrypted.data \
  --output text \
  --query Plaintext | base64 --decode

# Generate data key
aws kms generate-data-key \
  --key-id alias/my-key \
  --key-spec AES_256

# Enable key rotation
aws kms enable-key-rotation \
  --key-id 1234abcd-12ab-34cd-56ef-1234567890ab

# Schedule key deletion (7-30 days waiting period)
aws kms schedule-key-deletion \
  --key-id 1234abcd-12ab-34cd-56ef-1234567890ab \
  --pending-window-in-days 7
```

```javascript
// KMS operations using AWS SDK
const AWS = require('aws-sdk');
const kms = new AWS.KMS({ region: 'us-east-1' });

// Encrypt data
async function encryptData(plaintext, keyId) {
  const params = {
    KeyId: keyId,
    Plaintext: Buffer.from(plaintext)
  };
  
  try {
    const result = await kms.encrypt(params).promise();
    return result.CiphertextBlob;
  } catch (error) {
    console.error('Encryption error:', error);
    throw error;
  }
}

// Decrypt data
async function decryptData(ciphertextBlob) {
  const params = {
    CiphertextBlob: ciphertextBlob
  };
  
  try {
    const result = await kms.decrypt(params).promise();
    return result.Plaintext.toString();
  } catch (error) {
    console.error('Decryption error:', error);
    throw error;
  }
}

// Generate data key
async function generateDataKey(keyId) {
  const params = {
    KeyId: keyId,
    KeySpec: 'AES_256'
  };
  
  try {
    const result = await kms.generateDataKey(params).promise();
    return {
      plaintextKey: result.Plaintext,
      encryptedKey: result.CiphertextBlob
    };
  } catch (error) {
    console.error('Key generation error:', error);
    throw error;
  }
}

// Usage
async function main() {
  const keyId = 'alias/my-key';
  
  // Encrypt
  const encrypted = await encryptData('Secret message', keyId);
  console.log('Encrypted data:', encrypted);
  
  // Decrypt
  const decrypted = await decryptData(encrypted);
  console.log('Decrypted:', decrypted);
  
  // Generate data key for envelope encryption
  const dataKey = await generateDataKey(keyId);
  console.log('Data key generated');
}
```

---

### Security Services

**CloudTrail**

Audit trail of API calls. Logs who made what call, from where, when. Integrates with CloudWatch Logs and S3 for storage and analysis.

```bash
# Create trail
aws cloudtrail create-trail \
  --name my-trail \
  --s3-bucket-name my-cloudtrail-bucket \
  --include-global-service-events \
  --is-multi-region-trail

# Start logging
aws cloudtrail start-logging \
  --name my-trail

# Get trail status
aws cloudtrail get-trail-status \
  --name my-trail

# Create event data store for CloudTrail Lake
aws cloudtrail create-event-data-store \
  --name my-event-data-store \
  --retention-period 90

# Query event data store
aws cloudtrail start-query \
  --event-data-store arn:aws:cloudtrail:us-east-1:123456789012:eventdatastore/abc123 \
  --query-statement 'SELECT * FROM my-event-data-store WHERE eventName = `CreateUser`'
```

**Security Hub**

Centralized security and compliance dashboard. Aggregates findings from GuardDuty, Inspector, Macie, and partner tools.

```bash
# Enable Security Hub
aws securityhub enable-security-hub

# Subscribe to security standards
aws securityhub subscribe-to-aggregate-organization \
  --organization-unit-id ou-1234-56789abc

# Get findings
aws securityhub get-findings \
  --filters '{"SeverityLabel":[{"value":"HIGH","comparison":"EQUALS"}]}'

# Import findings
aws securityhub import-findings \
  --findings-file file://findings.json

# Update finding
aws securityhub batch-update-findings \
  --finding-identifiers '[{"Id":"finding-id","ProductArn":"arn:aws:securityhub:us-east-1::product/aws/guardduty"}]' \
  --note '{"Text":"Investigated - no action required","UpdatedBy":"John Doe"}' \
  --workflow '{"Status":"RESOLVED"}'
```

---

## AWS Deployment Strategies

### CI/CD with CodePipeline

**Pipeline Configuration**

Automated release process: Source → Build → Test → Deploy. Integrates with CodeCommit, CodeBuild, CodeDeploy, and third-party tools.

```yaml
# pipeline.yaml - CloudFormation template for CodePipeline
AWSTemplateFormatVersion: '2010-09-09'
Description: 'CodePipeline for CI/CD'

Resources:
  CodePipelineRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: codepipeline.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AWSCodePipelineFullAccess

  MyPipeline:
    Type: AWS::CodePipeline::Pipeline
    Properties:
      Name: MyDeploymentPipeline
      RoleArn: !GetAtt CodePipelineRole.Arn
      Stages:
        - Name: Source
          Actions:
            - Name: SourceAction
              ActionTypeId:
                Category: Source
                Owner: AWS
                Provider: CodeCommit
                Version: '1'
              Configuration:
                RepositoryName: my-repo
                BranchName: main
              OutputArtifacts:
                - Name: SourceOutput
        - Name: Build
          Actions:
            - Name: BuildAction
              ActionTypeId:
                Category: Build
                Owner: AWS
                Provider: CodeBuild
                Version: '1'
              Configuration:
                ProjectName: MyBuildProject
              InputArtifacts:
                - Name: SourceOutput
              OutputArtifacts:
                - Name: BuildOutput
        - Name: Deploy
          Actions:
            - Name: DeployAction
              ActionTypeId:
                Category: Deploy
                Owner: AWS
                Provider: CodeDeploy
                Version: '1'
              Configuration:
                ApplicationName: MyApplication
                DeploymentGroupName: MyDeploymentGroup
              InputArtifacts:
                - Name: BuildOutput
```

```bash
# Create CodePipeline using CloudFormation
aws cloudformation create-stack \
  --stack-name my-pipeline-stack \
  --template-body file://pipeline.yaml \
  --capabilities CAPABILITY_NAMED_IAM

# Update pipeline
aws codepipeline update-pipeline \
  --cli-input-json file://pipeline-update.json

# Get pipeline state
aws codepipeline get-pipeline-state \
  --name MyDeploymentPipeline

# Start pipeline execution manually
aws codepipeline start-pipeline-execution \
  --name MyDeploymentPipeline
```

---

### CodeBuild

**Build Specifications**

Build environment, phases, and artifacts defined in buildspec.yml. Supports multiple environments: Ubuntu, Windows, Amazon Linux 2, Docker.

```yaml
# buildspec.yml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 18
    commands:
      - npm install -g npm
  pre_build:
    commands:
      - echo Installing dependencies...
      - npm ci
  build:
    commands:
      - echo Running tests...
      - npm test
      - echo Building application...
      - npm run build
  post_build:
    commands:
      - echo Build completed on `date`

artifacts:
  files:
    - '**/*'
  base-directory: build
  discard-paths: yes

cache:
  paths:
    - node_modules/**/*

reports:
  test-reports:
    files:
      - 'coverage/**/*.xml'
    file-format: JUNITXML
```

```bash
# Create CodeBuild project
aws codebuild create-project \
  --name MyBuildProject \
  --source type=CODECOMMIT,location=https://git-codecommit.us-east-1.amazonaws.com/v1/repos/my-repo \
  --artifacts type=NO_ARTIFACTS \
  --environment type=LINUX_CONTAINER,computeType=BUILD_GENERAL1_SMALL,image=aws/codebuild/standard:6.0 \
  --service-role-arn arn:aws:iam::123456789012:role/service-role/CodeBuildServiceRole \
  --buildspec buildspec.yml

# Start build
aws codebuild start-build \
  --project-name MyBuildProject

# List builds
aws codebuild list-builds-for-project \
  --project-name MyBuildProject

# Get build details
aws codebuild batch-get-builds \
  --ids my-build-id
```

---

### CodeDeploy

**Deployment Strategies**

Automated deployments to EC2, Lambda, on-premises, ECS. Strategies: In-place, Blue/Green, Canary, Linear.

```yaml
# appspec.yml for EC2/On-Premises
version: 0.0
os: linux

files:
  - source: /
    destination: /var/www/html

hooks:
  BeforeInstall:
    - location: scripts/before_install.sh
      timeout: 300
      runas: root
  AfterInstall:
    - location: scripts/after_install.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 300
      runas: root
```

```yaml
# appspec.yml for Lambda
version: 0.0
Resources:
  - MyLambdaFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: MyFunction
        Alias: MyLambdaAlias
        CurrentVersion: 1
        TargetVersion: 2
Hooks:
  BeforeAllowTraffic:
    - location: scripts/before_allow_traffic.sh
      timeout: 300
  AfterAllowTraffic:
    - location: scripts/after_allow_traffic.sh
      timeout: 300
```

```bash
# Create CodeDeploy application
aws deploy create-application \
  --application-name MyApplication

# Create deployment group
aws deploy create-deployment-group \
  --application-name MyApplication \
  --deployment-group-name MyDeploymentGroup \
  --deployment-config-name CodeDeployDefault.OneAtATime \
  --service-role-arn arn:aws:iam::123456789012:role/service-role/CodeDeployServiceRole

# Create deployment
aws deploy create-deployment \
  --application-name MyApplication \
  --deployment-group-name MyDeploymentGroup \
  --description "Deploy version 1.0" \
  --s3-location bucket=my-deployment-bucket,bundleType=zip,key=application.zip \
  --deployment-config-name CodeDeployDefault.AllAtOnce

# Get deployment status
aws deploy get-deployment \
  --deployment-id d-123456789

# List deployments
aws deploy list-deployments \
  --application-name MyApplication
```

---

### Elastic Beanstalk Deployment

**Application Deployment**

Platform as a service with built-in load balancing, auto-scaling, and health monitoring. Supports multiple deployment policies.

```bash
# Initialize Elastic Beanstalk application
eb init my-app \
  --platform node.js \
  --region us-east-1

# Create environment
eb create production-env \
  --single \
  --instance-type t3.micro

# Deploy application
eb deploy production-env \
  --message "Deploying version 1.0"

# Set environment variables
eb setenv NODE_ENV=production DB_HOST=production.db.com

# Configure environment
eb config save prod-config

# Scale environment
eb scale 4

# Swap environment CNAMEs (blue/green)
eb swap production-env staging-env
```

```yaml
# .ebextensions/config.config - Custom configuration
option_settings:
  aws:elasticbeanstalk:container:nodejs:
    NodeCommand: "npm start"
    NodeVersion: "18.x"
  aws:elasticbeanstalk:application:environment:
    NODE_ENV: production
    PORT: 8080
  aws:elasticbeanstalk:healthreporting:system:
    SystemType: enhanced
    HealthCheckSuccessThreshold: OK

packages:
  yum:
    git: []
    postgresql96: []

commands:
  01_install_dependencies:
    command: "npm install --production"
    cwd: /var/app/current
```

---

## AWS CLI & SDK

### AWS CLI Configuration

**Installation & Setup**

Command-line interface for AWS services. Supports multiple profiles, environment variables, and configuration files.

```bash
# Install AWS CLI on Linux/Mac
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Verify installation
aws --version

# Configure AWS CLI
aws configure

# Configure named profile
aws configure --profile dev

# Configure with specific values
aws configure set region us-east-1
aws configure set output json

# List profiles
aws configure list-profiles

# View configuration
aws configure list
```

**Configuration Files**

```ini
# ~/.aws/credentials
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

[dev]
aws_access_key_id = AKIAI44QH8DHBEXAMPLE
aws_secret_access_key = je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY

[prod]
aws_access_key_id = AKIAI44QH8DHBEXAMPLE
aws_secret_access_key = je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
```

```ini
# ~/.aws/config
[default]
region = us-west-2
output = json

[profile dev]
region = us-east-1
output = table

[profile prod]
region = eu-west-1
output = json
```

---

### AWS SDK for Node.js

**Installation & Setup**

Official JavaScript SDK for AWS. Supports Node.js and browser. Modular packages for each service.

```bash
# Install AWS SDK v3
npm install @aws-sdk/client-s3 @aws-sdk/client-ec2

# Install AWS SDK v2 (legacy)
npm install aws-sdk
```

```javascript
// AWS SDK v3 - Modern modular approach
const { S3Client, PutObjectCommand, GetObjectCommand } = require('@aws-sdk/client-s3');
const { EC2Client, RunInstancesCommand } = require('@aws-sdk/client-ec2');

// Create S3 client
const s3Client = new S3Client({
  region: 'us-east-1',
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
  }
});

// Upload object
async function uploadObject(bucket, key, body) {
  const command = new PutObjectCommand({
    Bucket: bucket,
    Key: key,
    Body: body,
    ContentType: 'application/json'
  });
  
  try {
    const response = await s3Client.send(command);
    console.log('Upload successful:', response.ETag);
    return response;
  } catch (error) {
    console.error('Upload failed:', error);
    throw error;
  }
}

// Download object
async function downloadObject(bucket, key) {
  const command = new GetObjectCommand({
    Bucket: bucket,
    Key: key
  });
  
  try {
    const response = await s3Client.send(command);
    const body = await response.Body.transformToString();
    return body;
  } catch (error) {
    console.error('Download failed:', error);
    throw error;
  }
}

// Create EC2 client
const ec2Client = new EC2Client({ region: 'us-east-1' });

// Launch EC2 instance
async function launchInstance(imageId, instanceType) {
  const command = new RunInstancesCommand({
    ImageId: imageId,
    InstanceType: instanceType,
    MinCount: 1,
    MaxCount: 1,
    TagSpecifications: [{
      ResourceType: 'instance',
      Tags: [{ Key: 'Name', Value: 'MyInstance' }]
    }]
  });
  
  try {
    const response = await ec2Client.send(command);
    console.log('Instance launched:', response.Instances[0].InstanceId);
    return response.Instances[0];
  } catch (error) {
    console.error('Launch failed:', error);
    throw error;
  }
}
```

```javascript
// AWS SDK v2 - Classic approach
const AWS = require('aws-sdk');

// Configure SDK
AWS.config.update({
  region: 'us-east-1',
  accessKeyId: process.env.AWS_ACCESS_KEY_ID,
  secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
});

// Create service clients
const s3 = new AWS.S3();
const ec2 = new AWS.EC2();
const lambda = new AWS.Lambda();
const dynamoDB = new AWS.DynamoDB.DocumentClient();

// S3 operations
async function listBuckets() {
  try {
    const result = await s3.listBuckets().promise();
    return result.Buckets;
  } catch (error) {
    console.error('Error listing buckets:', error);
    throw error;
  }
}

// EC2 operations
async function describeInstances() {
  try {
    const result = await ec2.describeInstances().promise();
    return result.Reservations.flatMap(r => r.Instances);
  } catch (error) {
    console.error('Error describing instances:', error);
    throw error;
  }
}

// Lambda operations
async function invokeFunction(functionName, payload) {
  const params = {
    FunctionName: functionName,
    Payload: JSON.stringify(payload)
  };
  
  try {
    const result = await lambda.invoke(params).promise();
    return JSON.parse(result.Payload);
  } catch (error) {
    console.error('Error invoking function:', error);
    throw error;
  }
}
```

---

### Common CLI Commands

**S3 Commands**

```bash
# List buckets
aws s3 ls

# List objects in bucket
aws s3 ls s3://my-bucket/

# Copy file to S3
aws s3 cp local-file.txt s3://my-bucket/

# Sync directory to S3
aws s3 sync ./local-dir s3://my-bucket/remote-dir --delete

# Delete object
aws s3 rm s3://my-bucket/file.txt

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled

# Set lifecycle policy
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-bucket \
  --lifecycle-configuration '{
    "Rules": [
      {
        "Id": "DeleteOldVersions",
        "Status": "Enabled",
        "Prefix": "",
        "NoncurrentVersionExpiration": {
          "NoncurrentDays": 30
        }
      }
    ]
  }'
```

**EC2 Commands**

```bash
# List instances
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].[InstanceId,State.Name,Tags[?Key==`Name`].Value]' \
  --output table

# Launch instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name my-key \
  --security-group-ids sg-12345678

# Stop instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Terminate instance
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# Create security group
aws ec2 create-security-group \
  --group-name my-sg \
  --description "My security group"

# Add rule to security group
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

**Lambda Commands**

```bash
# List functions
aws lambda list-functions

# Create function
aws lambda create-function \
  --function-name my-function \
  --runtime nodejs18.x \
  --handler index.handler \
  --role arn:aws:iam::123456789012:role/lambda-role \
  --zip-file fileb://function.zip

# Invoke function
aws lambda invoke \
  --function-name my-function \
  --payload '{"key": "value"}' \
  response.json

# Update function code
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://new-function.zip

# Add environment variables
aws lambda update-function-configuration \
  --function-name my-function \
  --environment Variables={API_KEY=secret123}
```

**RDS Commands**

```bash
# List DB instances
aws rds describe-db-instances \
  --query 'DBInstances[*].[DBInstanceIdentifier,DBInstanceStatus,Engine]'

# Create DB instance
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --master-username admin \
  --master-user-password MyPassword123!

# Create snapshot
aws rds create-db-snapshot \
  --db-instance-identifier mydb \
  --db-snapshot-identifier mydb-snapshot

# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier mydb-restored \
  --db-snapshot-identifier mydb-snapshot
```

---

## Hands-on AWS Projects

### Project 1: Serverless REST API

**Architecture**

API Gateway → Lambda → DynamoDB. Fully serverless, auto-scaling, pay-per-use.

```
Architecture Diagram:

Client Request
       ↓
API Gateway
       ↓
   Lambda Function
       ↓
   DynamoDB Table
```

```javascript
// lambda/user-service.js
const AWS = require('aws-sdk');
const dynamoDB = new AWS.DynamoDB.DocumentClient();
const TABLE_NAME = process.env.USERS_TABLE;

// Create user
exports.createUser = async (event) => {
  const body = JSON.parse(event.body);
  const params = {
    TableName: TABLE_NAME,
    Item: {
      id: Date.now().toString(),
      name: body.name,
      email: body.email,
      createdAt: new Date().toISOString()
    }
  };
  
  await dynamoDB.put(params).promise();
  
  return {
    statusCode: 201,
    body: JSON.stringify(params.Item)
  };
};

// Get user
exports.getUser = async (event) => {
  const params = {
    TableName: TABLE_NAME,
    Key: { id: event.pathParameters.id }
  };
  
  const result = await dynamoDB.get(params).promise();
  
  if (!result.Item) {
    return {
      statusCode: 404,
      body: JSON.stringify({ message: 'User not found' })
    };
  }
  
  return {
    statusCode: 200,
    body: JSON.stringify(result.Item)
  };
};

// List users
exports.listUsers = async () => {
  const params = {
    TableName: TABLE_NAME
  };
  
  const result = await dynamoDB.scan(params).promise();
  
  return {
    statusCode: 200,
    body: JSON.stringify(result.Items)
  };
};

// Update user
exports.updateUser = async (event) => {
  const body = JSON.parse(event.body);
  const params = {
    TableName: TABLE_NAME,
    Key: { id: event.pathParameters.id },
    UpdateExpression: 'SET #n = :n, #e = :e',
    ExpressionAttributeNames: {
      '#n': 'name',
      '#e': 'email'
    },
    ExpressionAttributeValues: {
      ':n': body.name,
      ':e': body.email
    },
    ReturnValues: 'ALL_NEW'
  };
  
  const result = await dynamoDB.update(params).promise();
  
  return {
    statusCode: 200,
    body: JSON.stringify(result.Attributes)
  };
};

// Delete user
exports.deleteUser = async (event) => {
  const params = {
    TableName: TABLE_NAME,
    Key: { id: event.pathParameters.id }
  };
  
  await dynamoDB.delete(params).promise();
  
  return {
    statusCode: 204,
    body: ''
  };
};
```

```bash
# Deploy infrastructure using CloudFormation
aws cloudformation create-stack \
  --stack-name user-api-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=UsersTableName,ParameterValue=Users

# template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  UsersTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: Users
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      KeySchema:
        - AttributeName: id
          KeyType: HASH
      BillingMode: PAY_PER_REQUEST
  
  CreateUserFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: CreateUser
      Runtime: nodejs18.x
      Handler: user-service.createUser
      Code:
        ZipFile: |
          const AWS = require('aws-sdk');
          const dynamoDB = new AWS.DynamoDB.DocumentClient();
          exports.createUser = async (event) => {
            const body = JSON.parse(event.body);
            const params = {
              TableName: process.env.USERS_TABLE,
              Item: { id: Date.now().toString(), ...body }
            };
            await dynamoDB.put(params).promise();
            return { statusCode: 201, body: JSON.stringify(params.Item) };
          };
      Environment:
        Variables:
          USERS_TABLE: !Ref UsersTable
      Role: !GetAtt LambdaRole.Arn
  
  ApiGateway:
    Type: AWS::ApiGateway::RestApi
    Properties:
      Name: UserAPI
  
  CreateUserResource:
    Type: AWS::ApiGateway::Resource
    Properties:
      RestApiId: !Ref ApiGateway
      ParentId: !GetAtt ApiGateway.RootResourceId
      PathPart: users
  
  CreateUserMethod:
    Type: AWS::ApiGateway::Method
    Properties:
      RestApiId: !Ref ApiGateway
      ResourceId: !Ref CreateUserResource
      HttpMethod: POST
      AuthorizationType: NONE
      Integration:
        Type: AWS_PROXY
        IntegrationHttpMethod: POST
        Uri: !Sub arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${CreateUserFunction.Arn}/invocations
  
  LambdaRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

---

### Project 2: EC2 Web Application

**Architecture**

ELB → Auto Scaling Group → EC2 instances → RDS. Highly available, scalable web application.

```
Architecture Diagram:

           Internet
              ↓
         [Load Balancer]
              ↓
    +---------+---------+
    |         |         |
[EC2]     [EC2]     [EC2]
    |         |         |
    +---------+---------+
              ↓
          [RDS Database]
```

```bash
# deployment/deploy.sh
#!/bin/bash

# Configuration
APP_NAME="web-app"
ENVIRONMENT="production"
REGION="us-east-1"
KEY_NAME="my-key-pair"
SECURITY_GROUP="sg-12345678"
SUBNET_IDS="subnet-12345678 subnet-87654321"

# Create AMI from existing instance
create_ami() {
  local instance_id=$1
  local ami_name="${APP_NAME}-${ENVIRONMENT}-$(date +%Y%m%d)"
  
  echo "Creating AMI from instance ${instance_id}..."
  aws ec2 create-image \
    --instance-id $instance_id \
    --name $ami_name \
    --description "AMI for ${APP_NAME} ${ENVIRONMENT}" \
    --region $REGION
}

# Create launch template
create_launch_template() {
  local ami_id=$1
  
  echo "Creating launch template..."
  aws ec2 create-launch-template \
    --launch-template-name ${APP_NAME}-template \
    --launch-template-data "{
      \"ImageId\": \"$ami_id\",
      \"InstanceType\": \"t3.micro\",
      \"KeyName\": \"$KEY_NAME\",
      \"SecurityGroupIds\": [\"$SECURITY_GROUP\"],
      \"UserData\": \"$(base64 -w 0 user-data.sh)\",
      \"TagSpecifications\": [{
        \"ResourceType\": \"instance\",
        \"Tags\": [{
          \"Key\": \"Name\",
          \"Value\": \"${APP_NAME}-${ENVIRONMENT}\"
        }]
      }]
    }"
}

# Create Auto Scaling Group
create_asg() {
  local template_name=$1
  
  echo "Creating Auto Scaling Group..."
  aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name ${APP_NAME}-asg \
    --launch-template LaunchTemplateName=$template_name \
    --min-size 2 \
    --max-size 6 \
    --desired-capacity 2 \
    --target-group-arns arn:aws:elasticloadbalancing:$REGION:123456789012:targetgroup/my-targets/abc123 \
    --vpc-zone-identifier $SUBNET_IDS \
    --health-check-type ELB \
    --health-check-grace-period 300
}

# Create scaling policy
create_scaling_policy() {
  echo "Creating scaling policy..."
  aws autoscaling put-scaling-policy \
    --auto-scaling-group-name ${APP_NAME}-asg \
    --policy-name ${APP_NAME}-scale-out \
    --scaling-adjustment 1 \
    --adjustment-type ChangeInCapacity \
    --cooldown 300
}

# Main deployment
main() {
  echo "Starting deployment of ${APP_NAME} to ${ENVIRONMENT}..."
  
  # Get latest instance
  INSTANCE_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${APP_NAME}-staging" \
    --query "Reservations[0].Instances[0].InstanceId" \
    --output text)
  
  # Create AMI
  AMI_ID=$(create_ami $INSTANCE_ID | jq -r '.ImageId')
  echo "AMI created: $AMI_ID"
  
  # Wait for AMI to be available
  aws ec2 wait image-available --image-ids $AMI_ID
  
  # Create launch template
  create_launch_template $AMI_ID
  
  # Create Auto Scaling Group
  create_asg ${APP_NAME}-template
  
  # Create scaling policy
  create_scaling_policy
  
  echo "Deployment completed successfully!"
}

main
```

```bash
# deployment/user-data.sh
#!/bin/bash

# Update system
yum update -y

# Install Node.js
curl -fsSL https://rpm.nodesource.com/setup_18.x | bash -
yum install -y nodejs

# Install PM2
npm install -g pm2

# Create app directory
mkdir -p /var/www/app
cd /var/www/app

# Download application from S3
aws s3 cp s3://my-app-bucket/app.zip .
unzip app.zip
rm app.zip

# Install dependencies
npm install --production

# Start application with PM2
pm2 start server.js --name web-app

# Configure PM2 to start on boot
pm2 startup systemd -u ec2-user --hp /home/ec2-user
pm2 save

# Configure Nginx
cat > /etc/nginx/conf.d/web-app.conf <<EOF
server {
    listen 80;
    server_name _;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host \$host;
        proxy_cache_bypass \$http_upgrade;
    }
}
EOF

# Start Nginx
systemctl start nginx
systemctl enable nginx
```

```javascript
// server.js - Express application
const express = require('express');
const mysql = require('mysql2/promise');
const app = express();

app.use(express.json());

// Database connection
const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({ status: 'healthy', timestamp: new Date().toISOString() });
});

// Get users
app.get('/api/users', async (req, res) => {
  try {
    const [rows] = await pool.query('SELECT * FROM users');
    res.json(rows);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Create user
app.post('/api/users', async (req, res) => {
  const { name, email } = req.body;
  
  try {
    const [result] = await pool.query(
      'INSERT INTO users (name, email) VALUES (?, ?)',
      [name, email]
    );
    
    res.status(201).json({ id: result.insertId, name, email });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

### Project 3: Data Pipeline

**Architecture**

Kinesis → Lambda → S3 → Athena. Real-time data ingestion and analytics.

```
Architecture Diagram:

Data Sources
     ↓
[Kinesis Data Stream]
     ↓
[Lambda Processor]
     ↓
[S3 Data Lake]
     ↓
[Athena Queries]
     ↓
[QuickSight Dashboards]
```

```javascript
// lambda/processor.js
const AWS = require('aws-sdk');
const s3 = new AWS.S3();
const BUCKET_NAME = process.env.DATA_LAKE_BUCKET;

exports.handler = async (event) => {
  const records = event.Records;
  const processedData = [];
  
  for (const record of records) {
    try {
      // Parse Kinesis record
      const data = Buffer.from(record.kinesis.data, 'base64').toString();
      const payload = JSON.parse(data);
      
      // Process data
      const processed = {
        ...payload,
        processedAt: new Date().toISOString(),
        partitionKey: `${payload.eventType}/${new Date().toISOString().slice(0, 10)}`
      };
      
      processedData.push(processed);
    } catch (error) {
      console.error('Error processing record:', error);
    }
  }
  
  // Write to S3
  if (processedData.length > 0) {
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
    const key = `processed/${timestamp}.json`;
    
    await s3.putObject({
      Bucket: BUCKET_NAME,
      Key: key,
      Body: JSON.stringify(processedData, null, 2),
      ContentType: 'application/json'
    }).promise();
    
    console.log(`Wrote ${processedData.length} records to ${key}`);
  }
  
  return { recordsProcessed: records.length };
};
```

```bash
# infrastructure/setup-pipeline.sh
#!/bin/bash

# Configuration
STREAM_NAME="data-stream"
FUNCTION_NAME="data-processor"
BUCKET_NAME="my-data-lake"
REGION="us-east-1"

# Create Kinesis data stream
echo "Creating Kinesis stream..."
aws kinesis create-stream \
  --stream-name $STREAM_NAME \
  --shard-count 2

# Wait for stream to become active
aws kinesis wait stream-exists --stream-name $STREAM_NAME

# Create S3 bucket
echo "Creating S3 bucket..."
aws s3 mb s3://$BUCKET_NAME

# Create IAM role for Lambda
echo "Creating IAM role..."
aws iam create-role \
  --role-name lambda-kinesis-role \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": { "Service": "lambda.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }]
  }'

# Attach policies to role
aws iam attach-role-policy \
  --role-name lambda-kinesis-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Create custom policy for Kinesis and S3
cat > lambda-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "kinesis:GetRecords",
        "kinesis:GetShardIterator",
        "kinesis:DescribeStream",
        "kinesis:ListShards"
      ],
      "Resource": "arn:aws:kinesis:$REGION:*:stream/$STREAM_NAME"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::$BUCKET_NAME/*"
    }
  ]
}
EOF

aws iam put-role-policy \
  --role-name lambda-kinesis-role \
  --policy-name lambda-kinesis-s3-policy \
  --policy-document file://lambda-policy.json

# Create Lambda function
echo "Creating Lambda function..."
aws lambda create-function \
  --function-name $FUNCTION_NAME \
  --runtime nodejs18.x \
  --handler processor.handler \
  --role arn:aws:iam::123456789012:role/lambda-kinesis-role \
  --zip-file fileb://function.zip \
  --environment Variables={DATA_LAKE_BUCKET=$BUCKET_NAME} \
  --timeout 60 \
  --memory-size 256

# Create event source mapping
echo "Creating event source mapping..."
aws lambda create-event-source-mapping \
  --function-name $FUNCTION_NAME \
  --batch-size 100 \
  --maximum-batching-window-in-seconds 5 \
  --starting-position LATEST \
  --event-source-arn arn:aws:kinesis:$REGION:123456789012:stream/$STREAM_NAME

# Create Athena database
echo "Creating Athena database..."
aws athena start-query-execution \
  --query-string "CREATE DATABASE IF NOT EXISTS data_lake" \
  --result-configuration "OutputLocation=s3://$BUCKET_NAME/athena-results/"

echo "Pipeline setup completed!"
```

```sql
-- analytics/queries.sql

-- Create external table for processed data
CREATE EXTERNAL TABLE IF NOT EXISTS data_lake.processed_events (
  event_type STRING,
  user_id STRING,
  timestamp STRING,
  processed_at STRING,
  partition_key STRING
)
PARTITIONED BY (dt STRING)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
STORED AS INPUTFORMAT 'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION 's3://my-data-lake/processed/';

-- Add partition
ALTER TABLE data_lake.processed_events ADD IF NOT EXISTS PARTITION (dt='2024-01-01')
LOCATION 's3://my-data-lake/processed/2024-01-01/';

-- Query event counts by type
SELECT 
  event_type,
  COUNT(*) as event_count,
  COUNT(DISTINCT user_id) as unique_users
FROM data_lake.processed_events
WHERE dt = '2024-01-01'
GROUP BY event_type
ORDER BY event_count DESC;

-- Query hourly activity
SELECT 
  hour(timestamp) as hour,
  COUNT(*) as event_count
FROM data_lake.processed_events
WHERE dt = '2024-01-01'
GROUP BY hour(timestamp)
ORDER BY hour;
```

---

### Project 4: CI/CD Pipeline

**Complete Pipeline**

CodeCommit → CodeBuild → CodeDeploy → Elastic Beanstalk. Automated testing and deployment.

```
Pipeline Flow:

Developer Push
       ↓
[CodeCommit]
       ↓
[CodeBuild - Test & Build]
       ↓
[CodeDeploy - Deploy to Staging]
       ↓
[Manual Approval]
       ↓
[CodeDeploy - Deploy to Production]
```

```yaml
# pipeline/pipeline.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Complete CI/CD Pipeline'

Parameters:
  RepositoryName:
    Type: String
    Default: my-app-repo
  BranchName:
    Type: String
    Default: main
  ApplicationName:
    Type: String
    Default: MyApplication

Resources:
  # CodePipeline Service Role
  PipelineRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: codepipeline.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AWSCodePipelineFullAccess
        - arn:aws:iam::aws:policy/AWSCodeBuildAdminAccess
        - arn:aws:iam::aws:policy/AWSCodeDeployFullAccess

  # CodeBuild Service Role
  BuildRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: codebuild.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AWSCodeBuildDeveloperAccess
        - arn:aws:iam::aws:policy/AmazonS3FullAccess

  # CodeBuild Project
  BuildProject:
    Type: AWS::CodeBuild::Project
    Properties:
      Name: !Sub '${ApplicationName}-build'
      ServiceRole: !GetAtt BuildRole.Arn
      Artifacts:
        Type: CODEPIPELINE
      Environment:
        Type: LINUX_CONTAINER
        ComputeType: BUILD_GENERAL1_SMALL
        Image: aws/codebuild/standard:6.0
        EnvironmentVariables:
          - Name: NODE_ENV
            Value: test
      Source:
        Type: CODEPIPELINE
        BuildSpec: buildspec.yml

  # CodeDeploy Application
  DeployApplication:
    Type: AWS::CodeDeploy::Application
    Properties:
      ApplicationName: !Ref ApplicationName
      ComputePlatform: Server

  # CodeDeploy Deployment Group (Staging)
  StagingDeploymentGroup:
    Type: AWS::CodeDeploy::DeploymentGroup
    Properties:
      ApplicationName: !Ref DeployApplication
      DeploymentGroupName: Staging
      DeploymentConfigName: CodeDeployDefault.OneAtATime
      ServiceRoleArn: !GetAtt CodeDeployRole.Arn
      AutoScalingGroups:
        - !Ref StagingASG

  # CodeDeploy Deployment Group (Production)
  ProductionDeploymentGroup:
    Type: AWS::CodeDeploy::DeploymentGroup
    Properties:
      ApplicationName: !Ref DeployApplication
      DeploymentGroupName: Production
      DeploymentConfigName: CodeDeployDefault.HalfAtATime
      ServiceRoleArn: !GetAtt CodeDeployRole.Arn
      AutoScalingGroups:
        - !Ref ProductionASG

  # CodeDeploy Service Role
  CodeDeployRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: codedeploy.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSCodeDeployRole

  # Auto Scaling Groups
  StagingASG:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      MinSize: 1
      MaxSize: 2
      DesiredCapacity: 1
      VPCZoneIdentifier:
        - subnet-12345678
      LaunchTemplate:
        LaunchTemplateId: !Ref LaunchTemplate
        Version: !GetAtt LaunchTemplate.LatestVersionNumber

  ProductionASG:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      MinSize: 2
      MaxSize: 6
      DesiredCapacity: 2
      VPCZoneIdentifier:
        - subnet-12345678
        - subnet-87654321
      LaunchTemplate:
        LaunchTemplateId: !Ref LaunchTemplate
        Version: !GetAtt LaunchTemplate.LatestVersionNumber

  # Launch Template
  LaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Properties:
      LaunchTemplateName: !Sub '${ApplicationName}-template'
      LaunchTemplateData:
        InstanceType: t3.micro
        ImageId: ami-0c55b159cbfafe1f0
        KeyName: my-key-pair
        SecurityGroupIds:
          - sg-12345678

  # Pipeline
  Pipeline:
    Type: AWS::CodePipeline::Pipeline
    Properties:
      Name: !Sub '${ApplicationName}-pipeline'
      RoleArn: !GetAtt PipelineRole.Arn
      Stages:
        - Name: Source
          Actions:
            - Name: SourceAction
              ActionTypeId:
                Category: Source
                Owner: AWS
                Provider: CodeCommit
                Version: '1'
              Configuration:
                RepositoryName: !Ref RepositoryName
                BranchName: !Ref BranchName
              OutputArtifacts:
                - Name: SourceOutput
        - Name: Build
          Actions:
            - Name: BuildAction
              ActionTypeId:
                Category: Build
                Owner: AWS
                Provider: CodeBuild
                Version: '1'
              Configuration:
                ProjectName: !Ref BuildProject
              InputArtifacts:
                - Name: SourceOutput
              OutputArtifacts:
                - Name: BuildOutput
        - Name: DeployStaging
          Actions:
            - Name: DeployStagingAction
              ActionTypeId:
                Category: Deploy
                Owner: AWS
                Provider: CodeDeploy
                Version: '1'
              Configuration:
                ApplicationName: !Ref DeployApplication
                DeploymentGroupName: Staging
              InputArtifacts:
                - Name: BuildOutput
        - Name: Approve
          Actions:
            - Name: ManualApproval
              ActionTypeId:
                Category: Approval
                Owner: AWS
                Provider: Manual
                Version: '1'
        - Name: DeployProduction
          Actions:
            - Name: DeployProductionAction
              ActionTypeId:
                Category: Deploy
                Owner: AWS
                Provider: CodeDeploy
                Version: '1'
              Configuration:
                ApplicationName: !Ref DeployApplication
                DeploymentGroupName: Production
              InputArtifacts:
                - Name: BuildOutput

Outputs:
  PipelineURL:
    Description: CodePipeline Console URL
    Value: !Sub 'https://console.aws.amazon.com/codesuite/codepipeline/pipelines/${ApplicationName}-pipeline/view?region=${AWS::Region}'
```

```yaml
# buildspec.yml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 18
    commands:
      - npm install -g npm
  pre_build:
    commands:
      - echo Installing dependencies...
      - npm ci
  build:
    commands:
      - echo Running tests...
      - npm test
      - echo Building application...
      - npm run build
      - echo Creating deployment package...
      - zip -r application.zip build/ node_modules/ package.json
  post_build:
    commands:
      - echo Build completed on `date`

artifacts:
  files:
    - application.zip
  discard-paths: yes

cache:
  paths:
    - node_modules/**/*
```

---

## Quick Reference

**AWS Service Comparison**

| Service | Type | Use Case | Pricing Model |
|---------|------|-----------|--------------|
| EC2 | Compute | Virtual servers | Per hour/second |
| Lambda | Compute | Serverless functions | Per request + compute time |
| Elastic Beanstalk | PaaS | Application deployment | Per underlying resources |
| S3 | Storage | Object storage | Per GB + requests |
| EBS | Storage | Block storage for EC2 | Per GB-month + IOPS |
| EFS | Storage | Network file system | Per GB-month + throughput |
| RDS | Database | Managed relational DB | Per instance hour + storage |
| DynamoDB | Database | NoSQL database | Per read/write + storage |
| VPC | Networking | Isolated network | Free (pay for components) |
| ELB | Networking | Load balancing | Per hour + LCU |
| API Gateway | Networking | API management | Per million calls + data |
| CloudFront | CDN | Content delivery | Per GB + requests |
| CloudWatch | Monitoring | Metrics & logs | Per metric + logs |
| CloudTrail | Auditing | API audit trail | Per event + storage |

**IAM Policy Elements**

| Element | Description | Example |
|---------|-------------|---------|
| Version | Policy language version | "2012-10-17" |
| Statement | Container for statements | [ {...}, {...} ] |
| Sid | Statement ID | "AllowS3Read" |
| Effect | Allow or Deny | "Allow" |
| Action | Specific AWS action | "s3:GetObject" |
| Resource | Target resource | "arn:aws:s3:::bucket/*" |
| Condition | Additional constraints | { "IpAddress": { "aws:SourceIp": "10.0.0.0/8" } } |

**Common CLI Commands**

```bash
# S3
aws s3 ls                                    # List buckets
aws s3 cp file.txt s3://bucket/              # Upload file
aws s3 sync ./dir s3://bucket/dir            # Sync directory
aws s3 rm s3://bucket/file.txt               # Delete file

# EC2
aws ec2 describe-instances                    # List instances
aws ec2 run-instances ...                    # Launch instance
aws ec2 stop-instances --instance-ids id      # Stop instance
aws ec2 terminate-instances --instance-ids id # Terminate instance

# Lambda
aws lambda list-functions                     # List functions
aws lambda invoke --function-name fn ...      # Invoke function
aws lambda update-function-code ...           # Update code

# RDS
aws rds describe-db-instances                # List databases
aws rds create-db-instance ...              # Create database
aws rds create-db-snapshot ...              # Create snapshot

# IAM
aws iam list-users                           # List users
aws iam create-user --user-name name         # Create user
aws iam attach-user-policy ...                # Attach policy
```

**Deployment Strategies Comparison**

| Strategy | Downtime | Rollback | Complexity | Use Case |
|----------|-----------|----------|------------|-----------|
| All at Once | Yes | Fast | Low | Small apps, non-critical |
| Rolling | No | Fast | Medium | Stateless apps |
| Blue/Green | No | Instant | High | Critical apps, zero downtime |
| Canary | No | Fast | High | Gradual rollout, testing |
| Linear | No | Medium | Medium | Controlled rollout |

**Cost Optimization Tips**

- ✅ Use Reserved Instances for steady workloads
- ✅ Use Spot Instances for fault-tolerant workloads
- ✅ Enable S3 lifecycle policies for cost-effective storage
- ✅ Use Auto Scaling to match capacity with demand
- ✅ Monitor costs with AWS Cost Explorer
- ✅ Set up billing alerts
- ✅ Use AWS Free Tier for testing
- ✅ Terminate unused resources
- ✅ Use appropriate instance types
- ✅ Enable S3 Transfer Acceleration for frequent transfers

**Security Checklist**

- ✅ Enable MFA for root account
- ✅ Use IAM roles instead of access keys
- ✅ Rotate credentials regularly
- ✅ Enable CloudTrail logging
- ✅ Use security groups and NACLs
- ✅ Encrypt data at rest and in transit
- ✅ Enable VPC flow logs
- ✅ Use AWS Config for compliance
- ✅ Regularly review IAM policies
- ✅ Enable GuardDuty for threat detection

**Monitoring & Logging**

```bash
# CloudWatch metrics
aws cloudwatch put-metric-data \
  --namespace MyApplication \
  --metric-name RequestCount \
  --value 100 \
  --unit Count

# Create CloudWatch alarm
aws cloudwatch put-metric-alarm \
  --alarm-name HighCPU \
  --alarm-description "CPU > 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold

# Query CloudWatch Logs
aws logs start-query \
  --log-group-name /aws/lambda/my-function \
  --start-time $(date -d '-1 hour' +%s) \
  --end-time $(date +%s) \
  --query-string "fields @timestamp, @message | filter @message like /ERROR/"

# Enable VPC Flow Logs
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-id vpc-12345678 \
  --traffic-type ALL \
  --log-group-name /aws/vpc/flow-logs \
  --deliver-logs-permission-arn arn:aws:iam::123456789012:role/flow-logs-role
```

**Troubleshooting Commands**

```bash
# Check EC2 instance status
aws ec2 describe-instance-status \
  --instance-ids i-1234567890abcdef0

# View CloudTrail events
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=ResourceType,AttributeValue=AWS::EC2::Instance

# Get Lambda logs
aws logs tail /aws/lambda/my-function --follow

# Check RDS performance
aws rds describe-db-log-files \
  --db-instance-identifier mydb

# Verify IAM permissions
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/john.doe \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::my-bucket/*
```

**Best Practices Summary**

- **Compute**: Use Auto Scaling, choose right instance types, leverage Spot Instances
- **Storage**: Use lifecycle policies, enable versioning, choose correct storage class
- **Database**: Use Multi-AZ for production, enable backups, use read replicas
- **Networking**: Use VPC, implement security groups, enable flow logs
- **Security**: Follow least privilege, enable MFA, encrypt everything
- **Deployment**: Use CI/CD, test thoroughly, implement rollback strategy
- **Monitoring**: Use CloudWatch, set up alarms, review logs regularly
- **Cost**: Monitor spending, use reserved instances, clean up resources
- **Compliance**: Use AWS Config, enable CloudTrail, conduct audits
- **Automation**: Use Infrastructure as Code, automate deployments, document everything
