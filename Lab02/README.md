# Kafka Lab 2 - Produce and Consume Streams with Kafka CLI Tools

You learn how to produce and consume messages via Apache Kafka CLI tool in the session.

Also see https://ibm.github.io/event-streams/tutorials/kafka-streams-app/


## Configure the Apache Kafka CLI

Kafka CLI version 2.3.0 has been downloaded and installed in the web terminal environment. The Apache Kafka console tools ship with the Apache Kafka distribution and can be found in the bin directory of the Kafka download. 

> Note: If you have already downloaded Kafka, skip to the Lab section.

1. Login to your web terminal.

1. Go to the Kafka distribution directory which is in folder `/userdata/kafka_2.12-2.3.0`.

	```console
	$ cd ~/kafka_2.12-2.3.0
	```

1. Locate your Event Streams Service.

	```console
	$ ibmcloud resource service-instances | "grep $ES_SVC_NAME"
	
	$ ibmcloud resource service-instance "$ES_SVC_NAME"
	```
	
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

1. Store the entire contents of the service credential of your Event Streams Service in a file as you need the information for a few times later.

1. Extract the following information from the service credential of your Event Streams Service.
	* `username` (always set to `token`)
	* `password`
	* `kafka_brokers_sasl` (everything within the `[ ..... ]`)

1. Create environment variable `$KAFKA_BROKERS_SASL` and set it to the `kafka_brokers_sasl` property in the service credential of your Event Streams Service. This environment variable helps make it easy to refer to the `kafka_brokers_sasl` property as it's very long. 

	> Note: Set the variable `$KAFKA_BROKERS_SASL` to the complete Array value of of the `kafka_brokers_sasl` property in the service credential, **excluding the square brackets** and enclosed by double quotes. **Separate `hostname:post` with `,`**.

	```shell
	$ export KAFKA_BROKERS_SASL="broker-1-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams. cloud.ibm.com:9999,broker-2-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999,broker-3-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999,broker-4-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999,broker-5-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999,broker-6-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999"
	```

1. Create new Kafka configuration file `mykafka.properties`.

    ```console
    $ vi mykafka.properties
    ```

	* Press the 'i' key to enable INSERT mode.

	* Copy and paste the following properties into the `mykafka.properties` file.

		```text
		sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="token" password="PASSWORD";
		security.protocol=SASL_SSL
		sasl.mechanism=PLAIN
		ssl.protocol=TLSv1.2
		ssl.enabled.protocols=TLSv1.2
		ssl.endpoint.identification.algorithm=HTTPS
		```

	* Press the ESC key to exit INSERT mode.

	* Change `PASSWORD` to the password value extracted from the service credential of your Event Streams Service.

	* Your `mykafka.properties` file should be similar to the sample file below.

		```
		sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="token" password="neOu63YFtp35vD0AKkrPKRpC284iriasYoKhWmamRj5e";
		security.protocol=SASL_SSL
		sasl.mechanism=PLAIN
		ssl.protocol=TLSv1.2
		ssl.enabled.protocols=TLSv1.2
		ssl.endpoint.identification.algorithm=HTTPS
		```

	* Press the ESC key one more time.

	* Type `:wq` to save the file and exit the `vi` editor.


## Produce Messages

Now you are ready to run the producer to publish messages to the Kafka messages stream.

1. Start the producer.

	```console
	$ bash bin/kafka-console-producer.sh --broker-list $KAFKA_BROKERS_SASL --producer.config mykafka.properties --topic greetings

	>
	```

1. Once the producer is running, you can enter messages and publish to the Kafka event stream that will be consumed in real time. The prompt symbol is displayed on the last line when the producer is ready.

1. Place messages on the event stream by entering messages at the prompt. Hit ENTER to send the current message and start a new one.

	```shell
	> hello1
	> hello2
	> hello3
	> we could go on forever and ever and ever
	```

1. Hit `CTRL-C` to stop the producer.


## Consume Messages

1. Start the consumer.

	```console
	$ bash bin/kafka-console-consumer.sh --bootstrap-server $KAFKA_BROKERS_SASL --consumer.config mykafka.properties --topic greetings --from-beginning
	```

1. The consumer listener retrieves the messages from the event stream of topic greetings, 

	```console
	hello1
	hello2
	hello3
	we could go on forever and ever and ever
	```

1. Hit `CTRL-C` to stop the consumer,


## Consumer Group

Consumers can be labeled with a consumer group name, so that each record published to a topic is delivered to one consumer instance within a subscribing consumer group.

1. In the web terminal, execute

	```shell
	$ echo $KAFKA_BROKERS_SASL
	
	broker-1-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams. cloud.ibm.com:9999,broker-2-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999, broker-3-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999, broker-4-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999, broker-5-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999, broker-6-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999
	```

1. Copy the output to the clipboard.

1. Open 3 new web terminal sessions.

1. In each new terminal, you need to set the $KAFKA_BROKERS_SASL variable.

	```shell
	$ cd ~/kafka_2.12-2.3.0

	$ export KAFKA_BROKERS_SASL="broker-1-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams. cloud.ibm.com:9999,broker-2-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999, broker-3-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999, broker-4-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999, broker-5-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999, broker-6-a1bc2d3efg4hijkl.kafka.svc01.us-south.eventstreams.cloud.ibm.com:9999"
	```

1. In the first 2 new terminals, start a consumer instance that is a member of group 1 by adding a `--group 1` flag to label a consumer.

	```console
	$ bash bin/kafka-console-consumer.sh --bootstrap-server $KAFKA_BROKERS_SASL --consumer.config mykafka.properties --topic greetings --group 1
	```

1. In the third new terminal, start a consumer instance that is a member of group 2 by adding a `--group 2` flag to label a consumer.

	```console
	$ bash bin/kafka-console-consumer.sh --bootstrap-server $KAFKA_BROKERS_SASL --consumer.config mykafka.properties --topic greetings --group 2
	```

1. Go back to the original terminal.

1. Start the producer,

	```shell
	$ bash bin/kafka-console-producer.sh --broker-list $KAFKA_BROKERS_SASL --producer.config mykafka.properties --topic greetings
	>
	```

1. Publish a new message to the topic `greetings`.

	```shell
	> listen carefully, i will say this only once
	```

1. Check the 3 consumers in the 3 new web-terminal. Only 1 consumer in group 1 and the single consumer in group 2, consumed the message once per group.

	```shell
	listen carefully, i will say this only once
	```

	
