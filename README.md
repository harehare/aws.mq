<h1 align="center">aws.mq</h1>

An AWS API response processor implemented as an [mq](https://github.com/harehare/mq) module.

Process AWS CLI / SDK JSON responses with filtering, extraction, and Markdown table rendering — covering the most commonly used services.

## Features

- **EC2** — instances, security groups, VPCs, subnets (filter by state, type, VPC)
- **S3** — buckets, objects (filter by storage class, total size)
- **IAM** — users, roles, policies
- **Lambda** — functions (filter by runtime, state, total memory)
- **RDS** — DB instances (filter by engine, status)
- **CloudFormation** — stacks, outputs, parameters
- **SQS** — queue URLs, queue names, message counts
- **SNS** — topics, subscriptions (filter by protocol or topic)
- **ECS** — clusters, services, tasks (filter by status)
- **DynamoDB** — table names, table details
- **CloudWatch** — metric alarms (filter by state: OK / ALARM / INSUFFICIENT_DATA)
- **SSM** — parameters (filter by type: String / SecureString / StringList)
- **Secrets Manager** — secrets
- **ELBv2** — load balancers, target groups (filter by type / state)
- **Route53** — hosted zones (public / private), resource record sets (filter by type)

## Installation

Copy `aws.mq` to your mq module directory:

```sh
cp aws.mq ~/.mq/
```

### HTTP Import (no local installation needed)

If `mq` was built with the `http-import` feature, you can import directly from GitHub without any local setup:

```sh
mq -I raw 'import "github.com/harehare/aws.mq" | aws::ec2::instances(.) | map(fn(i): i["InstanceId"];)' response.json
```

Pin to a specific release with `@vX.Y.Z`:

```sh
mq -I raw 'import "github.com/harehare/aws.mq@v1.0.0" | aws::ec2::running_instances(.)' response.json
```

## Usage

```sh
# List running EC2 instance IDs
aws ec2 describe-instances | mq -I raw 'import "aws" | aws::ec2::instance_ids(json::json_parse(.))'

# Show S3 bucket list as a Markdown table
aws s3api list-buckets | mq -I raw 'import "aws" | aws::s3::buckets_to_markdown_table(json::json_parse(.))'

# Count Lambda functions per runtime
aws lambda list-functions | mq -I raw 'import "aws" | aws::lambda::by_runtime(json::json_parse(.), "python3.11") | map(fn(f): f["FunctionName"];)'
```

## API

### EC2

| Function | Description |
|---|---|
| `ec2::instances(response)` | All instances from `DescribeInstances` |
| `ec2::instances_by_state(response, state)` | Filter by state (`"running"`, `"stopped"`, …) |
| `ec2::running_instances(response)` | Running instances |
| `ec2::stopped_instances(response)` | Stopped instances |
| `ec2::instances_by_type(response, type)` | Filter by instance type |
| `ec2::instance_ids(response)` | List of instance IDs |
| `ec2::instance_name(instance)` | Name tag value for an instance |
| `ec2::instances_to_markdown_table(response)` | Markdown table |
| `ec2::security_groups(response)` | All security groups from `DescribeSecurityGroups` |
| `ec2::security_groups_by_vpc(response, vpc_id)` | Filter by VPC |
| `ec2::security_groups_to_markdown_table(response)` | Markdown table |
| `ec2::vpcs(response)` | All VPCs from `DescribeVpcs` |
| `ec2::default_vpc(response)` | The default VPC |
| `ec2::vpcs_to_markdown_table(response)` | Markdown table |
| `ec2::subnets(response)` | All subnets from `DescribeSubnets` |
| `ec2::subnets_by_vpc(response, vpc_id)` | Filter by VPC |
| `ec2::subnets_to_markdown_table(response)` | Markdown table |

### S3

| Function | Description |
|---|---|
| `s3::buckets(response)` | All buckets from `ListBuckets` |
| `s3::bucket_names(response)` | List of bucket names |
| `s3::buckets_to_markdown_table(response)` | Markdown table |
| `s3::objects(response)` | All objects from `ListObjectsV2` |
| `s3::object_keys(response)` | List of object keys |
| `s3::total_size(response)` | Total size in bytes |
| `s3::objects_by_storage_class(response, class)` | Filter by storage class |
| `s3::objects_to_markdown_table(response)` | Markdown table |

### IAM

| Function | Description |
|---|---|
| `iam::users(response)` | All users from `ListUsers` |
| `iam::usernames(response)` | List of usernames |
| `iam::users_to_markdown_table(response)` | Markdown table |
| `iam::roles(response)` | All roles from `ListRoles` |
| `iam::role_names(response)` | List of role names |
| `iam::roles_to_markdown_table(response)` | Markdown table |
| `iam::policies(response)` | All policies from `ListPolicies` |
| `iam::policy_names(response)` | List of policy names |
| `iam::policies_to_markdown_table(response)` | Markdown table |

### Lambda

| Function | Description |
|---|---|
| `lambda::functions(response)` | All functions from `ListFunctions` |
| `lambda::function_names(response)` | List of function names |
| `lambda::by_runtime(response, runtime)` | Filter by runtime (e.g. `"python3.11"`) |
| `lambda::by_state(response, state)` | Filter by state (`"Active"`, `"Inactive"`) |
| `lambda::total_memory(response)` | Total allocated memory in MB |
| `lambda::functions_to_markdown_table(response)` | Markdown table |

### RDS

| Function | Description |
|---|---|
| `rds::db_instances(response)` | All instances from `DescribeDBInstances` |
| `rds::available_instances(response)` | Available instances |
| `rds::by_engine(response, engine)` | Filter by engine (`"mysql"`, `"postgres"`) |
| `rds::instance_ids(response)` | List of DB instance identifiers |
| `rds::db_instances_to_markdown_table(response)` | Markdown table |

### CloudFormation

| Function | Description |
|---|---|
| `cloudformation::stacks(response)` | All stacks from `DescribeStacks` |
| `cloudformation::by_status(response, status)` | Filter by status |
| `cloudformation::stack_names(response)` | List of stack names |
| `cloudformation::stack_outputs(stack)` | Outputs array from a stack |
| `cloudformation::stack_parameters(stack)` | Parameters array from a stack |
| `cloudformation::output_value(stack, key)` | Get an output value by key |
| `cloudformation::parameter_value(stack, key)` | Get a parameter value by key |
| `cloudformation::stacks_to_markdown_table(response)` | Markdown table |

### SQS

| Function | Description |
|---|---|
| `sqs::queue_urls(response)` | All queue URLs from `ListQueues` |
| `sqs::queue_names(response)` | Queue names extracted from URLs |
| `sqs::queue_count(response)` | Number of queues |
| `sqs::queue_urls_to_markdown_table(response)` | Markdown table |
| `sqs::attribute(response, name)` | Attribute from `GetQueueAttributes` |
| `sqs::message_count(response)` | Approximate visible message count |
| `sqs::in_flight_count(response)` | Approximate in-flight message count |

### SNS

| Function | Description |
|---|---|
| `sns::topics(response)` | All topics from `ListTopics` |
| `sns::topic_arns(response)` | List of topic ARNs |
| `sns::topics_to_markdown_table(response)` | Markdown table |
| `sns::subscriptions(response)` | All subscriptions from `ListSubscriptions` |
| `sns::subscriptions_by_protocol(response, protocol)` | Filter by protocol (`"email"`, `"sqs"`, …) |
| `sns::subscriptions_by_topic(response, topic_arn)` | Filter by topic ARN |
| `sns::subscriptions_to_markdown_table(response)` | Markdown table |

### ECS

| Function | Description |
|---|---|
| `ecs::clusters(response)` | All clusters from `DescribeClusters` |
| `ecs::cluster_names(response)` | List of cluster names |
| `ecs::active_clusters(response)` | ACTIVE clusters |
| `ecs::clusters_to_markdown_table(response)` | Markdown table |
| `ecs::services(response)` | All services from `DescribeServices` |
| `ecs::running_services(response)` | ACTIVE services |
| `ecs::services_to_markdown_table(response)` | Markdown table |
| `ecs::tasks(response)` | All tasks from `DescribeTasks` |
| `ecs::running_tasks(response)` | RUNNING tasks |
| `ecs::tasks_to_markdown_table(response)` | Markdown table |

### DynamoDB

| Function | Description |
|---|---|
| `dynamodb::table_names(response)` | All table names from `ListTables` |
| `dynamodb::table_count(response)` | Number of tables |
| `dynamodb::table_names_to_markdown_table(response)` | Markdown table |
| `dynamodb::table(response)` | Table object from `DescribeTable` |
| `dynamodb::table_status(response)` | Table status |
| `dynamodb::item_count(response)` | Item count |
| `dynamodb::table_info_to_markdown_table(response)` | Markdown table |

### CloudWatch

| Function | Description |
|---|---|
| `cloudwatch::alarms(response)` | All alarms from `DescribeAlarms` |
| `cloudwatch::alarms_in_state(response, state)` | Filter by state |
| `cloudwatch::ok_alarms(response)` | Alarms in OK state |
| `cloudwatch::alarm_alarms(response)` | Alarms in ALARM state |
| `cloudwatch::insufficient_data_alarms(response)` | Alarms in INSUFFICIENT_DATA state |
| `cloudwatch::alarm_names(response)` | List of alarm names |
| `cloudwatch::alarms_to_markdown_table(response)` | Markdown table |

### SSM Parameter Store

| Function | Description |
|---|---|
| `ssm::parameters(response)` | All parameters from `DescribeParameters` / `GetParametersByPath` |
| `ssm::parameter_names(response)` | List of parameter names |
| `ssm::by_type(response, type)` | Filter by type (`"String"`, `"SecureString"`, `"StringList"`) |
| `ssm::parameter_values(response)` | List of values (only from `GetParametersByPath`) |
| `ssm::parameters_to_markdown_table(response)` | Markdown table |

### Secrets Manager

| Function | Description |
|---|---|
| `secretsmanager::secrets(response)` | All secrets from `ListSecrets` |
| `secretsmanager::secret_names(response)` | List of secret names |
| `secretsmanager::secrets_to_markdown_table(response)` | Markdown table |

### ELBv2

| Function | Description |
|---|---|
| `elbv2::load_balancers(response)` | All load balancers from `DescribeLoadBalancers` |
| `elbv2::active_load_balancers(response)` | Load balancers with state `active` |
| `elbv2::by_type(response, type)` | Filter by type (`"application"`, `"network"`, `"gateway"`) |
| `elbv2::load_balancer_names(response)` | List of names |
| `elbv2::load_balancers_to_markdown_table(response)` | Markdown table |
| `elbv2::target_groups(response)` | All target groups from `DescribeTargetGroups` |
| `elbv2::target_group_names(response)` | List of names |
| `elbv2::target_groups_to_markdown_table(response)` | Markdown table |

### Route53

| Function | Description |
|---|---|
| `route53::hosted_zones(response)` | All hosted zones from `ListHostedZones` |
| `route53::zone_names(response)` | List of zone names |
| `route53::public_zones(response)` | Public hosted zones |
| `route53::private_zones(response)` | Private hosted zones |
| `route53::hosted_zones_to_markdown_table(response)` | Markdown table |
| `route53::record_sets(response)` | All records from `ListResourceRecordSets` |
| `route53::by_record_type(response, type)` | Filter by DNS type (`"A"`, `"CNAME"`, `"MX"`, …) |
| `route53::record_sets_to_markdown_table(response)` | Markdown table |

## Examples

```sh
# Show all running EC2 instances as a Markdown table
aws ec2 describe-instances \
  | mq -I raw 'import "aws" | aws::ec2::instances_to_markdown_table(json::json_parse(.))'

# List SQS queue names
aws sqs list-queues \
  | mq -I raw 'import "aws" | aws::sqs::queue_names(json::json_parse(.))'

# Show ALARM-state CloudWatch alarms
aws cloudwatch describe-alarms \
  | mq -I raw 'import "aws" | aws::cloudwatch::alarm_alarms(json::json_parse(.)) | map(fn(a): a["AlarmName"];)'

# Count SecureString parameters
aws ssm describe-parameters \
  | mq -I raw 'import "aws" | len(aws::ssm::by_type(json::json_parse(.), "SecureString"))'

# List public Route53 hosted zones
aws route53 list-hosted-zones \
  | mq -I raw 'import "aws" | aws::route53::public_zones(json::json_parse(.)) | map(fn(z): z["Name"];)'

# Show ECS running tasks as a Markdown table
aws ecs describe-tasks --cluster prod-cluster --tasks $(aws ecs list-tasks --cluster prod-cluster --query taskArns --output text) \
  | mq -I raw 'import "aws" | aws::ecs::tasks_to_markdown_table(json::json_parse(.))'
```

## Compatibility

Requires [mq](https://github.com/harehare/mq) v0.5 or later.

## License

MIT
