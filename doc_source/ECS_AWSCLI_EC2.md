# Tutorial: Creating a Cluster with an EC2 Task Using the AWS CLI<a name="ECS_AWSCLI_EC2"></a>

The following steps help you set up a cluster, register a task definition, run a task, and perform other common scenarios in Amazon ECS with the AWS CLI\. Ensure that you are using the latest version of the AWS CLI\. For more information on how to upgrade to the latest version, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html)\.

**Topics**
+ [Prerequisites](#AWSCLI_EC2_prereq)
+ [Step 1: \(Optional\) Create a Cluster](#AWSCLI_EC2_create_cluster)
+ [Step 2: Launch an Instance with the Amazon ECS AMI](#AWSCLI_EC2_launch_container_instance)
+ [Step 3: List Container Instances](#AWSCLI_EC2_list_container_instances)
+ [Step 4: Describe your Container Instance](#AWSCLI_EC2_describe_container_instance)
+ [Step 5: Register a Task Definition](#AWSCLI_EC2_register_task_definition)
+ [Step 6: List Task Definitions](#AWSCLI_EC2_list_task_definitions)
+ [Step 7: Run a Task](#AWSCLI_EC2_run_task)
+ [Step 8: List Tasks](#AWSCLI_EC2_list_tasks)
+ [Step 9: Describe the Running Task](#AWSCLI_EC2_describe_task)

## Prerequisites<a name="AWSCLI_EC2_prereq"></a>

This tutorial assumes that the following prerequisites have been completed:
+ The latest version of the AWS CLI is installed and configured\. For more information about installing or upgrading your AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html)\.
+ The steps in [Setting Up with Amazon ECS](get-set-up-for-amazon-ecs.md) have been completed\.
+ Your AWS user has the required permissions specified in the [Amazon ECS First Run Wizard Permissions](security_iam_id-based-policy-examples.md#first-run-permissions) IAM policy example\.
+ You have a VPC and security group created to use\. For more information, see [Tutorial: Creating a VPC with Public and Private Subnets for Your Clusters](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-public-private-vpc.html)\.

## Step 1: \(Optional\) Create a Cluster<a name="AWSCLI_EC2_create_cluster"></a>

By default, your account receives a `default` cluster when you launch your first container instance\.

**Note**  
The benefit of using the `default` cluster that is provided for you is that you don't have to specify the `--cluster cluster_name` option in the subsequent commands\. If you do create your own, non\-default, cluster, you must specify `--cluster cluster_name` for each command that you intend to use with that cluster\.

Create your own cluster with a unique name with the following command:

```
aws ecs create-cluster --cluster-name MyCluster
```

Output:

```
{
    "cluster": {
        "clusterName": "MyCluster",
        "status": "ACTIVE",
        "clusterArn": "arn:aws:ecs:region:aws_account_id:cluster/MyCluster"
    }
}
```

## Step 2: Launch an Instance with the Amazon ECS AMI<a name="AWSCLI_EC2_launch_container_instance"></a>

You must have an Amazon ECS container instance in your cluster before you can run tasks on it\. If you do not have any container instances in your cluster, see [Launching an Amazon ECS Container Instance](launch_container_instance.md) for more information\.

The following table lists the current Amazon ECS\-optimized Amazon Linux AMI IDs by Region\.


| Region Name | Region | AMI ID | EC2 Console Link | 
| --- | --- | --- | --- | 
| US East \(Ohio\) | `us-east-2` | [View AMI ID](https://us-east-2.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=us-east-2#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=us-east-2#LaunchInstanceWizard:ami=ami-035a1bdaf0e4bf265) | 
| US East \(N\. Virginia\) | `us-east-1` | [View AMI ID](https://us-east-1.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=us-east-1#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#LaunchInstanceWizard:ami=ami-066ce9bb9f4cbb03d) | 
| US West \(N\. California\) | `us-west-1` | [View AMI ID](https://us-west-1.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=us-west-1#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=us-west-1#LaunchInstanceWizard:ami=ami-0413317a44231a219) | 
| US West \(Oregon\) | `us-west-2` | [View AMI ID](https://us-west-2.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=us-west-2#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=us-west-2#LaunchInstanceWizard:ami=ami-0fd6e28e415664140) | 
| Asia Pacific \(Hong Kong\) | `ap-east-1` | [View AMI ID](https://ap-east-1.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=ap-east-1#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=ap-east-1#LaunchInstanceWizard:ami=ami-0bc9197107aa2e36d) | 
| Asia Pacific \(Tokyo\) | ap\-northeast\-1 | [View AMI ID](https://ap-northeast-1.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=ap-northeast-1#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=ap-northeast-1#LaunchInstanceWizard:ami=ami-0a4b5b999281c955b) | 
| Asia Pacific \(Seoul\) | ap\-northeast\-2 | [View AMI ID](https://ap-northeast-2.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=ap-northeast-2#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=ap-northeast-2#LaunchInstanceWizard:ami=ami-0c6e04cbad6e3e13f) | 
| Asia Pacific \(Mumbai\) | ap\-south\-1 | [View AMI ID](https://ap-south-1.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=ap-south-1#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=ap-south-1#LaunchInstanceWizard:ami=ami-04a56cb1908b6ce94) | 
| Asia Pacific \(Singapore\) | ap\-southeast\-1 | [View AMI ID](https://ap-southeast-1.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=ap-southeast-1#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=ap-southeast-1#LaunchInstanceWizard:ami=ami-003cb73efe1eb03cc) | 
| Asia Pacific \(Sydney\) | ap\-southeast\-2 | [View AMI ID](https://ap-southeast-2.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=ap-southeast-2#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=ap-southeast-2#LaunchInstanceWizard:ami=ami-0112bb4988eedc594) | 
| Canada \(Central\) | ca\-central\-1 | [View AMI ID](https://ca-central-1.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=ca-central-1#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=ca-central-1#LaunchInstanceWizard:ami=ami-02a835991f91d92cb) | 
| EU \(Frankfurt\) | eu\-central\-1 | [View AMI ID](https://eu-central-1.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=eu-central-1#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=eu-central-1#LaunchInstanceWizard:ami=ami-0b0a910db6581d75f) | 
| EU \(Stockholm\) | eu\-north\-1 | [View AMI ID](https://eu-north-1.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=eu-north-1#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=eu-north-1#LaunchInstanceWizard:ami=ami-03147f238a3100ad7) | 
| EU \(Ireland\) | eu\-west\-1 | [View AMI ID](https://eu-west-1.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=eu-west-1#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=eu-west-1#LaunchInstanceWizard:ami=ami-0d2aaec13a6b7e7ca) | 
| EU \(London\) | eu\-west\-2 | [View AMI ID](https://eu-west-2.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=eu-west-2#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=eu-west-2#LaunchInstanceWizard:ami=ami-0e2d2ad19e82df43b) | 
| EU \(Paris\) | eu\-west\-3 | [View AMI ID](https://eu-west-3.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=eu-west-3#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=eu-west-3#LaunchInstanceWizard:ami=ami-0244cc5d7b6e6d95e) | 
| Middle East \(Bahrain\) | me\-south\-1 | [View AMI ID](https://me-south-1.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=me-south-1#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=me-south-1#LaunchInstanceWizard:ami=ami-06af5a12626c763dc) | 
| South America \(Sao Paulo\) | sa\-east\-1 | [View AMI ID](https://sa-east-1.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=sa-east-1#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=sa-east-1#LaunchInstanceWizard:ami=ami-0149bb9bc7a19cf0e) | 
| AWS GovCloud \(US\-East\) | us\-gov\-east\-1 | [View AMI ID](https://us-gov-east-1.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=us-gov-east-1#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=us-gov-east-1#LaunchInstanceWizard:ami=ami-09c03f1d4c50330f7) | 
| AWS GovCloud \(US\-West\) | us\-gov\-west\-1 | [View AMI ID](https://us-gov-west-1.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=us-gov-west-1#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=us-gov-west-1#LaunchInstanceWizard:ami=ami-ac521dcd) | 
| China \(Beijing\) | `cn-north-1` | [View AMI ID](https://cn-north-1.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=cn-north-1#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=cn-north-1#LaunchInstanceWizard:ami=ami-099eefd6de9767dd1) | 
| China \(Ningxia\) | `cn-northwest-1` | [View AMI ID](https://cn-northwest-1.console.aws.amazon.com/systems-manager/parameters/%252Faws%252Fservice%252Fecs%252Foptimized-ami%252Famazon-linux%252Frecommended%252Fimage_id/description?region=cn-northwest-1#) | [Launch instance](https://console.aws.amazon.com/ec2/v2/home?region=cn-northwest-1#LaunchInstanceWizard:ami=ami-009c89a26befaa039) | 

## Step 3: List Container Instances<a name="AWSCLI_EC2_list_container_instances"></a>

Within a few minutes of launching your container instance, the Amazon ECS agent registers the instance with your default cluster\. You can list the container instances in a cluster by running the following command:

```
aws ecs list-container-instances --cluster default
```

Output:

```
{
    "containerInstanceArns": [
        "arn:aws:ecs:us-east-1:aws_account_id:container-instance/container_instance_ID"
    ]
}
```

## Step 4: Describe your Container Instance<a name="AWSCLI_EC2_describe_container_instance"></a>

After you have the ARN or ID of a container instance, you can use the describe\-container\-instances command to get valuable information on the instance, such as remaining and registered CPU and memory resources\.

```
aws ecs describe-container-instances --cluster default --container-instances container_instance_ID
```

Output:

```
{
    "failures": [],
    "containerInstances": [
        {
            "status": "ACTIVE",
            "registeredResources": [
                {
                    "integerValue": 1024,
                    "longValue": 0,
                    "type": "INTEGER",
                    "name": "CPU",
                    "doubleValue": 0.0
                },
                {
                    "integerValue": 995,
                    "longValue": 0,
                    "type": "INTEGER",
                    "name": "MEMORY",
                    "doubleValue": 0.0
                },
                {
                    "name": "PORTS",
                    "longValue": 0,
                    "doubleValue": 0.0,
                    "stringSetValue": [
                        "22",
                        "2376",
                        "2375",
                        "51678"
                    ],
                    "type": "STRINGSET",
                    "integerValue": 0
                },
                {
                    "name": "PORTS_UDP",
                    "longValue": 0,
                    "doubleValue": 0.0,
                    "stringSetValue": [],
                    "type": "STRINGSET",
                    "integerValue": 0
                }
            ],
            "ec2InstanceId": "instance_id",
            "agentConnected": true,
            "containerInstanceArn": "arn:aws:ecs:us-west-2:aws_account_id:container-instance/container_instance_ID",
            "pendingTasksCount": 0,
            "remainingResources": [
                {
                    "integerValue": 1024,
                    "longValue": 0,
                    "type": "INTEGER",
                    "name": "CPU",
                    "doubleValue": 0.0
                },
                {
                    "integerValue": 995,
                    "longValue": 0,
                    "type": "INTEGER",
                    "name": "MEMORY",
                    "doubleValue": 0.0
                },
                {
                    "name": "PORTS",
                    "longValue": 0,
                    "doubleValue": 0.0,
                    "stringSetValue": [
                        "22",
                        "2376",
                        "2375",
                        "51678"
                    ],
                    "type": "STRINGSET",
                    "integerValue": 0
                },
                {
                    "name": "PORTS_UDP",
                    "longValue": 0,
                    "doubleValue": 0.0,
                    "stringSetValue": [],
                    "type": "STRINGSET",
                    "integerValue": 0
                }
            ],
            "runningTasksCount": 0,
            "attributes": [
                {
                    "name": "com.amazonaws.ecs.capability.privileged-container"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.17"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.18"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.19"
                },
                {
                    "name": "com.amazonaws.ecs.capability.logging-driver.json-file"
                },
                {
                    "name": "com.amazonaws.ecs.capability.logging-driver.syslog"
                }
            ],
            "versionInfo": {
                "agentVersion": "1.5.0",
                "agentHash": "b197edd",
                "dockerVersion": "DockerVersion: 1.7.1"
            }
        }
    ]
}
```

You can also find the Amazon EC2 instance ID that you can use to monitor the instance in the Amazon EC2 console or with the aws ec2 describe\-instances \-\-instance\-id *instance\_id* command\.

## Step 5: Register a Task Definition<a name="AWSCLI_EC2_register_task_definition"></a>

Before you can run a task on your ECS cluster, you must register a task definition\. Task definitions are lists of containers grouped together\. The following example is a simple task definition that uses a `busybox` image from Docker Hub and simply sleeps for 360 seconds\. For more information about the available task definition parameters, see [Amazon ECS Task Definitions](task_definitions.md)\.

```
{
  "containerDefinitions": [
    {
      "name": "sleep",
      "image": "busybox",
      "cpu": 10,
      "command": [
        "sleep",
        "360"
      ],
      "memory": 10,
      "essential": true
    }
  ],
  "family": "sleep360"
}
```

The above example JSON can be passed to the AWS CLI in two ways: You can save the task definition JSON as a file and pass it with the `--cli-input-json file://path_to_file.json` option\. Or, you can escape the quotation marks in the JSON and pass the JSON container definitions on the command line as in the below example\. If you choose to pass the container definitions on the command line, your command additionally requires a `--family` parameter that is used to keep multiple versions of your task definition associated with each other\.

To use a JSON file for container definitions:

```
aws ecs register-task-definition --cli-input-json file://$HOME/tasks/sleep360.json
```

To use a JSON string for container definitions:

```
aws ecs register-task-definition --family sleep360 --container-definitions "[{\"name\":\"sleep\",\"image\":\"busybox\",\"cpu\":10,\"command\":[\"sleep\",\"360\"],\"memory\":10,\"essential\":true}]"
```

The register\-task\-definition returns a description of the task definition after it completes its registration\.

```
{
    "taskDefinition": {
        "volumes": [],
        "taskDefinitionArn": "arn:aws:ec2:us-east-1:aws_account_id:task-definition/sleep360:1",
        "containerDefinitions": [
            {
                "environment": [],
                "name": "sleep",
                "mountPoints": [],
                "image": "busybox",
                "cpu": 10,
                "portMappings": [],
                "command": [
                    "sleep",
                    "360"
                ],
                "memory": 10,
                "essential": true,
                "volumesFrom": []
            }
        ],
        "family": "sleep360",
        "revision": 1
    }
}
```

## Step 6: List Task Definitions<a name="AWSCLI_EC2_list_task_definitions"></a>

You can list the task definitions for your account at any time with the list\-task\-definitions command\. The output of this command shows the `family` and `revision` values that you can use together when calling run\-task or start\-task\.

```
aws ecs list-task-definitions
```

Output:

```
{
    "taskDefinitionArns": [
        "arn:aws:ec2:us-east-1:aws_account_id:task-definition/sleep300:1",
        "arn:aws:ec2:us-east-1:aws_account_id:task-definition/sleep300:2",
        "arn:aws:ec2:us-east-1:aws_account_id:task-definition/sleep360:1",
        "arn:aws:ec2:us-east-1:aws_account_id:task-definition/wordpress:3",
        "arn:aws:ec2:us-east-1:aws_account_id:task-definition/wordpress:4",
        "arn:aws:ec2:us-east-1:aws_account_id:task-definition/wordpress:5",
        "arn:aws:ec2:us-east-1:aws_account_id:task-definition/wordpress:6"
    ]
}
```

## Step 7: Run a Task<a name="AWSCLI_EC2_run_task"></a>

After you have registered a task for your account and have launched a container instance that is registered to your cluster, you can run the registered task in your cluster\. For this example, you place a single instance of the `sleep360:1` task definition in your default cluster\.

```
aws ecs run-task --cluster default --task-definition sleep360:1 --count 1
```

Output:

```
{
    "tasks": [
        {
            "taskArn": "arn:aws:ecs:us-east-1:aws_account_id:task/task_ID",
            "overrides": {
                "containerOverrides": [
                    {
                        "name": "sleep"
                    }
                ]
            },
            "lastStatus": "PENDING",
            "containerInstanceArn": "arn:aws:ecs:us-east-1:aws_account_id:container-instance/container_instance_ID",
            "clusterArn": "arn:aws:ecs:us-east-1:aws_account_id:cluster/default",
            "desiredStatus": "RUNNING",
            "taskDefinitionArn": "arn:aws:ecs:us-east-1:aws_account_id:task-definition/sleep360:1",
            "containers": [
                {
                    "containerArn": "arn:aws:ecs:us-east-1:aws_account_id:container/container_ID",
                    "taskArn": "arn:aws:ecs:us-east-1:aws_account_id:task/task_ID",
                    "lastStatus": "PENDING",
                    "name": "sleep"
                }
            ]
        }
    ]
}
```

## Step 8: List Tasks<a name="AWSCLI_EC2_list_tasks"></a>

List the tasks for your cluster\. You should see the task that you ran in the previous section\. You can take the task ID or the full ARN that is returned from this command and use it to describe the task later\.

```
aws ecs list-tasks --cluster default
```

Output:

```
{
    "taskArns": [
        "arn:aws:ecs:us-east-1:aws_account_id:task/task_ID"
    ]
}
```

## Step 9: Describe the Running Task<a name="AWSCLI_EC2_describe_task"></a>

Describe the task using the task ID retrieved earlier to get more information about the task\.

```
aws ecs describe-tasks --cluster default --task task_ID
```

Output:

```
{
    "failures": [],
    "tasks": [
        {
            "taskArn": "arn:aws:ecs:us-east-1:aws_account_id:task/task_ID",
            "overrides": {
                "containerOverrides": [
                    {
                        "name": "sleep"
                    }
                ]
            },
            "lastStatus": "RUNNING",
            "containerInstanceArn": "arn:aws:ecs:us-east-1:aws_account_id:container-instance/container_instance_ID",
            "clusterArn": "arn:aws:ecs:us-east-1:aws_account_id:cluster/default",
            "desiredStatus": "RUNNING",
            "taskDefinitionArn": "arn:aws:ecs:us-east-1:aws_account_id:task-definition/sleep360:1",
            "containers": [
                {
                    "containerArn": "arn:aws:ecs:us-east-1:aws_account_id:container/container_ID",
                    "taskArn": "arn:aws:ecs:us-east-1:aws_account_id:task/task_ID",
                    "lastStatus": "RUNNING",
                    "name": "sleep",
                    "networkBindings": []
                }
            ]
        }
    ]
}
```