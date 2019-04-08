# Investigating Elasticsearch Solution


## Authentication options

### AWS ES Service User Authentication and Access for Kibana with Amazon Cognito

1. AWS ES service is now integrate with Amazon Cognito to provide user-level authorization for Kibana, without the need to configure and manage proxy server.

2. With Cognito you can also set access policy for user or group of users, and make it easy to manage access control. 

3. Kibana authentication is supported on all domains that use Elasticsearch 5.1 or greater and is available in 15 regions globally. 

4.  You can authenticate users to the service through Amazon Cognito and restrict access to specified authenticated users using AWS Identity and Access Management (IAM).

5. AWS support three type of access policy to your domain 
	* Resource-based policies
	* Identity-based policies
	* IP-based policies

### Resource-based policies

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

### Identity-based policies vs Resource-based policies

![Odentity-based policies vs Resource-based policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/images/Types_of_Permissions.diagram.png)

### IP-based policies

1. IP-based policies restrict access to a domain to one or more IP addresses. They are just resource base policy that just specify an anonymous prinicple and include a special condition element.

2. The following IP-base policy grand access to all request that originate from `12.345.678.901`
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": [
        "es:*"
      ],
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": [
            "12.345.678.901"
          ]
        }
      },
      "Resource": "arn:aws:es:us-west-1:987654321098:domain/test-domain/*"
    }
  ]
}

```


### Elasticsearch Authentication

1. X-Pack security provides the means to secure the Elastic cluster
  * User authentication
    * Authentication process is handled by one or more authentication services called realms
    * Support external user management systems such as LDAP(Lightweight Directory Access Protocol) and Active Directory

  * User authorization and access control
    * Role-based access control
      * In role base access control you authorize user by assigning priviliages to roles and assign roles to users and groups.

    ![Role-based access control](https://www.elastic.co/guide/en/elastic-stack-overview/current/security/authorization/images/authorization.png)

    * Attribute-based access control
      * X-Pack provide attribute base access mechanism, Which allow you to use attribute to restict access to documents. For example you can assign attributes to user and then implement the access policy in an role definition.

  * Node/client authentication and channel encryption
    * X-Pack security supports configuring SSL/TLS for securing the communication channels
    * Certificate based node authentication
    * X-Pack security also enables you to configure IP Filters which can be seen as a light mechanism for node/client authentication.
    * To access control feature that allows or rejects hosts, domains, or subnets.

  * Auditing
    * Audit trails log various activities/events that occur in the system. (e.g. security breach)
    * Which accounts for the type of events that are logged
    * These events include failed authentication attempts, user access denied, node connection denied, and more.


## Pricing

1. [Amazon Elasticsearch Service pricing](https://aws.amazon.com/elasticsearch-service/pricing/)

2. [Elasticsearch Service pricing](https://cloud.elastic.co/pricing)


## Management effort

### AWS manage ES

1. Good for those with no Elasticsearch experience or limited resources, easy to add and remove nodes from a cluster.

2. Change the instance sizes of your nodes from a drop-down. You get a semi-useful dashboard of metrics.

3. When nodes go down, they are automatically brought back up.

4. Support for in-place Elasticsearch upgrades for domains(automated). Take a snapshot of cluster before proceding. It also preserves the same domain endpoint URL. You dont have to:
	* Create manual snapshots 
	* Create a new domain for the targeted version
	* Restore the manual index snapshot to the new domain
	* Adjust your application configuration

5. You cannot use automated snapshots to migrate to new domains. Automated snapshots are read-only from within a given domain For migrations, you must use manual snapshots stored in your own repository. Standard S3 charges apply to manual snapshots.

6. There are tons of things that can cause it to become unstable, most of which are related to query patterns: 
  	* The documents being indexed, 
  	* The number of dynamic fields being created,
  	* Imbalances in the sizes of shards, 
  	* The ratio of documents to heap space, etc,
  	* Limited to clusters of 20 nodes, limited scalability.

  Diagnosing these problems is a bit of an art, and one needs a lot of metrics, log files and administrative APIs to drill down and find the root cause of an issue. AWS’s Elasticsearch doesn’t provide access to any of those things, leaving you no other option but to contact AWS’s support team.

### ES manage

1. Comparing to AWS ES: 
  * Elasticsearch can be deploy in less then 5 minutes. They have simple and straightfarword interface. 
  * One can easily decide about the instance by using [Elasticsearch Service pricing calculator](https://cloud.elastic.co/pricing). You can easily change the instance type and calculate the price at the same time.
  * Elasticsearch uses Rolling Upgrade to upgrade one node at a time so the upgrading does not interrupt the serivce.


## Feature set

### AWS manage ES

1. You get automatic and manual snapshots(recover or create a new domain).
	* Retain them for 14 days.
	* Elasticsearch snapshot APIs to create additional manual snapshots.
	* Automated snapshots are stored free of charge in Amazon S3.
	* Manual snapshots will incur standard Amazon S3 [usage charges](https://aws.amazon.com/s3/pricing/).

2. Plugins are automatically deployed and managed for you.

3. Monitoring and domain performance metrics:
	* Domain health,
	* Searchable documents,
	* Amazon EBS metrics, 
	* CPU, memory, and disk utilization for data and master nodes through Amazon CloudWatch

### ES manage

1. You have bigger clusters, less cost, more control over settings (index, cluster), more instance types and sizes available, Cloudwatch monitoring included

2. On demand equivalent instances are cheaper by 29%. but it vary from instance to instance. You can have even more discount if you use reserve instances.

3. You can use bigger i2 instances than AWS Elasticsearch, and you have access to the latest generation of c4 and m4 instances. This way you can scale further.

4. Using ES you have more control on index settings, beyond analysis and shards. 
	* Specially Delay allocation when you have a lot of data per node. 
	* You can also change the setting of all the indices at onces. 
	* If you propertly utalize the ES setting you can better optamize different use case as you have more control on the resources as compare to AWS.
	* You can change cluster wide settings, e.g. number of shard to rebalance at once.
	* You also get access to all other API such as Hot Threads which is useful for debugging. 

5. You can use a more comprehensive Elasticsearch monitoring solution. Currently CloudWatch only collect few matrics e.g. cluster status, number of nodes and documents, heap pressure and disk space. For more use case you need more information such as query latency and indexing throughput.


## Available/possible plugins

### AWS ES 

| Elasticsearch Version     | Plugins                          |
|---------------------------|----------------------------------|
| 6.3.                      | ICU Analysis              		   |
|                  		      | Ingest Attachment Processor      |
|                 			    | Ingest User Agent Processor      |
|       					          | Japanese (kuromoji) Analysis     |
|        		                | Mapper Murmur3                   |
|          		              | Mapper Size                      |
|         				          | Phonetic Analysis                |
|       			              | Smart Chinese Analysis           |
|      				              | Stempel Polish Analysis          |
|    					            	| Ukrainian Analysis   			       |
|   					             	| Seunjeon Korean Analysis         |

[List of AWS ES Plugins](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-plugins.html)

### ES plugins

| Elasticsearch Version     | Plugins                          |
|---------------------------|----------------------------------|
| 6.4. Core	                | Alerting	 	               		   |
|                  		      | ICU Analysis   				           |
|                 			    | Japanese (kuromoji) Analysis     |
|       					          | Korean (nori) Analysis  		     |
|        		                | Phonetic Analysis                |
|          		              | Smart Chinese Analysis           |
|         				          | Stempel Polish Analysis          |
|       			              | Ukrainian Analysis       		     |
|      				              | EC2 Discovery       			       |
|    						            | Azure Classic Discovery  		     |
|   						            | GCE Discovery      			         |
|   						            | File-Based Discovery Plugin      |
|   						            | Ingest Attachment Processor      |
|   						            | Ingest Geoip Processor   	       |
|   						            | Ingest user agent processor      |
|   					             	| Management(X-Pack)               |
|   					             	| Mapper Size  				             |
|   						            | Mapper Murmur3  		             |
|   						            | Security(X-Pack)  			         |
|   						            | S3 Repository  			   	         |
|   						            | Hadoop HDFS Repository  	       |
|   						            | Google Cloud Storage Repository  |
|   						            | Store SMB 			           		   |


List of all plugins and integration for version 6.4, It also include community plugins and external tools that make it easier to work with ES:
[Elasticsearch Plugins](https://www.elastic.co/guide/en/elasticsearch/plugins/6.4/index.html)



|                                        | Amazon Elasticsearch Service     | Elastic Elasticsearch Service    |
|----------------------------------------|----------------------------------|----------------------------------|
| Current Version                        | 6.3                              | 6.4
| Elastic Stack - Open Source Features   | Some                             | All
| Same Day Elastic Stack Version Release | No                               | Yes
| X-Pack (Elastic Commercial Plugins)    | No                               | Yes, Security, Alerting, Monitoring, Graph, Reporting, Machine Learning
| Elastic Technical Support              | No                               | Yes, Elastic Cloud Subscriptions
| One-Click Upgrades                     | Yes                              | Yes
| Hot-Warm Deployment Template           | No                               | Yes
| Underlying Cluster Hardware            | Single instance type for all roles | Multiple instance types (via deployment templates) |
| Default Snapshots                      | 1 time per day                   | 48 times per day Every 30 minutesStored for 48 hours
| Instant Rollout of Elasticsearch and Kibana Security Patches | No         | Yes
| Custom Plugin Support                  | Not supported                    | Supported
| Java Transport Client                  | Not supported                    | Supported
| Cross Zone Replication                 | Support for up to 2 availability zones | Support for up to 3 availability zones
| SLA-Based Support                      | General level support, not specific to AWS ES | Yes
| Uptime SLA                             | No                               | Yes, 99.95% cluster uptime in a given month as long as the cluster is deployed across 2 or more zones
| Elastic Maps Service                   | Does not work out of the box     | Yes
| Security                               | Only perimeter-level security and standard IAM policies  | <ul><li>Transport encryption</li><li>Authentication</li><li>Role-based access control</li><li>Field- and document-level security </li><li>Encryption at rest</li></ul>
| Alerting                          | Need to build and manage your own system to create alerting functionalities. This depends on Amazon Cloudwatch, which comes with predefined, simple metrics. If you want something more sophisticated, or related to your data, you'll need to build a custom metric and alerts. | <ul><li>Allows you to create scheduled queries, conditions, and actions on your data in Elasticsearch.</li><li> UI to create, manage, and take actions on alerts.</li></ul> |
| Monitoring                          | Depends on Amazon Cloudwatch, that covers a few metrics including cluster state, node information, etc.  | Feature-rich and complete monitoring product specifically designed for Elasticsearch and Kibana. <ul><li>Captures a wider range of metrics including search/index rate and latency, garbage collection count and duration, thread pool bulk rejection/queue, Lucene memory breakdown, and more with 10-second data granularity to ensure that clusters are running healthy.</li><li>Robust tools to diagnose, troubleshoot and keep your cluster healthy including automatic alerts on cluster issues.</li></ul>
| Graph                          | No                    | Yes
| Reporting                      | No                    | Yes




## Reference:

1. [In-place version upgrades for Amazon Elasticsearch Service](https://aws.amazon.com/blogs/database/in-place-version-upgrades-for-amazon-elasticsearch-service/)

2. [Amazon Elasticsearch Service Access Control](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-ac.html)

3. [Identity-Based Policies and Resource-Based Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_identity-vs-resource.html)

4. [Amazon Cognito for Kibana access control](https://aws.amazon.com/blogs/database/get-started-with-amazon-elasticsearch-service-use-amazon-cognito-for-kibana-access-control/)

5. [Hosted Elasticsearch Services Roundup: Elastic Cloud and Amazon Elasticsearch Service](https://www.elastic.co/blog/hosted-elasticsearch-services-roundup-elastic-cloud-and-amazon-elasticsearch-service)

6. [Elasticsearch Service pricing calculator](https://cloud.elastic.co/pricing)

