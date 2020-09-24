# Kafka Lab 4 - Kafka Admin API

You explore IBM Event Streams Administration REST API in this sesson.

The Swagger 2.0 docs for the IBM Event Streams Administration REST API is found at https://raw.githubusercontent.com/ibm-messaging/event-streams-docs/master/admin-rest-api/admin-rest-api.yaml. You can load the file via URL import in the http://editor.swagger.io, click File > Import URL > OK. Or read the README for the Admin API at https://github.com/ibm-messaging/event-streams-docs/tree/master/admin-rest-api.


## Retrieve Property `kafka_admin_url`

The property `apikey` and `kafka_admin_url` of the service credential of your Event Streams Service are required to call the Kafka Admin API.

1. Login to your web terminal.

1. Retrieve the service credential of your Event Streams Service by the credential name. The command returns everything of the service credential of your Event Streams Service. For example, if you have `Service credentail-1` as your credential name, the following sample command retrieves your credential.

	```shell
	$ ibmcloud resource service-key "Service credentials-1"

	Retrieving service key account-eventstreams-user8888-credentials1 in resource group workshop-nov2019 under account Account as me@email.com...
                  
	Name:          account-eventstreams-user8888-credentials1   
	ID:            crn:v1:bluemix:public:messagehub:us-south:a/accf1c23dd456789befedcd0cda123e4:56ce78aa-d9a0-1c23-34ce-5a6cf7bd8d90:resource-key:1fe2ad34-5678-90fe-12d3-d4567d890c12   
	Created At:    Thu Oct 31 03:18:44 UTC 2019   
	State:         active   
	Credentials:                                   
               api_key:                  someapikey      
               apikey:                   someapikey      
               iam_apikey_description:   Auto-generated for key someapikey     
               iam_apikey_name:          account-eventstreams-user8888-credentials1      
               iam_role_crn:             crn:v1:bluemix:public:iam::::serviceRole:Manager      
               iam_serviceid_crn:        crn:v1:bluemix:public:iam-identity::a/accf1c23dd456789befedcd0cda123e4::serviceid:ServiceId-123456789      
               instance_id:              56ce78aa-d9a0-1c23-34ce-5a6cf7bd8d90      
               kafka_admin_url:          https://a12bcdefg3hij45.svc01.us-south.eventstreams.cloud.ibm.com      
               kafka_brokers_sasl:       [broker-1-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams. cloud.ibm.com:9999,
				broker-2-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999,
				broker-3-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999,
				broker-4-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999,
				broker-5-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999,
				broker-6-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999]      
               kafka_http_url:           https://a12bcdefg3hij45.svc01.us-south.eventstreams.cloud.ibm.com      
               password:                 123abc4567sdagdh2678akd7890hh      
               user:                     token 
	```

	> Note: your service credential of your Event Streams Service is also available in IBM Cloud console.

1. Extract the following information from the service credential of your Event Streams Service.
	* `apikey`
	* `kafka_admin_url`


## Authenticate of Kafka Admin API

To authenticate, add the X-Auth-Token HTTP Header with the apikey found in the service credentials.


## Call Kafka Admin API

To call the Kafka Admin API, you need `kafka_admin_url` and the `apikey` from the `Service Credentials`. 

In the examples below, replace the placeholders for these properties with the values from the `Service Credentials` of your Event Streams service,

1. List the Kafka topics.
  
	```console
	$ curl -X GET https://<kafka_admin_url>/admin/topics -H 'X-Auth-Token: <apikey>'

	[
		{
			"name": "greetings",
			"partitions": 1,
			"replicationFactor": 3,
			"retentionMs": 86400000,
			"cleanupPolicy": "delete",
			"configs": {
				"cleanup.policy": "delete",
				"min.insync.replicas": "2",
				"retention.bytes": "104857600",
				"retention.ms": "86400000",
				"segment.bytes": "536870912"
			},
			"replicaAssignments": [
				{
					"id": 0,
					"brokers": {
						"replicas": [
							0,
							1,
							2
						]
					}
				}
			]
		}
	]
	```

1. Delete a topic.
  
	```console
	$ curl -X DELETE https://<kafka_admin_url>/admin/topics/greetings -H 'X-Auth-Token: <apikey>'

	Create a topic,
	$ curl -X POST \
	https://<kafka_admin_url>/admin/topics \
	-H 'X-Auth-Token: <apikey>' \
	-d '{
	"name": "greetings",
	"partitions": 1
	}'
	```

1. Retrieve a topic.

	```console
	$ curl -X GET https://<kafka_admin_url>/admin/topics/greetings -H 'X-Auth-Token: <apikey>'
	```

1. Create a new topic called `greetings` with 1 partition

	```console
	$ ibmcloud es topic-create greetings --partitions 1

	Created topic greetings
	OK
	```

1. Update a topic configuration,

	```console
	$ curl -X PATCH https://<kafka_admin_url>/admin/topics/greetings -H 'X-Auth-Token: <apikey>' -d '{"new_total_partition_count": 2 }'
	```

	> Note: The above command fails if you run a Lite instance of `Event Streams` service. For a Lite instance of `Event Streams` service, the maximum number of partitions allowed is 1.


