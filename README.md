<h1 align="center">aws.mq</h1>

An AWS API response processor implemented as an [mq](https://github.com/harehare/mq) module.

Process AWS CLI / SDK JSON responses with filtering, extraction, and Markdown table rendering — covering the most commonly used services.

## Features

- **EC2** — instances, security groups, VPCs, subnets, route tables, internet gateways, VPC endpoints (filter by state, type, VPC)
- **S3** — buckets, objects (filter by storage class, total size)
- **IAM** — users, roles, policies, groups, instance profiles, access keys, MFA devices
- **Lambda** — functions (filter by runtime, state, total memory), event source mappings, layers
- **RDS** — DB instances (filter by engine, status)
- **CloudFormation** — stacks, outputs, parameters
- **SQS** — queue URLs, queue names, message counts
- **SNS** — topics, subscriptions (filter by protocol or topic)
- **ECS** — clusters, services, tasks (filter by status), task definitions, capacity providers
- **DynamoDB** — table names, table details, backups (filter by table / status)
- **CloudWatch** — metric alarms (filter by state: OK / ALARM / INSUFFICIENT_DATA), dashboards
- **SSM** — parameters (filter by type: String / SecureString / StringList)
- **Secrets Manager** — secrets
- **ELBv2** — load balancers, target groups (filter by type / state)
- **Route53** — hosted zones (public / private), resource record sets (filter by type), health checks
- **CloudFront** — distributions (filter by enabled/deployed status)
- **ACM** — SSL/TLS certificates (filter by issued status)
- **KMS** — keys, aliases (filter customer-managed aliases)
- **CloudTrail** — trails (filter by multi-region / log validation)
- **SES** — verified identities, send statistics (delivery/bounce/complaint totals)
- **Redshift** — clusters (filter by status)
- **CodePipeline** — pipelines, stage states (filter failed/in-progress)
- **CodeBuild** — build projects, builds (filter by status)
- **WAF** — Web ACLs (WAFv2)
- **Glue** — ETL jobs, crawlers (filter by state)
- **EKS** — clusters, nodegroups (cluster info, nodegroup scaling config)
- **Kinesis** — streams, stream details (shard count, retention period)
- **ElastiCache** — cache clusters, replication groups (filter by engine / status)
- **API Gateway** — REST APIs (v1), HTTP/WebSocket APIs (v2, filter by protocol)
- **Step Functions** — state machines (filter by type), executions (filter by status)
- **EventBridge** — rules (filter by state), event buses
- **ECR** — repositories, images (tags, total size)
- **Cognito** — user pools, users (filter by status / enabled, attribute lookup)
- **Auto Scaling** — Auto Scaling groups, scaling policies (filter by type / desired capacity)
- **Firehose** — Kinesis Data Firehose delivery streams (list, describe)
- **Athena** — workgroups (filter by state), query executions (filter by state)
- **CodeCommit** — repositories, branches
- **CodeDeploy** — applications, deployments (filter by status)
- **Batch** — job queues, compute environments, jobs (filter by state/status)
- **GuardDuty** — detectors, findings (filter by severity)
- **Backup** — backup plans, backup jobs (filter by state)
- **EFS** — file systems (filter by state / encryption), mount targets
- **SageMaker** — training jobs, models, endpoints (filter by status)

## Installation

Copy `aws.mq` to your mq module directory:

```sh
cp aws.mq ~/.mq/
```

### HTTP Import (no local installation needed)

If `mq` was built with the `http-import` feature, you can import directly from GitHub without any local setup:

```sh
mq -I raw 'import "github.com/harehare/aws.mq" | aws::from_json | aws::ec2::instances | map(fn(i): i["InstanceId"];)' response.json
```

Pin to a specific release with `@vX.Y.Z`:

```sh
mq -I raw 'import "github.com/harehare/aws.mq@v1.0.0" | aws::from_json | aws::ec2::running_instances' response.json
```

## Usage

```sh
# List running EC2 instance IDs
aws ec2 describe-instances | mq -I raw 'import "aws" | aws::from_json | aws::ec2::instance_ids'

# Show S3 bucket list as a Markdown table
aws s3api list-buckets | mq -I raw 'import "aws" | aws::from_json | aws::s3::buckets_to_markdown_table'

# Count Lambda functions per runtime
aws lambda list-functions | mq -I raw 'import "aws" | aws::from_json | aws::lambda::by_runtime("python3.11") | map(fn(f): f["FunctionName"];)'
```

## API

### Core

| Function | Description |
|---|---|
| `from_json(s)` | Parse a raw JSON string into a value; use with `\|` to feed AWS CLI output into any function |

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
| `ec2::images(response)` | All AMIs from `DescribeImages` |
| `ec2::available_images(response)` | Available AMIs |
| `ec2::images_by_owner(response, owner_id)` | Filter by owner ID |
| `ec2::images_to_markdown_table(response)` | Markdown table |
| `ec2::addresses(response)` | Elastic IPs from `DescribeAddresses` |
| `ec2::unassociated_addresses(response)` | Unattached Elastic IPs |
| `ec2::addresses_to_markdown_table(response)` | Markdown table |
| `ec2::nat_gateways(response)` | NAT Gateways from `DescribeNatGateways` |
| `ec2::available_nat_gateways(response)` | Available NAT Gateways |
| `ec2::nat_gateways_by_vpc(response, vpc_id)` | Filter by VPC |
| `ec2::nat_gateways_to_markdown_table(response)` | Markdown table |
| `ec2::route_tables(response)` | Route tables from `DescribeRouteTables` |
| `ec2::route_tables_by_vpc(response, vpc_id)` | Filter by VPC |
| `ec2::route_tables_to_markdown_table(response)` | Markdown table |
| `ec2::internet_gateways(response)` | Internet gateways from `DescribeInternetGateways` |
| `ec2::internet_gateways_by_vpc(response, vpc_id)` | Filter by attached VPC |
| `ec2::internet_gateway_ids(response)` | List of gateway IDs |
| `ec2::internet_gateways_to_markdown_table(response)` | Markdown table |
| `ec2::vpc_endpoints(response)` | VPC endpoints from `DescribeVpcEndpoints` |
| `ec2::vpc_endpoints_by_vpc(response, vpc_id)` | Filter by VPC |
| `ec2::vpc_endpoints_by_type(response, type)` | Filter by type (`"Gateway"`, `"Interface"`) |
| `ec2::vpc_endpoints_to_markdown_table(response)` | Markdown table |

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
| `iam::groups(response)` | All groups from `ListGroups` |
| `iam::group_names(response)` | List of group names |
| `iam::groups_to_markdown_table(response)` | Markdown table |
| `iam::instance_profiles(response)` | All profiles from `ListInstanceProfiles` |
| `iam::instance_profile_names(response)` | List of profile names |
| `iam::instance_profiles_to_markdown_table(response)` | Markdown table |
| `iam::access_keys(response)` | Access key metadata from `ListAccessKeys` |
| `iam::active_access_keys(response)` | Active access keys |
| `iam::access_keys_to_markdown_table(response)` | Markdown table |
| `iam::mfa_devices(response)` | MFA devices from `ListMFADevices` |
| `iam::mfa_device_usernames(response)` | Usernames with MFA devices |
| `iam::mfa_devices_to_markdown_table(response)` | Markdown table |

### Lambda

| Function | Description |
|---|---|
| `lambda::functions(response)` | All functions from `ListFunctions` |
| `lambda::function_names(response)` | List of function names |
| `lambda::by_runtime(response, runtime)` | Filter by runtime (e.g. `"python3.11"`) |
| `lambda::by_state(response, state)` | Filter by state (`"Active"`, `"Inactive"`) |
| `lambda::total_memory(response)` | Total allocated memory in MB |
| `lambda::functions_to_markdown_table(response)` | Markdown table |
| `lambda::event_source_mappings(response)` | Event source mappings from `ListEventSourceMappings` |
| `lambda::enabled_event_source_mappings(response)` | Enabled mappings only |
| `lambda::event_source_mappings_to_markdown_table(response)` | Markdown table |
| `lambda::layers(response)` | Layers from `ListLayers` |
| `lambda::layer_names(response)` | List of layer names |
| `lambda::layers_to_markdown_table(response)` | Markdown table |

### RDS

| Function | Description |
|---|---|
| `rds::db_instances(response)` | All instances from `DescribeDBInstances` |
| `rds::available_instances(response)` | Available instances |
| `rds::by_engine(response, engine)` | Filter by engine (`"mysql"`, `"postgres"`) |
| `rds::instance_ids(response)` | List of DB instance identifiers |
| `rds::db_instances_to_markdown_table(response)` | Markdown table |
| `rds::db_snapshots(response)` | All snapshots from `DescribeDBSnapshots` |
| `rds::automated_snapshots(response)` | Automated snapshots |
| `rds::manual_snapshots(response)` | Manual snapshots |
| `rds::available_snapshots(response)` | Available snapshots |
| `rds::db_snapshots_to_markdown_table(response)` | Markdown table |
| `rds::db_clusters(response)` | Aurora clusters from `DescribeDBClusters` |
| `rds::available_db_clusters(response)` | Available clusters |
| `rds::db_cluster_ids(response)` | List of cluster identifiers |
| `rds::db_clusters_to_markdown_table(response)` | Markdown table |

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
| `ecs::task_definition_arns(response)` | Task definition ARNs from `ListTaskDefinitions` |
| `ecs::task_definition_count(response)` | Count of task definitions |
| `ecs::task_definition_arns_to_markdown_table(response)` | Markdown table |
| `ecs::task_definition(response)` | Task definition object from `DescribeTaskDefinition` |
| `ecs::task_definition_info_to_markdown_table(response)` | Markdown table |

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
| `dynamodb::backups(response)` | Backup summaries from `ListBackups` |
| `dynamodb::backups_by_table(response, table_name)` | Filter by table name |
| `dynamodb::available_backups(response)` | Available backups only |
| `dynamodb::backups_to_markdown_table(response)` | Markdown table |

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
| `cloudwatch::log_groups(response)` | Log groups from `DescribeLogGroups` |
| `cloudwatch::log_group_names(response)` | List of log group names |
| `cloudwatch::total_stored_bytes(response)` | Total stored bytes across all log groups |
| `cloudwatch::log_groups_to_markdown_table(response)` | Markdown table |
| `cloudwatch::log_streams(response)` | Log streams from `DescribeLogStreams` |
| `cloudwatch::log_stream_names(response)` | List of log stream names |
| `cloudwatch::log_streams_to_markdown_table(response)` | Markdown table |
| `cloudwatch::dashboards(response)` | Dashboards from `ListDashboards` |
| `cloudwatch::dashboard_names(response)` | List of dashboard names |
| `cloudwatch::dashboards_to_markdown_table(response)` | Markdown table |

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
| `route53::health_checks(response)` | Health checks from `ListHealthChecks` |
| `route53::health_checks_by_type(response, type)` | Filter by type (`"HTTP"`, `"HTTPS"`, `"TCP"`) |
| `route53::health_check_count(response)` | Count of health checks |
| `route53::health_checks_to_markdown_table(response)` | Markdown table |

### CloudFront

| Function | Description |
|---|---|
| `cloudfront::distributions(response)` | Distributions from `ListDistributions` |
| `cloudfront::deployed_distributions(response)` | Deployed distributions |
| `cloudfront::enabled_distributions(response)` | Enabled distributions |
| `cloudfront::distribution_domain_names(response)` | List of domain names |
| `cloudfront::distributions_to_markdown_table(response)` | Markdown table |

### ACM

| Function | Description |
|---|---|
| `acm::certificates(response)` | Certificates from `ListCertificates` |
| `acm::issued_certificates(response)` | Issued certificates |
| `acm::certificate_domain_names(response)` | List of domain names |
| `acm::certificates_to_markdown_table(response)` | Markdown table |

### KMS

| Function | Description |
|---|---|
| `kms::keys(response)` | Keys from `ListKeys` |
| `kms::key_count(response)` | Number of keys |
| `kms::keys_to_markdown_table(response)` | Markdown table |
| `kms::aliases(response)` | Aliases from `ListAliases` |
| `kms::alias_names(response)` | List of alias names |
| `kms::customer_aliases(response)` | Aliases with a customer-managed TargetKeyId |
| `kms::aliases_to_markdown_table(response)` | Markdown table |

### CloudTrail

| Function | Description |
|---|---|
| `cloudtrail::trails(response)` | Trails from `DescribeTrails` |
| `cloudtrail::trail_names(response)` | List of trail names |
| `cloudtrail::multi_region_trails(response)` | Multi-region trails |
| `cloudtrail::validated_trails(response)` | Trails with log file validation |
| `cloudtrail::trails_to_markdown_table(response)` | Markdown table |

### SES

| Function | Description |
|---|---|
| `ses::identities(response)` | Identities from `ListIdentities` |
| `ses::identity_count(response)` | Number of verified identities |
| `ses::identities_to_markdown_table(response)` | Markdown table |
| `ses::send_statistics(response)` | Data points from `GetSendStatistics` |
| `ses::total_deliveries(response)` | Total delivery attempts |
| `ses::total_bounces(response)` | Total bounces |
| `ses::total_complaints(response)` | Total complaints |
| `ses::send_statistics_to_markdown_table(response)` | Markdown table |

### Redshift

| Function | Description |
|---|---|
| `redshift::clusters(response)` | Clusters from `DescribeClusters` |
| `redshift::available_clusters(response)` | Available clusters |
| `redshift::cluster_identifiers(response)` | List of cluster identifiers |
| `redshift::clusters_to_markdown_table(response)` | Markdown table |

### CodePipeline

| Function | Description |
|---|---|
| `codepipeline::pipelines(response)` | Pipelines from `ListPipelines` |
| `codepipeline::pipeline_names(response)` | List of pipeline names |
| `codepipeline::pipelines_to_markdown_table(response)` | Markdown table |
| `codepipeline::stage_states(response)` | Stage states from `GetPipelineState` |
| `codepipeline::failed_stages(response)` | Stages with Failed latest execution |
| `codepipeline::in_progress_stages(response)` | Stages with InProgress latest execution |
| `codepipeline::stage_states_to_markdown_table(response)` | Markdown table |

### CodeBuild

| Function | Description |
|---|---|
| `codebuild::project_names(response)` | Project names from `ListProjects` |
| `codebuild::project_count(response)` | Number of build projects |
| `codebuild::project_names_to_markdown_table(response)` | Markdown table |
| `codebuild::builds(response)` | Builds from `BatchGetBuilds` |
| `codebuild::builds_by_status(response, status)` | Filter by status (`"SUCCEEDED"`, `"FAILED"`, …) |
| `codebuild::succeeded_builds(response)` | Succeeded builds |
| `codebuild::failed_builds(response)` | Failed builds |
| `codebuild::builds_to_markdown_table(response)` | Markdown table |

### WAF

| Function | Description |
|---|---|
| `waf::web_acls(response)` | Web ACLs from `ListWebACLs` (WAFv2) |
| `waf::web_acl_names(response)` | List of Web ACL names |
| `waf::web_acls_to_markdown_table(response)` | Markdown table |

### Glue

| Function | Description |
|---|---|
| `glue::jobs(response)` | ETL jobs from `GetJobs` |
| `glue::job_names(response)` | List of job names |
| `glue::jobs_to_markdown_table(response)` | Markdown table |
| `glue::crawlers(response)` | Crawlers from `GetCrawlers` |
| `glue::crawler_names(response)` | List of crawler names |
| `glue::ready_crawlers(response)` | Crawlers in READY state |
| `glue::running_crawlers(response)` | Crawlers in RUNNING state |
| `glue::crawlers_to_markdown_table(response)` | Markdown table |

### EKS

| Function | Description |
|---|---|
| `eks::cluster_names(response)` | Cluster names from `ListClusters` |
| `eks::cluster_count(response)` | Number of clusters |
| `eks::cluster_names_to_markdown_table(response)` | Markdown table |
| `eks::cluster(response)` | Cluster object from `DescribeCluster` |
| `eks::cluster_status(response)` | Cluster status |
| `eks::cluster_version(response)` | Kubernetes version |
| `eks::cluster_info_to_markdown_table(response)` | Markdown table |
| `eks::nodegroup_names(response)` | Nodegroup names from `ListNodegroups` |
| `eks::nodegroup_names_to_markdown_table(response)` | Markdown table |
| `eks::nodegroup(response)` | Nodegroup object from `DescribeNodegroup` |
| `eks::nodegroup_info_to_markdown_table(response)` | Markdown table |

### Kinesis

| Function | Description |
|---|---|
| `kinesis::stream_names(response)` | Stream names from `ListStreams` |
| `kinesis::stream_count(response)` | Number of streams |
| `kinesis::stream_names_to_markdown_table(response)` | Markdown table |
| `kinesis::stream_description(response)` | Stream description from `DescribeStream` |
| `kinesis::stream_status(response)` | Stream status |
| `kinesis::shards(response)` | List of shards |
| `kinesis::shard_count(response)` | Number of shards |
| `kinesis::stream_info_to_markdown_table(response)` | Markdown table |

### ElastiCache

| Function | Description |
|---|---|
| `elasticache::cache_clusters(response)` | All clusters from `DescribeCacheClusters` |
| `elasticache::available_clusters(response)` | Available clusters |
| `elasticache::by_engine(response, engine)` | Filter by engine (`"redis"`, `"memcached"`) |
| `elasticache::cluster_ids(response)` | List of cluster IDs |
| `elasticache::cache_clusters_to_markdown_table(response)` | Markdown table |
| `elasticache::replication_groups(response)` | All groups from `DescribeReplicationGroups` |
| `elasticache::replication_group_ids(response)` | List of group IDs |
| `elasticache::available_replication_groups(response)` | Available replication groups |
| `elasticache::replication_groups_to_markdown_table(response)` | Markdown table |

### API Gateway

| Function | Description |
|---|---|
| `apigateway::rest_apis(response)` | REST APIs from `GetRestApis` (v1) |
| `apigateway::rest_api_names(response)` | List of REST API names |
| `apigateway::rest_apis_to_markdown_table(response)` | Markdown table |
| `apigateway::http_apis(response)` | HTTP/WebSocket APIs from `GetApis` (v2) |
| `apigateway::http_api_names(response)` | List of HTTP API names |
| `apigateway::http_apis_by_protocol(response, protocol)` | Filter by protocol (`"HTTP"`, `"WEBSOCKET"`) |
| `apigateway::http_apis_to_markdown_table(response)` | Markdown table |

### Step Functions

| Function | Description |
|---|---|
| `stepfunctions::state_machines(response)` | State machines from `ListStateMachines` |
| `stepfunctions::state_machine_names(response)` | List of state machine names |
| `stepfunctions::by_type(response, type)` | Filter by type (`"STANDARD"`, `"EXPRESS"`) |
| `stepfunctions::state_machines_to_markdown_table(response)` | Markdown table |
| `stepfunctions::executions(response)` | Executions from `ListExecutions` |
| `stepfunctions::executions_by_status(response, status)` | Filter by status |
| `stepfunctions::running_executions(response)` | Running executions |
| `stepfunctions::executions_to_markdown_table(response)` | Markdown table |

### EventBridge

| Function | Description |
|---|---|
| `eventbridge::rules(response)` | Rules from `ListRules` |
| `eventbridge::rule_names(response)` | List of rule names |
| `eventbridge::enabled_rules(response)` | Enabled rules |
| `eventbridge::disabled_rules(response)` | Disabled rules |
| `eventbridge::rules_to_markdown_table(response)` | Markdown table |
| `eventbridge::event_buses(response)` | Event buses from `ListEventBuses` |
| `eventbridge::event_bus_names(response)` | List of event bus names |
| `eventbridge::event_buses_to_markdown_table(response)` | Markdown table |

### ECR

| Function | Description |
|---|---|
| `ecr::repositories(response)` | Repositories from `DescribeRepositories` |
| `ecr::repository_names(response)` | List of repository names |
| `ecr::repositories_to_markdown_table(response)` | Markdown table |
| `ecr::images(response)` | Images from `DescribeImages` |
| `ecr::image_tags(response)` | All image tags (flattened) |
| `ecr::total_image_size(response)` | Total image size in bytes |
| `ecr::images_to_markdown_table(response)` | Markdown table |

### Cognito

| Function | Description |
|---|---|
| `cognito::user_pools(response)` | User pools from `ListUserPools` |
| `cognito::user_pool_names(response)` | List of user pool names |
| `cognito::user_pools_to_markdown_table(response)` | Markdown table |
| `cognito::users(response)` | Users from `ListUsers` |
| `cognito::usernames(response)` | List of usernames |
| `cognito::confirmed_users(response)` | CONFIRMED users |
| `cognito::users_by_status(response, status)` | Filter by status (`"CONFIRMED"`, `"UNCONFIRMED"`, …) |
| `cognito::enabled_users(response)` | Enabled users |
| `cognito::user_attribute(user, name)` | Get a user attribute value by name |
| `cognito::users_to_markdown_table(response)` | Markdown table |

### Auto Scaling

| Function | Description |
|---|---|
| `autoscaling::groups(response)` | All groups from `DescribeAutoScalingGroups` |
| `autoscaling::group_names(response)` | List of group names |
| `autoscaling::groups_by_min_desired(response, min)` | Filter by minimum desired capacity |
| `autoscaling::groups_to_markdown_table(response)` | Markdown table |
| `autoscaling::scaling_policies(response)` | Policies from `DescribePolicies` |
| `autoscaling::policies_by_type(response, type)` | Filter by type (`"TargetTrackingScaling"`, `"StepScaling"`) |
| `autoscaling::policy_names(response)` | List of policy names |
| `autoscaling::scaling_policies_to_markdown_table(response)` | Markdown table |

### Firehose

| Function | Description |
|---|---|
| `firehose::stream_names(response)` | Delivery stream names from `ListDeliveryStreams` |
| `firehose::stream_count(response)` | Number of delivery streams |
| `firehose::stream_names_to_markdown_table(response)` | Markdown table |
| `firehose::stream_description(response)` | Stream description from `DescribeDeliveryStream` |
| `firehose::stream_status(response)` | Stream status |
| `firehose::stream_info_to_markdown_table(response)` | Markdown table |

### Athena

| Function | Description |
|---|---|
| `athena::workgroups(response)` | All workgroups from `ListWorkGroups` |
| `athena::workgroup_names(response)` | List of workgroup names |
| `athena::enabled_workgroups(response)` | ENABLED workgroups |
| `athena::workgroups_to_markdown_table(response)` | Markdown table |
| `athena::query_executions(response)` | Executions from `BatchGetQueryExecution` |
| `athena::executions_by_state(response, state)` | Filter by state (`"SUCCEEDED"`, `"FAILED"`, `"RUNNING"`) |
| `athena::succeeded_executions(response)` | Succeeded executions |
| `athena::failed_executions(response)` | Failed executions |
| `athena::query_executions_to_markdown_table(response)` | Markdown table |

### CodeCommit

| Function | Description |
|---|---|
| `codecommit::repositories(response)` | All repositories from `ListRepositories` |
| `codecommit::repository_names(response)` | List of repository names |
| `codecommit::repositories_to_markdown_table(response)` | Markdown table |
| `codecommit::branches(response)` | Branches from `ListBranches` |
| `codecommit::branch_count(response)` | Number of branches |
| `codecommit::branches_to_markdown_table(response)` | Markdown table |

### CodeDeploy

| Function | Description |
|---|---|
| `codedeploy::application_names(response)` | Application names from `ListApplications` |
| `codedeploy::application_count(response)` | Number of applications |
| `codedeploy::application_names_to_markdown_table(response)` | Markdown table |
| `codedeploy::deployments(response)` | Deployments from `BatchGetDeployments` |
| `codedeploy::deployments_by_status(response, status)` | Filter by status (`"Succeeded"`, `"Failed"`, `"InProgress"`) |
| `codedeploy::succeeded_deployments(response)` | Succeeded deployments |
| `codedeploy::failed_deployments(response)` | Failed deployments |
| `codedeploy::deployments_to_markdown_table(response)` | Markdown table |

### Batch

| Function | Description |
|---|---|
| `batch::job_queues(response)` | Job queues from `DescribeJobQueues` |
| `batch::queue_names(response)` | List of queue names |
| `batch::enabled_queues(response)` | ENABLED job queues |
| `batch::job_queues_to_markdown_table(response)` | Markdown table |
| `batch::compute_environments(response)` | Compute environments from `DescribeComputeEnvironments` |
| `batch::environment_names(response)` | List of environment names |
| `batch::enabled_environments(response)` | ENABLED compute environments |
| `batch::compute_environments_to_markdown_table(response)` | Markdown table |
| `batch::jobs(response)` | Job summaries from `ListJobs` |
| `batch::jobs_by_status(response, status)` | Filter by status (`"SUCCEEDED"`, `"FAILED"`, `"RUNNING"`) |
| `batch::running_jobs(response)` | RUNNING jobs |
| `batch::failed_jobs(response)` | FAILED jobs |
| `batch::jobs_to_markdown_table(response)` | Markdown table |

### GuardDuty

| Function | Description |
|---|---|
| `guardduty::detector_ids(response)` | Detector IDs from `ListDetectors` |
| `guardduty::detector_count(response)` | Number of detectors |
| `guardduty::detector_ids_to_markdown_table(response)` | Markdown table |
| `guardduty::findings(response)` | Findings from `GetFindings` |
| `guardduty::findings_above_severity(response, threshold)` | Filter by minimum severity (0–10) |
| `guardduty::high_severity_findings(response)` | High-severity findings (≥ 7.0) |
| `guardduty::medium_severity_findings(response)` | Medium-severity findings (4.0 – 6.9) |
| `guardduty::findings_to_markdown_table(response)` | Markdown table |

### Backup

| Function | Description |
|---|---|
| `backup::backup_plans(response)` | Backup plans from `ListBackupPlans` |
| `backup::plan_names(response)` | List of plan names |
| `backup::backup_plans_to_markdown_table(response)` | Markdown table |
| `backup::backup_jobs(response)` | Backup jobs from `ListBackupJobs` |
| `backup::jobs_by_state(response, state)` | Filter by state (`"COMPLETED"`, `"FAILED"`, `"RUNNING"`) |
| `backup::completed_jobs(response)` | Completed backup jobs |
| `backup::failed_jobs(response)` | Failed backup jobs |
| `backup::backup_jobs_to_markdown_table(response)` | Markdown table |

### EFS

| Function | Description |
|---|---|
| `efs::file_systems(response)` | File systems from `DescribeFileSystems` |
| `efs::file_system_ids(response)` | List of file system IDs |
| `efs::available_file_systems(response)` | Available file systems |
| `efs::encrypted_file_systems(response)` | Encrypted file systems |
| `efs::file_systems_to_markdown_table(response)` | Markdown table |
| `efs::mount_targets(response)` | Mount targets from `DescribeMountTargets` |
| `efs::available_mount_targets(response)` | Available mount targets |
| `efs::mount_targets_to_markdown_table(response)` | Markdown table |

### SageMaker

| Function | Description |
|---|---|
| `sagemaker::training_jobs(response)` | Training jobs from `ListTrainingJobs` |
| `sagemaker::training_jobs_by_status(response, status)` | Filter by status (`"Completed"`, `"Failed"`, `"InProgress"`) |
| `sagemaker::completed_training_jobs(response)` | Completed training jobs |
| `sagemaker::training_job_names(response)` | List of job names |
| `sagemaker::training_jobs_to_markdown_table(response)` | Markdown table |
| `sagemaker::models(response)` | Models from `ListModels` |
| `sagemaker::model_names(response)` | List of model names |
| `sagemaker::models_to_markdown_table(response)` | Markdown table |
| `sagemaker::endpoints(response)` | Endpoints from `ListEndpoints` |
| `sagemaker::endpoints_by_status(response, status)` | Filter by status (`"InService"`, `"Creating"`, `"Failed"`) |
| `sagemaker::in_service_endpoints(response)` | InService endpoints |
| `sagemaker::endpoint_names(response)` | List of endpoint names |
| `sagemaker::endpoints_to_markdown_table(response)` | Markdown table |

## Examples

```sh
# Show all running EC2 instances as a Markdown table
aws ec2 describe-instances \
  | mq -I raw 'import "aws" | aws::from_json | aws::ec2::instances_to_markdown_table'

# List SQS queue names
aws sqs list-queues \
  | mq -I raw 'import "aws" | aws::from_json | aws::sqs::queue_names'

# Show ALARM-state CloudWatch alarms
aws cloudwatch describe-alarms \
  | mq -I raw 'import "aws" | aws::from_json | aws::cloudwatch::alarm_alarms | map(fn(a): a["AlarmName"];)'

# Count SecureString parameters
aws ssm describe-parameters \
  | mq -I raw 'import "aws" | aws::from_json | aws::ssm::by_type("SecureString") | len'

# List public Route53 hosted zones
aws route53 list-hosted-zones \
  | mq -I raw 'import "aws" | aws::from_json | aws::route53::public_zones | map(fn(z): z["Name"];)'

# Show ECS running tasks as a Markdown table
aws ecs describe-tasks --cluster prod-cluster --tasks $(aws ecs list-tasks --cluster prod-cluster --query taskArns --output text) \
  | mq -I raw 'import "aws" | aws::from_json | aws::ecs::tasks_to_markdown_table'
```

## Compatibility

Requires [mq](https://github.com/harehare/mq) v0.5 or later.

## License

MIT
