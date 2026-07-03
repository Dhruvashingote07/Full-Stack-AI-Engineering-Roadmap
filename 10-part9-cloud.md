# Part 9: Cloud

---

## Chapter 78: AWS — EC2 & Compute

### Introduction

Amazon Web Services (AWS) is the largest public cloud provider with over 200 services. EC2 (Elastic Compute Cloud) provides virtual machines in the cloud.

### EC2 Instance Types

| Family | Use Case | Examples |
|--------|----------|---------|
| General purpose | Balanced compute, memory, networking | t3, m7g |
| Compute optimized | Compute-intensive workloads | c7g, c6i |
| Memory optimized | Memory-intensive workloads | r7g, x2iedn |
| Storage optimized | High sequential I/O | i4i, d3en |
| Accelerated computing | GPU/FPGA workloads | p4d, g5 |

### EC2 Key Concepts

- **AMI**: Amazon Machine Image (OS template)
- **Instance Type**: CPU, memory, storage specification
- **Security Group**: Virtual firewall (allow/deny rules)
- **Key Pair**: SSH public/private key for access
- **EBS**: Elastic Block Store (persistent block storage)
- **User Data**: Scripts that run on first boot

```bash
# Launch EC2 via AWS CLI
aws ec2 run-instances \
  --image-id ami-12345678 \
  --instance-type t3.medium \
  --key-name my-key \
  --security-group-ids sg-12345678 \
  --subnet-id subnet-12345678 \
  --user-data file://bootstrap.sh \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=web-server}]'

# SSH into instance
ssh -i my-key.pem ec2-user@54.123.45.67
```

> ⚠️ **Warning:** Leaving security groups overly permissive (e.g., `0.0.0.0/0` on SSH port 22) is the most common way EC2 instances get compromised. Always restrict SSH access to specific IP ranges or use a bastion host.

> ⚠️ **Warning:** EC2 root volumes are deleted by default when an instance is terminated. Always configure `DeleteOnTermination=false` for the root EBS volume if you need to preserve data after termination.

> ✅ **Best Practice:** Use EC2 instance profiles (IAM Roles) instead of hardcoding AWS access keys on instances. Attach a role with least-privilege permissions to the EC2 instance at launch time.

> ✅ **Best Practice:** Use Auto Scaling Groups with multiple Availability Zones to ensure high availability. Spread instances across at least 2-3 AZs so a single AZ failure doesn't take down your application.

> ❓ **Interview Question:** What is the difference between stopping and terminating an EC2 instance?
> **Answer:** Stopping transitions the instance to the `stopped` state — EBS volumes persist, and you can start it again. Terminating permanently deletes the instance, and by default, root EBS volumes are also deleted.

> 💡 **Pro Tip:** Use EC2 Instance Connect (AWS Console or CLI) as an alternative to managing SSH keys. It uses temporary SSH keys that expire after 60 seconds, reducing the key management overhead.

### Real World Analogy

EC2 instances are like rental apartments. You choose the **instance type** (studio, 1-bedroom, penthouse). **AMIs** are furnished vs unfurnished units. **Security groups** are the building security — who can enter which floor. **EBS volumes** are the storage unit you rent. **Key pairs** are your apartment keys. You pay by the hour (or second), and you can move out (terminate) anytime, but the storage (EBS) can be kept or discarded.

### Practical Exercises

1. Launch an EC2 instance using AWS CLI with a security group that only allows SSH from your IP
2. Create an AMI from a running EC2 instance and launch a second instance from it
3. Attach an additional EBS volume, format it with ext4, mount it, and configure auto-mount on reboot
4. Set up an Application Load Balancer in front of two EC2 instances and verify traffic distribution
5. Create an Auto Scaling Group with a launch template, min 2 and max 6 instances, and test scale-out

### Revision Notes

- **EC2**: Virtual machines with various instance families (general, compute, memory, storage, accelerated)
- **AMI**: Pre-configured OS template; can be custom or from AWS Marketplace
- **Security Groups**: Stateful virtual firewalls at instance level — allow rules only
- **EBS**: Persistent block storage — gp3 (general), io2 (provisioned IOPS), st1 (throughput)
- **Instance metadata**: Accessible at `http://169.254.169.254/latest/meta-data/`
- **User Data**: Scripts run on first boot — useful for bootstrapping and configuration

## Chapter 79: AWS — S3

### Introduction

Amazon S3 (Simple Storage Service) is an object storage service offering industry-leading scalability, durability (99.999999999%), and security.

### S3 Storage Classes

| Class | Durability | Availability | Min Storage | Retrieval | Use Case |
|-------|-----------|--------------|-------------|-----------|----------|
| Standard | 11 nines | 99.99% | None | Instant | Frequently accessed |
| Intelligent-Tiering | 11 nines | 99.9% | None | Instant | Unknown patterns |
| Standard-IA | 11 nines | 99.9% | 30 days | Instant | Infrequent access |
| One Zone-IA | 11 nines | 99.5% | 30 days | Instant | Recreatable data |
| Glacier Instant | 11 nines | 99.9% | 90 days | Milliseconds | Archive instant access |
| Glacier Flexible | 11 nines | 99.99% | 90 days | Minutes to hours | Archive backups |
| Glacier Deep Archive | 11 nines | 99.99% | 180 days | 12 hours | Long-term archives |

### S3 Lifecycle Policies

```json
{
  "Rules": [
    {
      "Id": "Move to IA after 30 days",
      "Status": "Enabled",
      "Filter": { "Prefix": "logs/" },
      "Transitions": [
        { "Days": 30, "StorageClass": "STANDARD_IA" },
        { "Days": 90, "StorageClass": "GLACIER" }
      ],
      "Expiration": { "Days": 365 }
    }
  ]
}
```

### S3 Bucket Operations

```bash
# Create bucket
aws s3 mb s3://my-unique-bucket-name --region us-east-1

# Upload file
aws s3 cp myfile.txt s3://my-bucket/path/

# Sync directory
aws s3 sync ./dist s3://my-bucket --delete

# List objects
aws s3 ls s3://my-bucket/

# Presigned URL (temporary access)
aws s3 presign s3://my-bucket/private-file.pdf --expires-in 3600

# Enable versioning
aws s3api put-bucket-versioning --bucket my-bucket --versioning-configuration Status=Enabled
```

### S3 Security

- **Block Public Access**: Default on for new buckets
- **Bucket Policies**: Resource-based JSON policies
- **ACLs**: Legacy access control (disabled by default)
- **Encryption**: SSE-S3, SSE-KMS, SSE-C
- **CORS**: Cross-origin resource sharing configuration

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/public/*"
    }
  ]
}
```

> ⚠️ **Warning:** A misconfigured S3 bucket policy with `"Principal": "*"` and `"Action": "s3:*"` makes the bucket publicly writable — anyone on the internet can upload, modify, or delete objects. Always use `"Principal": "*"` only with read-only actions and restricted paths.

> ⚠️ **Warning:** S3 bucket names are globally unique across all AWS accounts. Once a name is taken, no one else can use it. Plan bucket naming conventions carefully for multi-account or multi-region deployments.

> ✅ **Best Practice:** Enable S3 Block Public Access at the account level as a safety net. Then explicitly grant public access only to specific buckets that need it (e.g., static website hosting).

> ✅ **Best Practice:** Enable S3 Versioning on all buckets to protect against accidental deletions and overwrites. Combine with a lifecycle policy to clean up old versions after a retention period.

> ❓ **Interview Question:** How does S3 achieve 99.999999999% (11 nines) durability?
> **Answer:** S3 automatically replicates objects across a minimum of three Availability Zones within a region (except One Zone-IA). It uses erasure coding and CRC checksums to detect and repair data corruption.

> 💡 **Pro Tip:** Use S3 Transfer Acceleration for faster uploads over long distances. It uses AWS edge locations to route traffic through optimized AWS network paths rather than the public internet.

### Real World Analogy

S3 is like a giant, infinitely expandable warehouse. Each **bucket** is a separate warehouse building. **Objects** (files) are stored on shelves with keys (filenames). **Storage classes** are different sections: Standard (front shelves, quick access), Standard-IA (back shelves), Glacier (offsite storage, slow retrieval). **Versioning** is like keeping old versions of documents in case you need to revert. **Lifecycle policies** are the automated system that moves items to cheaper storage as they age.

### Practical Exercises

1. Create an S3 bucket, enable versioning, upload a file, modify it, then restore the previous version
2. Configure a lifecycle policy that transitions objects to Standard-IA after 30 days and Glacier after 90 days
3. Set up a static website hosting bucket with a bucket policy allowing public read access
4. Generate a presigned URL for a private object with 1-hour expiry and verify access using `curl`
5. Use S3 Event Notifications to trigger a Lambda function when a new file is uploaded

### Revision Notes

- **S3**: Object storage, 11 nines durability, unlimited storage
- **Storage classes**: Standard → Intelligent-Tiering → Standard-IA → Glacier → Deep Archive
- **Security**: Block Public Access, Bucket Policies, IAM policies, Encryption (SSE-S3/SSE-KMS/SSE-C)
- **Versioning**: Protects against accidental deletions; combine with lifecycle policies
- **Lifecycle**: Automate transitions between storage classes and expiration
- **Performance**: Use multipart upload for >100MB objects, S3 Transfer Acceleration for cross-region
- **Events**: S3 → SNS, SQS, Lambda for event-driven processing

## Chapter 80: AWS — Lambda

### Introduction

AWS Lambda is a serverless compute service that runs code in response to events and automatically manages the underlying compute resources.

### Lambda Function

```javascript
exports.handler = async (event, context) => {
  console.log("Event:", JSON.stringify(event, null, 2));

  const { httpMethod, path, queryStringParameters, body } = event;

  if (httpMethod === "GET" && path === "/users") {
    const users = await db.listUsers(queryStringParameters);
    return {
      statusCode: 200,
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(users),
    };
  }

  if (httpMethod === "POST" && path === "/users") {
    const user = await db.createUser(JSON.parse(body));
    return {
      statusCode: 201,
      body: JSON.stringify(user),
    };
  }

  return {
    statusCode: 404,
    body: JSON.stringify({ error: "Not found" }),
  };
};
```

### Lambda Triggers

| Trigger | Service |
|---------|---------|
| API Gateway | HTTP/REST API |
| S3 | Bucket events (create/delete objects) |
| DynamoDB Streams | Table changes |
| SQS | Queue messages |
| SNS | Topic notifications |
| CloudWatch Events | Scheduled events (cron) |
| EventBridge | Event bus |

### Lambda Best Practices

- Keep function cold start under 1 second
- Use environment variables for configuration
- Set appropriate memory (more memory = more CPU)
- Use provisioned concurrency for latency-sensitive workloads
- Keep deployment package small (< 50MB ideal)
- Use AWS SDK v3 for modular imports

> ⚠️ **Warning:** Lambda cold starts can take several seconds, especially for functions with large deployment packages, VPC configuration, or Java/.NET runtimes. For latency-sensitive APIs, use Provisioned Concurrency or keep the deployment package under 10MB.

> ⚠️ **Warning:** Configuring a Lambda function inside a VPC without a NAT Gateway or VPC Endpoint means it cannot access the public internet (including DynamoDB, S3, or external APIs). Use VPC Endpoints for AWS services or place a NAT Gateway in your VPC.

> ✅ **Best Practice:** Right-size your Lambda memory. Since CPU scales proportionally with memory (up to 10GB/6 vCPUs), increasing memory often reduces execution time — and you may end up paying less due to the shorter duration.

> ✅ **Best Practice:** Use Powertools for AWS Lambda (available in Python, TypeScript, Java, .NET) to add structured logging, tracing with X-Ray, metrics, and batch processing without boilerplate code.

> ❓ **Interview Question:** What happens when a Lambda function scales to thousands of concurrent executions?
> **Answer:** Lambda has a regional concurrency limit (default 1000, adjustable via quota request). If the limit is reached, invocations are throttled and return a 429 error. Use Reserved Concurrency to guarantee capacity for critical functions.

> 💡 **Pro Tip:** For Lambda functions that call other AWS services, use the AWS SDK's `keepAlive` option to reuse TCP connections. This can reduce latency by 20-50% for successive invocations reusing the same execution context.

### Real World Analogy

Lambda functions are like food trucks at a festival. They only exist when someone places an order (request). **Cold starts** are the time to fire up the grill when the truck first arrives. **Provisioned Concurrency** is having the grill already hot before the crowd arrives. **Memory/CPU** is the size of the cooking surface — more space lets you cook faster. **Execution environment** is the truck itself, which stays warm for a while between orders.

### Practical Exercises

1. Create a Lambda function triggered by API Gateway that reads and writes to DynamoDB
2. Configure a Lambda function inside a VPC with VPC Endpoints for S3 access
3. Use Lambda Layers to share a common utility library across multiple functions
4. Set up Reserved Concurrency for a production function and test throttling behavior
5. Instrument a Lambda function with AWS X-Ray and visualize the service map

### Revision Notes

- **Lambda**: Serverless compute — pay per request and duration (rounded to 1ms)
- **Triggers**: API Gateway, S3, DynamoDB Streams, SQS, SNS, EventBridge, CloudWatch Events
- **Cold start**: ~100ms-1s for Node/Python, >1s for Java/.NET; mitigated by Provisioned Concurrency
- **Execution context**: Reused for ~5-15 minutes between invocations — use for DB connection pooling
- **Limits**: 10GB memory, 15-min timeout, 50MB deployment package (direct), 250MB (with layers)
- **VPC**: Functions in VPC lose internet access unless NAT Gateway or VPC Endpoints are configured

## Chapter 81: AWS — IAM

### Introduction

AWS Identity and Access Management (IAM) enables you to manage access to AWS services and resources securely.

### IAM Components

| Component | Description |
|-----------|-------------|
| User | Individual person or service account |
| Group | Collection of users with shared permissions |
| Role | Set of permissions assumed by users/services |
| Policy | JSON document defining permissions |
| Policy Document | Grant or deny access to specific resources |

### IAM Policy Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-app-bucket",
        "arn:aws:s3:::my-app-bucket/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:us-east-1:123456789012:table/users"
    }
  ]
}
```

### IAM Best Practices

- Follow principle of least privilege
- Use IAM roles instead of long-term access keys
- Use groups to manage permissions
- Enable MFA for root and privileged users
- Use conditions to restrict access (IP, MFA, time)
- Rotate access keys regularly
- Use IAM Access Analyzer to identify unused permissions

> ⚠️ **Warning:** Never use root account credentials for daily operations. The root user has unrestricted access to all resources and billing. Enable MFA on the root account and create administrative users with limited privileges instead.

> ⚠️ **Warning:** Hardcoded AWS access keys in code, configuration files, or environment variables are a frequent source of credential leaks. Use IAM Roles for EC2, Lambda, and ECS; use environment variables only with Secrets Manager references.

> ✅ **Best Practice:** Use IAM Roles (not Users) for applications and services. Roles provide temporary credentials via the AWS STS service, which automatically rotates every hour, eliminating long-term access key management.

> ✅ **Best Practice:** Implement least privilege using IAM Access Analyzer. It generates policies based on actual API usage, making it easy to create granular permissions instead of starting with overly permissive wildcard policies.

> ❓ **Interview Question:** What is the difference between an IAM Role and an IAM User?
> **Answer:** An IAM User is a permanent identity associated with a person or service with long-term credentials. An IAM Role is a temporary identity assumed by users, applications, or services, providing short-term credentials via AWS STS that automatically expire.

> 💡 **Pro Tip:** Use permission boundaries to delegate administration. A permission boundary sets the maximum permissions a user/role can have, allowing you to delegate IAM management while preventing privilege escalation.

### Real World Analogy

IAM is like a secure office building with multiple layers of access control. **Users** are individual employees with ID badges. **Groups** are departments (Engineering, HR, Finance) — everyone in the department gets the same base access. **Roles** are visitor badges or project-specific credentials — temporary and specific. **Policies** are the access control list on each door. **MFA** is the fingerprint scanner in addition to the ID badge. **Conditions** restrict access to business hours or specific entry points.

### Practical Exercises

1. Create an IAM user in a group with read-only S3 access and verify by listing buckets
2. Create an IAM role for an EC2 instance with DynamoDB read access and attach it to a running instance
3. Use IAM Access Analyzer to generate a least-privilege policy from CloudTrail logs
4. Configure a permission boundary that prevents users from creating IAM admin roles
5. Set up a cross-account IAM role that allows a user from Account A to access an S3 bucket in Account B

### Revision Notes

- **IAM**: Identity and Access Management — who can do what to which resources
- **User**: Long-term identity for people or service accounts
- **Group**: Collection of users with shared permissions
- **Role**: Temporary identity for apps/services assuming permissions via STS
- **Policy**: JSON document defining allowed/denied actions on resources
- **Conditions**: Restrict by IP, MFA status, time, SSL, VPC endpoint
- **Best practices**: Least privilege, roles over keys, MFA, Access Analyzer, permission boundaries

## Chapter 82: AWS — CloudFront, RDS, VPC, ELB, Auto Scaling, Route53

### CloudFront (CDN)

- **Origin**: S3, ALB, EC2, Custom HTTP
- **Edge Locations**: 600+ points of presence
- **Caching**: TTL configurable per path pattern
- **Security**: AWS Shield, WAF integration
- **Signed URLs/Cookies**: Restrict access to premium content

### RDS (Relational Database Service)

| Engine | Use Case |
|--------|----------|
| Aurora MySQL/PostgreSQL | High performance, serverless option |
| MySQL | Web applications |
| PostgreSQL | Advanced features, JSONB, GIS |
| MariaDB | MySQL-compatible, open-source |
| SQL Server | .NET applications |
| Oracle | Enterprise applications |

- **Multi-AZ**: Synchronous standby replica for failover
- **Read Replicas**: Asynchronous replicas for read scaling (up to 15)
- **Aurora**: 5x faster than MySQL, 3x faster than PostgreSQL

### VPC (Virtual Private Cloud)

```bash
# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Create subnets (public and private)
aws ec2 create-subnet --vpc-id vpc-123 --cidr-block 10.0.1.0/24
aws ec2 create-subnet --vpc-id vpc-123 --cidr-block 10.0.2.0/24

# Internet Gateway (for public subnets)
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --vpc-id vpc-123 --internet-gateway-id igw-123

# NAT Gateway (for private subnets to access internet)
aws ec2 create-nat-gateway --subnet-id subnet-public --allocation-id eip-123

# Security Group (stateful firewall)
aws ec2 create-security-group --group-name web-sg --description "Web security group"
aws ec2 authorize-security-group-ingress --group-id sg-123 --protocol tcp --port 443 --cidr 0.0.0.0/0

# VPC Peering
aws ec2 create-vpc-peering-connection --vpc-id vpc-111 --peer-vpc-id vpc-222
```

### ELB (Elastic Load Balancing)

| Type | Layer | Use Case |
|------|-------|----------|
| ALB (Application) | Layer 7 | HTTP/HTTPS, path-based routing, host-based routing |
| NLB (Network) | Layer 4 | TCP/UDP, extreme performance, static IP |
| CLB (Classic) | Layer 4/7 | Legacy, not recommended for new apps |

### Auto Scaling

```json
{
  "AutoScalingGroupName": "web-asg",
  "LaunchTemplate": { "LaunchTemplateName": "web-template", "Version": "1" },
  "MinSize": 2,
  "MaxSize": 10,
  "DesiredCapacity": 3,
  "VPCZoneIdentifier": "subnet-1,subnet-2,subnet-3",
  "TargetGroupARNs": ["arn:aws:elasticloadbalancing:..."]
}
```

### Route53 (DNS)

| Record Type | Use Case |
|-------------|----------|
| A | Map domain to IPv4 address |
| AAAA | Map domain to IPv6 address |
| CNAME | Map domain to another domain |
| ALIAS | Map domain to AWS resource (free, unlike CNAME) |
| MX | Mail exchange records |
| TXT | Text records (verification, SPF) |

- **Routing Policies**: Simple, Weighted, Latency, Failover, Geolocation, Geoproximity, IP-based
- **Health Checks**: Monitor endpoint health and route traffic away from unhealthy endpoints

> ⚠️ **Warning:** Using a single NAT Gateway in a single AZ creates a single point of failure. If that AZ goes down, all private instances lose internet access. Use NAT Gateways in at least two AZs for production workloads.

> ⚠️ **Warning:** Default VPC security groups allow all outbound traffic. If you need to restrict data exfiltration, create custom security groups with outbound rules that only allow traffic to specific destinations.

> ✅ **Best Practice:** Use ALB for HTTP/HTTPS workloads — it supports path-based routing, host-based routing, SSL termination, WebSocket, and integrates with WAF, Cognito, and Lambda as targets.

> ✅ **Best Practice:** For Route53, always use ALIAS records instead of CNAME when pointing to AWS resources (CloudFront, ELB, S3). ALIAS records work at the zone apex and are free, while CNAME records cannot be used at the root domain.

> ❓ **Interview Question:** What is the difference between a Security Group and a Network ACL (NACL)?
> **Answer:** Security Groups are stateful firewalls at the instance level supporting allow rules only (return traffic is automatically allowed). NACLs are stateless at the subnet level supporting both allow and deny rules with numbered priority ordering.

> 💡 **Pro Tip:** Use CloudFront Origin Shield to create an additional caching layer that reduces load on your origin. This is especially beneficial for origins with high request rates or expensive compute.

### Real World Analogy

Think of AWS networking as a gated community. **VPC** is the community perimeter wall. **Subnets** are the neighborhoods (public = houses with street-facing doors, private = houses inside security gates). **Internet Gateway** is the main gate with direct road access. **NAT Gateway** is the security gate that lets private houses access the outside world. **Security Groups** are the individual house door locks. **NACLs** are the neighborhood checkpoints. **Route53** is the GPS navigation system mapping street names to house numbers.

### Practical Exercises

1. Create a VPC with public and private subnets across two AZs, an Internet Gateway, and NAT Gateways
2. Deploy an Application Load Balancer with path-based routing (`/api/*` → target group A, `/*` → target group B)
3. Set up CloudFront with an S3 origin and a custom domain using Route53 alias records
4. Create an RDS MySQL instance with Multi-AZ enabled and configure a read replica
5. Configure Route53 latency-based routing to direct traffic to the nearest region

### Revision Notes

- **CloudFront**: CDN, 600+ edge locations, origin shielding, WAF integration, signed URLs
- **RDS**: Managed databases — Multi-AZ (HA), Read Replicas (scale reads), Aurora (high perf)
- **VPC**: Virtual network — subnets, IGW, NAT Gateway, VPC Peering, Transit Gateway, VPN
- **ELB**: ALB (HTTP/HTTPS, Layer 7), NLB (TCP/UDP, Layer 4, static IP), CLB (legacy)
- **Auto Scaling**: EC2 Auto Scaling + Application Auto Scaling for dynamic capacity
- **Route53**: DNS — routing policies (Simple, Weighted, Latency, Failover, Geolocation, Geoproximity)

---

## Chapter 83: GCP — Compute Engine, Cloud Storage, Cloud Functions, IAM, Cloud SQL, VPC

### Introduction

Google Cloud Platform (GCP) offers infrastructure and platform services with strengths in data analytics, machine learning, and containerized workloads (being the home of Kubernetes).

### Compute Engine (VMs)

```bash
# Create VM
gcloud compute instances create my-instance \
  --zone us-central1-a \
  --machine-type e2-medium \
  --image-family ubuntu-2204-lts \
  --image-project ubuntu-os-cloud \
  --boot-disk-size 50GB \
  --tags http-server,https-server

# SSH
gcloud compute ssh my-instance --zone us-central1-a

# List instances
gcloud compute instances list

# Stop/start
gcloud compute instances stop my-instance
gcloud compute instances start my-instance

# Create instance group (managed)
gcloud compute instance-groups managed create my-mig \
  --base-instance-name my-instance \
  --size 3 \
  --template my-instance-template \
  --zone us-central1-a
```

### Cloud Storage (Object Storage)

- **Storage Classes**: Standard, Nearline, Coldline, Archive
- **Features**: Object versioning, lifecycle management, object holds
- **Consistency**: Strong consistency for all operations
- **Access Control**: IAM, ACLs, signed URLs, uniform bucket-level access

```bash
gsutil mb gs://my-unique-bucket
gsutil cp myfile.txt gs://my-bucket/
gsutil rsync -r ./dist gs://my-bucket/
gsutil ls gs://my-bucket/
gsutil signurl -d 1h key.json gs://my-bucket/private-file.pdf
gsutil lifecycle set lifecycle-config.json gs://my-bucket/
```

### Cloud Functions (Serverless)

```javascript
exports.helloWorld = (req, res) => {
  res.send('Hello, World!');
};

exports.processFile = async (event, context) => {
  const file = event;
  console.log(`Processing: ${file.name}`);
  await processFileContents(file);
};
```

```bash
gcloud functions deploy hello-world \
  --runtime nodejs20 \
  --trigger-http \
  --allow-unauthenticated \
  --entry-point helloWorld

gcloud functions deploy process-file \
  --runtime nodejs20 \
  --trigger-bucket my-input-bucket \
  --entry-point processFile
```

### IAM on GCP

- **Roles**: Basic (Owner/Editor/Viewer), Predefined, Custom
- **Policy**: Bind members to roles at resource hierarchy
- **Service Accounts**: Identity for applications and VMs
- **Resource Hierarchy**: Organization -> Folder -> Project -> Resource

```bash
gcloud projects add-iam-policy-binding my-project \
  --member="user:alice@example.com" \
  --role="roles/storage.objectViewer"

gcloud iam service-accounts create my-app-sa \
  --display-name="My App Service Account"

gcloud iam service-accounts keys create key.json \
  --iam-account=my-app-sa@my-project.iam.gserviceaccount.com
```

### Cloud SQL

```bash
# Create PostgreSQL instance
gcloud sql instances create my-postgres \
  --database-version POSTGRES_16 \
  --tier db-custom-2-7680 \
  --region us-central1 \
  --storage-size 100GB \
  --backup-start-time 02:00

# Create database
gcloud sql databases create mydb --instance my-postgres

# Connect via Cloud SQL Auth Proxy
gcloud sql connect my-postgres --user=postgres

# Configure high availability
gcloud sql instances patch my-postgres --availability-type REGIONAL
```

### VPC

```bash
gcloud compute networks create my-vpc --subnet-mode custom

gcloud compute networks subnets create my-subnet-us \
  --network my-vpc \
  --region us-central1 \
  --range 10.0.1.0/24

gcloud compute networks subnets create my-subnet-eu \
  --network my-vpc \
  --region europe-west1 \
  --range 10.0.2.0/24

gcloud compute firewall-rules create allow-http \
  --network my-vpc \
  --allow tcp:80,tcp:443 \
  --source-ranges 0.0.0.0/0

gcloud compute vpn-gateways create my-vpn-gateway \
  --network my-vpc \
  --region us-central1
```

> ⚠️ **Warning:** GCP enforces default VPC firewall rules that allow SSH (port 22), RDP (port 3389), and ICMP from `0.0.0.0/0`. Disable or restrict these defaults in production environments.

> ⚠️ **Warning:** Cloud Functions in GCP have a 9-minute timeout limit (540 seconds) for HTTP-triggered functions and 60 minutes for event-driven functions. Long-running operations should use Cloud Run or GKE instead.

> ✅ **Best Practice:** Use GCP's uniform bucket-level access for Cloud Storage instead of ACLs. This disables per-object ACLs and forces all access control through IAM, reducing complexity and security gaps.

> ✅ **Best Practice:** Use Cloud SQL Auth Proxy instead of whitelisting IP addresses for database access. The proxy encrypts connections and handles IAM-based authentication, eliminating the need to manage firewall rules.

> ❓ **Interview Question:** How does GCP's resource hierarchy differ from AWS?
> **Answer:** GCP uses a hierarchy of Organization → Folder → Project → Resource. Policies and IAM permissions are inherited downward. AWS uses a flatter structure with accounts as the main boundary and Organizations for multi-account management.

> 💡 **Pro Tip:** Use `gcloud compute images list` to find the latest public images from various OS vendors. Filter with `--filter="family=ubuntu-2204-lts"` and sort by `--sort-by=~creationTimestamp` to get the most recent.

### Real World Analogy

GCP is like a city built by urban planners (Google) who designed everything from the ground up. **Compute Engine** is traditional housing. **Cloud Functions** are pop-up shops that appear when needed. **Cloud Storage** is centralized warehousing. **Cloud SQL** is managed plumbing — you don't worry about pipe maintenance. **VPC** is the road network. **IAM** is the city permit system. GCP's strong suit is its data analytics and ML services, like having the best public transportation system.

### Practical Exercises

1. Create a Compute Engine VM with a startup script that installs Nginx and hosts a custom web page
2. Set up Cloud Storage with uniform bucket-level access and IAM-based permissions for a service account
3. Deploy a Cloud Function triggered by Cloud Storage upload that resizes images using ImageMagick
4. Create a Cloud SQL PostgreSQL instance and connect using the Cloud SQL Auth Proxy
5. Configure a VPC with custom subnet mode, firewall rules, and a Cloud VPN gateway

### Revision Notes

- **Compute Engine**: VMs with various machine families (E2, N2, C2, M1, A2)
- **Cloud Storage**: Object storage with strong consistency — Standard, Nearline, Coldline, Archive
- **Cloud Functions**: Serverless, max 9-min timeout (HTTP), 60-min (background)
- **IAM**: Roles at Organization → Folder → Project → Resource hierarchy
- **Cloud SQL**: Managed MySQL, PostgreSQL, SQL Server — with Auth Proxy for secure access
- **VPC**: Global networking, custom subnet mode, firewall rules, Cloud VPN / Interconnect

---

## Chapter 84: Azure — VMs, Blob Storage, Functions, Entra ID, SQL Database, VNet

### Introduction

Microsoft Azure is a comprehensive cloud platform with deep integration with Microsoft products (Windows, Active Directory, SQL Server, .NET).

### Virtual Machines

```bash
# Create VM
az vm create \
  --resource-group my-rg \
  --name my-vm \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-sku Standard

# SSH
ssh azureuser@<public-ip>

# List VMs
az vm list --output table

# Stop/start/deallocate
az vm stop --resource-group my-rg --name my-vm
az vm start --resource-group my-rg --name my-vm
```

### Blob Storage (Object Storage)

| Access Tier | Use Case | Retrieval Cost | Storage Cost |
|-------------|----------|----------------|--------------|
| Hot | Frequently accessed | Low | High |
| Cool | Infrequent access (30+ days) | Medium | Medium |
| Cold | Rare access (90+ days) | High | Low |
| Archive | Backup (180+ days) | Highest | Lowest |

```bash
# Create storage account and container
az storage account create --name mystorageaccount --resource-group my-rg --sku Standard_GRS
az storage container create --name my-container --account-name mystorageaccount

# Upload/download
az storage blob upload --account-name mystorageaccount --container-name my-container --name file.txt --file ./file.txt
az storage blob download --account-name mystorageaccount --container-name my-container --name file.txt --file ./downloaded.txt

# Generate SAS token
az storage blob generate-sas --account-name mystorageaccount --container-name my-container --name file.txt --permissions r --expiry 2026-08-01
```

### Azure Functions (Serverless)

```javascript
module.exports = async function (context, req) {
  context.log('HTTP function processed a request');

  const name = req.query.name || (req.body && req.body.name);

  context.res = {
    status: 200,
    body: `Hello, ${name || "World"}!`,
  };
};
```

```bash
# Create function app
az functionapp create \
  --resource-group my-rg \
  --consumption-plan-location eastus \
  --runtime node \
  --runtime-version 20 \
  --functions-version 4 \
  --name my-function-app \
  --storage-account mystorageaccount
```

### Entra ID (formerly Azure AD)

- **Identity**: Users, Groups, Service Principals, Managed Identities
- **Authentication**: OAuth 2.0, OpenID Connect, SAML
- **Conditional Access**: Context-aware access policies
- **RBAC**: Role-based access control for Azure resources

```bash
# Create user
az ad user create --display-name "Alice Smith" --user-principal-name alice@domain.com --password "P@ssw0rd!"

# Create service principal
az ad sp create-for-rbac --name my-app-sp --role Contributor --scopes /subscriptions/{sub-id}/resourceGroups/my-rg

# Assign role
az role assignment create --assignee alice@domain.com --role "Contributor" --resource-group my-rg
```

### Azure SQL Database

```bash
# Create SQL Server
az sql server create --name my-sql-server --resource-group my-rg --admin-user adminuser --admin-password "P@ssw0rd!"

# Create database
az sql db create --resource-group my-rg --server my-sql-server --name mydb --service-objective S2

# Firewall rule
az sql server firewall-rule create --resource-group my-rg --server my-sql-server --name AllowMyIP --start-ip-address 203.0.113.0 --end-ip-address 203.0.113.255
```

### VNet (Virtual Network)

```bash
# Create VNet with subnet
az network vnet create \
  --resource-group my-rg \
  --name my-vnet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name my-subnet \
  --subnet-prefix 10.0.1.0/24

# Create additional subnet
az network vnet subnet create --resource-group my-rg --vnet-name my-vnet --name private-subnet --address-prefix 10.0.2.0/24

# Network Security Group
az network nsg create --resource-group my-rg --name my-nsg
az network nsg rule create --resource-group my-rg --nsg-name my-nsg --name AllowHTTP --protocol tcp --destination-port-ranges 80 --access allow --priority 100

# VNet Peering
az network vnet peering create --resource-group my-rg --name my-vnet-to-other --vnet-name my-vnet --remote-vnet /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Network/virtualNetworks/other-vnet --allow-vnet-access
```

> ⚠️ **Warning:** Azure VM disks have an 8GB temp drive (`/mnt/resource` on Linux) that is NOT persistent — data is lost on deallocation or reboot. Never store important data on the temp drive; always use managed disks for persistent storage.

> ⚠️ **Warning:** Blob Storage with Hot access tier is more expensive than Cool for infrequently accessed data. Always match the access tier to the expected access pattern to avoid unexpected cost spikes.

> ✅ **Best Practice:** Use Managed Identities instead of service principals for Azure resources that need to authenticate to other Azure services. Managed Identities are automatically managed and eliminate credential rotation overhead.

> ✅ **Best Practice:** Use Azure Blueprints or Bicep (instead of ARM JSON) for resource organization. Bicep provides a cleaner, modular syntax for Infrastructure as Code compared to raw ARM templates.

> ❓ **Interview Question:** What is the difference between a managed disk and an unmanaged disk in Azure?
> **Answer:** Managed disks are automatically managed by Azure — you specify size and performance tier, and Azure handles storage account placement, replication, and scaling. Unmanaged disks require you to manually create and manage storage accounts and stay within storage account limits.

> 💡 **Pro Tip:** Use `az vm list-sizes --location eastus` to explore available VM sizes in a region. Filter by memory, vCPUs, or pricing tier to find the optimal instance type for your workload.

### Real World Analogy

Azure is like a large enterprise campus with deep integration across all buildings. **Virtual Machines** are individual offices. **Blob Storage** is the centralized document archive with sections based on access frequency. **Azure Functions** are meeting rooms that are only booked when needed. **Entra ID** is the corporate badge system integrated with every door. **VNet** is the internal road network connecting all buildings. Azure shines when your organization already uses Microsoft products — it's like having a campus with perfectly connected buildings.

### Practical Exercises

1. Create an Azure VM with a custom managed image, attach a data disk, and configure NSG rules
2. Set up Blob Storage with lifecycle management moving blobs from Hot → Cool → Archive
3. Deploy an Azure Function with an HTTP trigger and connect it to Cosmos DB using a managed identity
4. Create a VNet with two subnets, an NSG, and peer it with another VNet
5. Configure Entra ID (Azure AD) for SSO with an app registration and conditional access policy

### Revision Notes

- **Virtual Machines**: IaaS with managed disks, availability sets/zones, VM scale sets
- **Blob Storage**: Object storage — Hot (frequent), Cool (infrequent), Cold (rare), Archive (backup)
- **Functions**: Serverless — supports HTTP, Blob, Queue, Event Grid, Timer triggers
- **Entra ID**: Identity and access — OAuth 2.0, OpenID Connect, Conditional Access, RBAC
- **SQL Database**: PaaS database — elastic pools, geo-replication, serverless compute tier
- **VNet**: Virtual network — subnets, NSGs, VNet peering, VPN Gateway, ExpressRoute

---

## Chapter 85: Cloud Architecture Patterns

### Introduction

Cloud architecture patterns are reusable solutions to common problems in cloud application design. They address scalability, resilience, security, and cost.

### Key Patterns

#### 1. Microservices Architecture

Small, independent services that communicate via APIs. Each service owns its data and can be deployed independently.

**Pros**: Independent scaling, technology diversity, team autonomy.
**Cons**: Distributed complexity, network latency, data consistency challenges.

#### 2. Event-Driven Architecture

Services communicate through events (messages) rather than direct API calls. Uses message brokers (Kafka, SQS, EventBridge).

**Pros**: Loose coupling, scalability, asynchronous processing.
**Cons**: Eventual consistency, debugging complexity, message ordering.

#### 3. Strangler Fig Pattern

Gradually replace monolithic components with microservices. Route traffic to new services incrementally.

#### 4. Circuit Breaker

Prevents cascading failures by detecting when a downstream service is failing and opening the circuit to fail fast.

```javascript
const breaker = new CircuitBreaker(asyncFunction, {
  timeout: 5000,
  errorThresholdPercentage: 50,
  resetTimeout: 30000,
});

try {
  const result = await breaker.fire(request);
} catch (error) {
  // Fallback logic
  return fallbackResponse();
}
```

#### 5. CQRS (Command Query Responsibility Segregation)

Separate read models from write models. Optimize each for its purpose.

#### 6. Saga Pattern

Manage distributed transactions across microservices using a sequence of local transactions with compensating actions.

- **Choreography**: Each service publishes events and listens for events
- **Orchestration**: A central coordinator tells services what to do

#### 7. Backend for Frontend (BFF)

Create separate API gateways for each client type (web, mobile, IoT) to tailor responses to each client's needs.

#### 8. Sidecar Pattern

Deploy helper components (logging, monitoring, proxy) alongside the main application in the same pod/VM.

### Architecture Decision Framework

| Concern | Consideration |
|---------|--------------|
| Scalability | Vertical vs horizontal, stateless design, caching strategy |
| Resilience | Redundancy, circuit breakers, retries, graceful degradation |
| Security | Defense in depth, least privilege, encryption at rest/transit |
| Observability | Logging, metrics, tracing, alerting |
| Cost | Right-sizing, reserved instances, spot instances, auto-scaling |
| Data | Consistency vs availability (CAP theorem), backup strategy |

### CAP Theorem

In a distributed system, you can only guarantee two of three properties:
- **Consistency**: All nodes see the same data at the same time
- **Availability**: Every request receives a response
- **Partition Tolerance**: System continues despite network failures

> ⚠️ **Warning:** Microservices don't automatically solve scalability problems — they introduce distributed complexity, network latency, data consistency challenges, and operational overhead. Start with a modular monolith and extract services only when needed.

> ⚠️ **Warning:** Event-driven architectures can lead to eventual consistency issues. If your application requires strict consistency (e.g., financial transactions), ensure compensating transactions or saga patterns are in place.

> ✅ **Best Practice:** Implement Circuit Breaker patterns with a fallback strategy. When a downstream service fails, return cached data, a default response, or gracefully degrade functionality instead of propagating the error to the user.

> ✅ **Best Practice:** Use the Strangler Fig pattern for incremental migrations from monolith to microservices. Route specific URL paths to new services while keeping the rest on the monolith, gradually shifting traffic until the monolith can be decommissioned.

> ❓ **Interview Question:** Explain the CAP theorem with examples of which two properties each database type prioritizes.
> **Answer:** Traditional RDBMS (PostgreSQL, MySQL) prioritize **Consistency + Availability** (CA). Cassandra and DynamoDB prioritize **Availability + Partition Tolerance** (AP). HBase and MongoDB (with default settings) prioritize **Consistency + Partition Tolerance** (CP). No system can guarantee all three simultaneously.

> 💡 **Pro Tip:** Start with the "2-pizza team" rule for microservices — each service should be small enough that a team of 6-8 people can own it end-to-end. If a service requires more than one team to maintain, it's too large.

### Real World Analogy

Architecture patterns are like different types of building construction. **Microservices** are a campus of separate buildings, each with its own purpose and maintenance team. **Event-Driven Architecture** is like a postal service — buildings send letters (events) via mail (message broker) and react to what they receive. **Circuit Breaker** is a circuit breaker box — when a short occurs (service fails), it cuts power to prevent a building fire (cascading failure). **CQRS** is having separate entrances for deliveries (writes) and visitors (reads).

### Practical Exercises

1. Design a circuit breaker for a service that calls an external payment gateway, with a fallback to cached data
2. Implement a simple event-driven flow using SQS (or EventBridge) where Order Service emits events consumed by Inventory and Notification services
3. Refactor a monolith using the Strangler Fig pattern — identify one module to extract as a separate service
4. Design a CQRS pattern with separate read and write databases, using DynamoDB Streams to sync them
5. Implement a Saga pattern using Step Functions for a multi-step order processing workflow

### Revision Notes

- **Microservices**: Independent services, own data, independent deployability
- **Event-Driven**: Loose coupling via message brokers (Kafka, SQS, EventBridge)
- **Strangler Fig**: Incremental monolith → microservices migration
- **Circuit Breaker**: Fail fast, avoid cascading failures, fallback to degraded mode
- **CQRS**: Separate read/write models for optimized performance
- **Saga**: Distributed transactions via choreography (events) or orchestration (central coordinator)
- **CAP**: Consistency, Availability, Partition Tolerance — pick two

---

## Chapter 86: Multi-Cloud Strategy

### Introduction

Multi-cloud uses services from multiple cloud providers to avoid vendor lock-in, improve resilience, and leverage best-of-breed services.

### Approaches

| Approach | Description | Complexity | Benefit |
|----------|-------------|------------|---------|
| Active-Active | Workloads run in multiple clouds simultaneously | Very High | Max resilience, no failover time |
| Active-Passive | Primary cloud, secondary on standby | High | DR with failover |
| Service-specific | Use best service per cloud (e.g., GCP for ML, AWS for compute) | Medium | Best-of-breed |
| Migration | Transitioning from one cloud to another | Medium | Vendor flexibility |

### Multi-Cloud Challenges

- **Network latency**: Data transfer between clouds is slow and expensive
- **Skill set**: Each cloud has different APIs, CLIs, and management consoles
- **Security**: Consistent security policies across clouds is difficult
- **Data gravity**: Moving large datasets is costly
- **Tooling**: Few tools work seamlessly across all major clouds

### Tools for Multi-Cloud

- **Terraform**: Infrastructure as Code across all major clouds
- **Kubernetes**: Container orchestration that works anywhere
- **Crossplane**: Control plane for multi-cloud resource management
- **Pulumi**: Infrastructure as Code with real programming languages

> ⚠️ **Warning:** Multi-cloud adds significant complexity in networking, security, compliance, and operations. For most organizations, the benefits of multi-cloud do not outweigh the costs unless there is a specific need (avoiding lock-in, regulatory requirements, M&A).

> ⚠️ **Warning:** Data transfer between clouds is expensive (standard inter-cloud egress rates apply on both sides). Ensure your architecture minimizes cross-cloud traffic, or consider a single-cloud strategy with disaster recovery replication.

> ✅ **Best Practice:** Use an abstraction layer (Terraform, Kubernetes, Crossplane) to maintain cloud-agnostic infrastructure definitions. This reduces vendor lock-in and makes migration easier if a provider's pricing or capabilities change.

> ✅ **Best Practice:** Start with a single-cloud strategy and expand to multi-cloud only when you have clear requirements. Many successful companies run entirely on one cloud provider with a well-architected disaster recovery plan.

> ❓ **Interview Question:** When would you choose a multi-cloud strategy over a single-cloud strategy?
> **Answer:** Multi-cloud is beneficial for avoiding vendor lock-in, leveraging best-of-breed services (e.g., GCP for ML, AWS for compute), meeting regulatory data residency requirements, or providing additional disaster recovery resilience. Single-cloud is simpler, cheaper, and operationally easier.

> 💡 **Pro Tip:** For multi-cloud networking, use a cloud-agnostic service mesh like Istio (on Kubernetes) or HashiCorp Consul. These provide service discovery, traffic management, and mTLS encryption across clusters in different clouds.

### Real World Analogy

Multi-cloud is like having homes in multiple cities instead of one home. You have flexibility (escape a natural disaster), but you pay for multiple properties, need to maintain each one, and deal with different local laws (APIs, compliance). Most families are better off with one well-built home (single cloud) and a good insurance policy (backup/recovery plan) than trying to maintain homes in three cities.

### Practical Exercises

1. Use Terraform to provision identical infrastructure (VPC, Compute Instance, Storage) on AWS and GCP
2. Deploy a Kubernetes cluster on two different cloud providers and connect them with a service mesh
3. Design a multi-cloud disaster recovery plan with active-passive failover
4. Calculate the cost of transferring 10TB of data between AWS and GCP per month
5. Implement a cross-cloud identity federation using OIDC between AWS IAM and GCP IAM

### Revision Notes

- **Approaches**: Active-Active, Active-Passive, Service-specific, Migration
- **Challenges**: Network latency/cost, skill requirements, security consistency, data gravity
- **Abstraction tools**: Terraform, Kubernetes, Crossplane, Pulumi
- **Best suited for**: Avoiding lock-in, best-of-breed services, regulatory requirements
- **Not suited for**: Simple applications, small teams, cost-sensitive projects

---

## Chapter 87: Cloud Cost Optimization

### Introduction

Cloud cost optimization is the practice of reducing cloud spending while maintaining performance, security, and reliability.

### Cost Optimization Pillars (AWS Well-Architected)

| Pillar | Strategy | Example |
|--------|----------|---------|
| Right-sizing | Match instance size to workload needs | Replace over-provisioned instances |
| Elasticity | Scale resources based on demand | Auto Scaling, serverless |
| Pricing models | Choose best pricing for usage patterns | Reserved, Savings Plans, Spot |
| Storage optimization | Use appropriate storage tiers | Lifecycle policies, delete unused |
| Data transfer | Minimize cross-region/cloud data transfer | Use CDN, same-region services |

### Pricing Models

| Model | Discount | Commitment | Best For |
|-------|----------|------------|----------|
| On-Demand | None | None | Short-term, unpredictable workloads |
| Reserved Instances | 30-60% | 1-3 years | Steady-state workloads |
| Savings Plans | 30-60% | 1-3 years (compute $/hr) | Flexible instance families |
| Spot Instances | 60-90% | None | Fault-tolerant, batch, stateless |

### Cost Optimization Checklist

- [ ] Right-size underutilized resources (CloudWatch, Trusted Advisor)
- [ ] Use auto-scaling to match demand
- [ ] Purchase Reserved Instances or Savings Plans for steady-state
- [ ] Use Spot Instances for fault-tolerant workloads
- [ ] Set up billing alerts and budgets
- [ ] Delete unused resources (unattached EBS volumes, idle load balancers, Elastic IPs)
- [ ] Use S3 lifecycle policies to move data to cheaper tiers
- [ ] Enable data compression to reduce storage and transfer costs
- [ ] Use CDN (CloudFront, Cloud CDN) to reduce origin server load
- [ ] Tag resources for cost allocation and tracking

```bash
# AWS Cost Explorer
aws ce get-cost-and-usage --time-period Start=2026-01-01,End=2026-07-04 --granularity MONTHLY --metrics "BlendedCost" --group-by Type=DIMENSION,Key=SERVICE

# Find unattached EBS volumes
aws ec2 describe-volumes --filters Name=status,Values=available --query "Volumes[*].{ID:VolumeId,Size:Size}" --output table

# Delete old snapshots
aws ec2 describe-snapshots --owner-ids self --query "Snapshots[?StartTime<='2025-01-01'].SnapshotId" --output text | xargs -n1 aws ec2 delete-snapshot --snapshot-id
```

### Common Cloud Mistakes

- Leaving unused resources running (dev/test instances over weekends)
- Not using auto-scaling (over-provisioned for peak load)
- Using on-demand when Reserved/Savings Plans would save 40%+
- Not leveraging Spot Instances for batch/non-critical workloads
- Storing infrequently accessed data in expensive storage tiers
- Transferring data across regions unnecessarily
- Over-provisioning database instances

> ⚠️ **Warning:** Unattached EBS volumes, idle load balancers, unused Elastic IPs, and stale snapshots accumulate costs silently. Run a monthly audit using AWS Trusted Advisor or Azure Advisor to identify and clean up orphaned resources.

> ⚠️ **Warning:** Data transfer costs (egress) are often the largest hidden cost in cloud bills. Data transfer between regions, between clouds, or to the internet is expensive. Cache content with CDNs and keep data processing in the same region.

> ✅ **Best Practice:** Implement FinOps — bring together engineering, finance, and product teams to take collective ownership of cloud costs. Use shared cost dashboards and make cost a factor in architectural decisions.

> ✅ **Best Practice:** For predictable workloads, purchase Reserved Instances (1-year or 3-year) or Compute Savings Plans. For variable workloads, use Auto Scaling with a mix of On-Demand and Spot Instances.

> ❓ **Interview Question:** What is the difference between vertical and horizontal scaling in the context of cost optimization?
> **Answer:** Vertical scaling (upgrading to a larger instance) has a cost ceiling — larger instances are disproportionately more expensive. Horizontal scaling (adding more instances) is more cost-effective because you can use smaller, cheaper instances and scale down during low traffic.

> 💡 **Pro Tip:** Use AWS Compute Optimizer or Azure Advisor to get right-sizing recommendations. These tools analyze utilization metrics and recommend instance type changes that could save 10-20% without performance impact.

### Real World Analogy

Cloud cost optimization is like managing a household budget. **On-Demand pricing** is buying everything at full retail price — convenient but expensive. **Reserved Instances** are buying in bulk at Costco — cheaper but requires commitment. **Spot Instances** are buying day-old bread at a discount — excellent when you're flexible. **Auto Scaling** is turning off lights and AC when rooms are empty. **Tagging** is labeling receipts so you know exactly where the money went.

### Practical Exercises

1. Use AWS Cost Explorer to identify the top 5 cost drivers in your account and create a monthly budget alert
2. Calculate potential savings by moving a steady-state workload from On-Demand to 1-year Reserved Instances
3. Set up a Lambda function that automatically stops non-production EC2 instances overnight and on weekends
4. Implement S3 lifecycle policies to move logs from Standard → Glacier after 30 days
5. Create a cost allocation report using resource tags and identify underutilized resources with Trusted Advisor

### Revision Notes

- **Pricing models**: On-Demand (flexible), Reserved (30-60% off, 1-3yr commitment), Spot (60-90% off, interruptible), Savings Plans (compute usage commitment)
- **Right-sizing**: Match instance size to actual utilization — use Compute Optimizer / Advisor
- **Storage optimization**: Lifecycle policies, delete orphaned volumes/snapshots, use correct tier
- **Auto Scaling**: Scale in/out based on demand — don't over-provision for peak
- **Data transfer**: Keep traffic in-region, use CDN, compress data
- **FinOps**: Cross-team cost accountability, tagging, budgets, regular reviews, anomaly detection

---

### Interview Questions (Cloud)

### Interview Questions (Cloud)

1. What is the difference between scalability and elasticity?
2. Explain the CAP theorem and its implications for distributed systems.
3. How would you design a highly available multi-region application?
4. Compare EC2, Lambda, and ECS for running a web application.
5. How do you secure an S3 bucket?
6. Explain blue-green deployment and canary releases.
7. What is the difference between a security group and a NACL?
8. How does CloudFront improve application performance?
9. Explain VPC peering, transit gateway, and VPN connections.
10. How do you estimate and optimize cloud costs?

### Revision Notes (Cloud)

- **EC2** = Virtual machines, choose right instance type and family
- **S3** = Object storage, 11 nines durability, lifecycle policies
- **Lambda** = Serverless compute, triggered by events
- **IAM** = Identity and access management, least privilege
- **VPC** = Virtual network, subnets, security groups, NAT gateway
- **ELB** = Load balancing, ALB (HTTP) vs NLB (TCP)
- **RDS** = Managed relational databases, Multi-AZ for HA, read replicas
- **CloudFront** = CDN, edge caching, DDoS protection
- **Route53** = DNS, routing policies, health checks
- **Auto Scaling** = Automatically adjust compute capacity

---

# End of Parts 5-9

---
