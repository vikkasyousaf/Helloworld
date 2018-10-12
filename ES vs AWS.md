# Investigating Elasticsearch Solution

```
## AWS ES Service

1. It automotically replace failed nodes. You can add or remove nodes through an API but you have to make sure that all the automation are in place so you dont have to do any manual installation or configuration.

2. AWS provide a access rights via IAM instead of using or setting up a reserve proxy or relay on security addon.

3. Daily snapshot to S3, you can use snapshot for recovering of data quickly and is more realiable in case of system failure.

4. 

5. There are tons of things that can cause it to become unstable, most of which are related to query patterns, 
	* The documents being indexed, 
	* The number of dynamic fields being created, 
	* Imbalances in the sizes of shards, 
	* The ratio of documents to heap space, etc. 

	Diagnosing these problems is a bit of an art, and one needs a lot of metrics, log files and administrative APIs to drill down and find the root cause of an issue. AWS’s Elasticsearch doesn’t provide access to any of those things, leaving you no other option but to contact AWS’s support team.

## ES

1. Comparing to AWS, instance are cheaper by 29%. But it vary from instance to instance. CHECK PRICE

2. You can use bigger i2 instances than AWS ES, and you have the access to latest generation of c4 and m4 instances. CHECK INSTANCES

3. Using ES you have more control on index settings, beyond analysis and shards. 
	* Specially Delay allocation when you have a lot of data per node. 
	* You can also change the setting of all the indices at onces. 
	* If you propertly utalize the ES setting you can better optamize different use case as you have more control on the resources as compare to AWS.
	* You can change cluster wide settings, e.g. number of shard to rebalance at once.
	* You also get access to all other API such as Hot Threads which is useful for debugging. 

4. CHECK DEBUGGING

5. You can use a more comprehensive Elasticsearch monitoring solution. Currently CloudWatch only collect few matrics e.g. cluster status, number of nodes and documents, heap pressure and disk space. For more use case you need more information such as query latency and indexing throughput.

6. Without access to logs, without access to admin APIs, without node-level metrics (all you get is cluster-level aggregate metrics) or even the query logs, it’s basically impossible to TROUBLESHOOT your own Elasticsearch cluster.

7. AWS Elasticsearch is easy to setup and come up with few feature, however it is limited in term of scaling (number and size of nodes and elasticsearch feature).
```

## Authentication options

### AWS ES Service

1.  You can authenticate users to the service through Amazon Cognito and restrict access to specified authenticated users using AWS Identity and Access Management (IAM).

2. IAM policies can be set up to provide fine-grained access control:
	* To the management API for operations like creating and scaling domains.
	* And the data plane API for operations like uploading documents and executing queries. 

3. The node-to-node encryption capability by implementing TLS for all communications between instances in the ES domain.

4. AWS Key Management Service (KMS) lets you encrypt data in Amazon Elasticsearch Service at-rest, including primary and replica indices, log files, memory swap files, and automated snapshots.

 AWS support three type of access policy to your domain 
	* Resource-based policies
	* Identity-based policies
	* IP-based policies

### Resource-based Policies

1. These policy define which action a principal can perform on the domain's subresource. Subresource include Elasticsearch, indices and API.

2. The `Principal` element specify the account, user, or role that are allow to access. The `Resource` element specify what subresources these principals can access. In the following the resource-base-policy grand full access by using `es:*` in the `Action` tag.
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::123456789012:user/test-user"
        ]
      },
      "Action": [
        "es:*"
      ],
      "Resource": "arn:aws:es:us-west-1:987654321098:domain/test-domain/*"
    }
  ]
}
```

3. To restric the user replace `es:*` with `es:ESHttpGet`, in this case 'test-user' can only perform search operation, all other indices within the domain are inaccessible, and without permissions to use the es:ESHttpPut or es:ESHttpPost actions, test-user can't add or modify documents.

4. We can also configure the role of power-user. Here will are allowing power-user-role to access all HTTP method excpet deleting cretical index.
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::123456789012:role/power-user-role"
        ]
      },
      "Action": [
        "es:ESHttpDelete",
        "es:ESHttpGet",
        "es:ESHttpHead",
        "es:ESHttpPost",
        "es:ESHttpPut"
      ],
      "Resource": "arn:aws:es:us-west-1:987654321098:domain/test-domain/*"
    },
    {
      "Effect": "Deny",
      "Principal": {
        "AWS": [
          "arn:aws:iam::123456789012:role/power-user-role"
        ]
      },
      "Action": [
        "es:ESHttpDelete"
      ],
      "Resource": "arn:aws:es:us-west-1:987654321098:domain/test-domain/critical-index*"
    }
  ]
}
```

### Identity-based policies

1. Identity-based IAM policies are attached to an IAM user, group, or role. These policies let you specify what that user, group, or role can do. For example, you can attach the policy to the IAM user named Bob, stating that he has permission to use the Amazon Elastic Compute Cloud (Amazon EC2) RunInstances action. The policy could further state that Bob has permission to get items from an Amazon DynamoDB table named MyCompany. You can also grant Bob access to manage his own IAM security credentials.
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "es:Describe*",
        "es:List*",
        "es:ESHttpGet"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```

![Odentity-based policies vs Resource-based policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/images/Types_of_Permissions.diagram.png)

### IP-based Policies

We are currently using this policy.

### AWS ES Service User Authentication and Access for Kibana with Amazon Cognito

1. AWS ES service is now integrate with Amazon Cognito to provide user-level authorization for Kibana, without the need to configure and manage proxy server.

2. With Cognito you can also set access policy for user or group of users, and make it easy to manage access control. 

3. Kibana authentication is supported on all domains that use Elasticsearch 5.1 or greater and is available in 15 regions globally.

### Reference: 

1. [Amazon Elasticsearch Service Access Control](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-ac.html)

2. [Identity-Based Policies and Resource-Based Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_identity-vs-resource.html)

3. [Amazon Cognito for Kibana access control](https://aws.amazon.com/blogs/database/get-started-with-amazon-elasticsearch-service-use-amazon-cognito-for-kibana-access-control/)


## Security concerns
## Pricing

1. On-Demand instance pricing
2. Reserved Instance pricing

## Management effort

### Automated upgrade

1. Support for in-place Elasticsearch upgrades for domains(automated). You dont have to:
	* Create manual snapshots 
	* Create a new domain for the targeted version
	* Restore the manual index snapshot to the new domain
	* Adjust your application configuration

2. You cannot use automated snapshots to migrate to new domains. Automated snapshots are read-only from within a given domain For migrations, you must use manual snapshots stored in your own repository. Standard S3 charges apply to manual snapshots.


### Reference:

1. [In-place version upgrades for Amazon Elasticsearch Service](https://aws.amazon.com/blogs/database/in-place-version-upgrades-for-amazon-elasticsearch-service/)

## Feature set

### Easy to deploy and manage

1. Add and remove nodes from a cluster.

2. Change the instance sizes of your nodes from a drop-down. 

3. You get a semi-useful dashboard of metrics.

4. When nodes go down, they are automatically brought back up.

5. You get automatic and manual snapshots(recover or create a new domain).
	* Retain them for 14 days.
	* Elasticsearch snapshot APIs to create additional manual snapshots.
	* Automated snapshots are stored free of charge in Amazon S3.
	* Manual snapshots will incur standard Amazon S3 [usage charges](https://aws.amazon.com/s3/pricing/).

6. Authentication works seamlessly within AWS’s ecosystem.

7. Plug-ins are automatically deployed and managed for you.

8. Monitoring and domain performance metrics:
	* Domain health,
	* Searchable documents,
	* Amazon EBS metrics, 
	* CPU, memory, and disk utilization for data and master nodes through Amazon CloudWatch

9. Integrated with open-source tools and AWS Services, ingest data into Amazon Elasticsearch domain using,
	* Amazon Kinesis Firehose, AWS IoT, or Amazon CloudWatch Logs.
	* [Amazon Elasticsearch Service data ingestion page.](https://aws.amazon.com/elasticsearch-service/data-ingestion/)


## Available/possible plugins

### AWS ES 

| Elasticsearch Version     | Plugins                       |
|---------------------------|-------------------------------|
| 6.3.                      | ICU Analysis           		|
|                  		    | Ingest Attachment Processor   |
|                 			| Ingest User Agent Processor   |
|       					| Japanese (kuromoji) Analysis  |
|        		            | Mapper Murmur3                |
|          		            | Mapper Size                   |
|         				    | Phonetic Analysis             |
|       			        | Smart Chinese Analysis        |
|      				        | Stempel Polish Analysis       |
|    						| Ukrainian Analysis   			|
|   						| Seunjeon Korean Analysis      |
|   						|    						    |

