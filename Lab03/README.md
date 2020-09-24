# Kafka Lab 3 - Produce and Consume Messages to/from a Kafka Stream with Spring Boot Application

The Spring Framework supports many dependencies. In this lab, you create a sample Spring Boot application to produce and consume Kafka Streams with the following dependencies:

* `web` to build web, including RESTful, applications using Spring MVC, uses Apache Tomcat as the default embedded container.
* `data-rest` to expose Spring Data repositories over REST via Spring Data REST. 
* `kafka` to publish, subscribe, store, and process streams of records.
* `kafka-streams` to build stream processing applications with Apache Kafka Streams.

The Spring Boot CLI should already be installed in your web-terminal. It resides in the directory `/userdata/spring-2.2.0.RELEASE`.


## Create a Spring App

To create a Spring application,

1. Navigate to the `/userdata` folder.

	```shell
	$ cd ~
	```

1. Review a complete list of supported dependencies.

	```shell
	$ spring init --list
	```

1. Create a new Spring Boot application with dependencies for `web`, `data-rest`, `kafka`, and `kafka-streams`.

	```shell
	$ spring init --dependencies=web,data-rest,kafka,kafka-streams spring-boot-kafka-app

	Using service at https://start.spring.io
	Project extracted to '/userdata/spring-boot-kafka-app'
	```

1. Navigate to the Spring project folder.

	```shell
	$ cd spring-boot-kafka-app/
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

1. Extract the following information from the service credential of your Event Streams Service.
	* `username` (always set to `token`)
	* `password`
	* `kafka_brokers_sasl` (everything within the `[ ..... ]`)

1. Create src/main/resources/application.properties file.

	* Open the file src/main/resources/application.properties in `vi` file editor.

    	```console
	    $ vi src/main/resources/application.properties
    	```

	* Press the 'i' key to enable INSERT mode.

	* Add the following common Spring Boot properties to configure Kafka.

		```text
		# Spring server config
		server.port=8080
		spring.application.name=spring-boot-kafka-app

		# Kafka connection
		spring.kafka.jaas.enabled=true
		spring.kafka.jaas.login-module=org.apache.kafka.common.security.plain.PlainLoginModule
		spring.kafka.jaas.options.username=token
		spring.kafka.jaas.options.password=<password>
		spring.kafka.bootstrap-servers=<brokerlist>
		spring.kafka.properties.security.protocol=SASL_SSL
		spring.kafka.properties.sasl.mechanism=PLAIN
		spring.kafka.ssl.protocol=TLSv1.2

		# Kafka Producer
		spring.kafka.template.default-topic=greetings
		spring.kafka.producer.client-id=event-streams-kafka
		spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
		spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer

		# Kafka Consumer
		listener.topic=greetings
		spring.kafka.consumer.group-id=channel1
		spring.kafka.consumer.auto-offset-reset=earliest
		spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
		spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
		```

	* Press the ESC key to exit the INSERT mode.

	* Replace `<password>` with the `password` property from the Event Streams credential.
	
	* Replace `<brokerlist>` with the `kafka_brokers_sasl` property from the Event Streams credentials, **without [ ], quotes and newlines**.

	* Press the ESC key to exit the INSERT mode.
	
	* Type `:wq` to save the changes and exist 'vi' file editor.s

1. Create file src/main/java/com/example/springbootkafkaapp/EventsStreamController.java.

	* Open the file src/main/java/com/example/springbootkafkaapp/EventsStreamController.java in `vi` file editor.

	    ```console
    	$ vi src/main/java/com/example/springbootkafkaapp/EventsStreamController.java
    	```
  
	* Press the 'i' key to enable INSERT mode.

	* Add the following code,

		```java
		package com.example.springbootkafkaapp;

		import org.apache.kafka.clients.consumer.ConsumerRecord;
		import org.slf4j.Logger;
		import org.slf4j.LoggerFactory;
		import org.springframework.beans.factory.annotation.Autowired;
		import org.springframework.kafka.annotation.KafkaListener;
		import org.springframework.kafka.core.KafkaTemplate;
		import org.springframework.web.bind.annotation.GetMapping;
		import org.springframework.web.bind.annotation.PathVariable;
		import org.springframework.web.bind.annotation.RestController;

		import java.util.List;
		import java.util.concurrent.CopyOnWriteArrayList;

		@RestController
		public class EventsStreamController {
	
			@Autowired
			private KafkaTemplate<String, String> template;
	
			private List<String> messages = new CopyOnWriteArrayList<>();

			@KafkaListener(topics = "${listener.topic}", groupId = "channel1")
			public void listen(ConsumerRecord<String, String> cr) throws Exception {
				messages.add(cr.value());
			}

			@GetMapping(value = "/send/{msg}")
			public void send(@PathVariable String msg) throws Exception {
				this.template.sendDefault(msg);
			}

			@GetMapping("/received")
			public String recv() throws Exception {
				String result = messages.toString();
				messages.clear();
				return result;
			}
		}
		```

	* Press the ESC key to exit the INSERT mode
	
	* Type `:wq` to save the changes and quit 'vi' file editor.

	* For the original code example see the Spring Boot guide's an Even Quicker with Spring Boot example.


## Complie and Run the Spring Boot Application

To complie and run the application,

1. Clean and Install the application.

	```console
	$ mvn clean install
	```

1. Run the application.

	```console
	$ mvn spring-boot:run
	```


## Verificaton

To verify,

1. Open a new web-terminal in a new browser tab.

1. Produce messages.

	```console
	$ curl -X GET http://localhost:8080/send/Hello1
	```

1. Consume messages.

	```console
	$ curl -X GET http://localhost:8080/received
	[Hello1]
	```
	> Note: When you call the /received endpoint for the first time, it will retrieve ALL previous messages. For the subsequent calls, you only retrieve the new messages as the old messages were `cleaned up` when the first call was made.

1. Repeat the /send and /received calls for additional verification.

