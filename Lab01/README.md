# Kafka Lab 1 - Setup

An instance of Apache Kafka in IBM Cloud is required during the lab. 

If you are doing the exercise in a workshop environment, an IBM Event Streams Service instance may have been created for you in IBM Cloud. An IBM Event Streams is a managed service of an Apache Kafka instance in IBM Cloud. It is a high-throughput message bus built on Apache Kafka, supporting all Kafka APIs and optimized for event ingestion and event stream distribution in IBM Cloud.


## Create an IBM Event Streams Service Instance

If an IBM Event Streams Service instance does not exist in your IBM Cloud account, you must create one before starting the exercise.

1. Login to IBM Cloud at https://cloud.ibm.com.

1. Click `Catalog` link on the top.

1. Search and select `Event Streams`.

1. Select a `Region` according to your location.

1. Select a `Plan`.

1. Remove any space in the instance name. Optionally, you may rename the instance name.

1. Click `Create` button at the bottom-right corner.

1. Navigate to `Service credential` tab after your `Event Stream` instance is created.

1. Click `New credential`.

1. Click `Add`. 

1. Expand the new `service credential`. Copy and save the credential contents for coming exercise.


## Install Kafka

To install `Kafka`,

1. Download `Kafka` package.

	```
	cd ~
	curl -L https://www-us.apache.org/dist/kafka/2.3.0/kafka_2.12-2.3.0.tgz -o kafka_2.12-2.3.0.tgz
	```

1. Unzip the `Kafka` package.

	```
	tar -xvf kafka_2.12-2.3.0.tgz  
	```

1. Cleanup.

	```
	rm kafka_2.12-2.3.0.tgz
	```


## Install the Event Streams Plugin

If you already have installed the Event Streams plugin and are logged into IBM Cloud, jump to "Initialize the Event Streams Service".

1. Login to your web terminal. Lab instructors should provide the access information.

	> Note: if you are doing the lab in a non-workshop environment, you may perform the steps in a command window or terminal. Pre-requisite CLIs are required to be installed locally.

1. Verify the Event Streams plugin

	The IBM Cloud Developer Tools CLI and the plugin for `Event Streams` are pre-installed in your web-terminal environment. To verify, run the commmand below to list the installed plugins.

	```
	$ ibmcloud plugin list

	Listing installed plug-ins...

	Plugin Name                            Version   Status   
	cloud-functions/wsk/functions/fn       1.0.35       
	cloud-object-storage                   1.1.0        
	container-registry                     0.1.437      
	container-service/kubernetes-service   0.4.42       
	dev                                    2.4.0        
	event-streams                          2.0.0
	```

	> Note: `event-streams` in the above sample output represents the the plugin for `Event Streams`.

1. Install the Event Streams plugin

	If for some reason, the Event Streams plugin is not listed, install it as follow,

	```shell
	$ ibmcloud plugin install event-streams

	Looking up 'event-streams' from repository 'IBM Cloud'...
	Plug-in 'event-streams 2.0.0' found in repository 'IBM Cloud'
	Attempting to download the binary file...
	30.95 MiB / 30.95 MiB [===========================================] 100.00% 17s
	32455568 bytes downloaded
	Installing binary...
	OK
	Plug-in 'event-streams 2.0.0' was successfully installed into /Users/user1/.bluemix/plugins/event-streams. 
	Use 'ibmcloud plugin show event-streams' to show its details.
	```


## Initialize the Event Streams Service Plugin

Before initializing the Event Streams plugin and connect to your Event Streams Service instance, you must be logged in to IBM Cloud.

1. Login to your IBM Cloud account. Enter your IBM Cloud account username and password when prompted.

    ```console
    $ ibmcloud login -r us-south -g Default -u <your IBM Cloud ID>
    ```

1.  Set CF API endpoint, Org and Space. If there are multiple choices, you'll be prompted.

	```console
	$ ibmcloud target --cf
	```

1. Set the environmentvariable `ES_SVC_NAME` pointing to your Event Streams Service instance in IBM Cloud. The `Event Streams Service Name` should be provided by the instructors for this workshop. 

	```shell
	$ export ES_SVC_NAME="<Event Streams Service Name>"

	$ echo $ES_SVC_NAME
	```

1. Initialize the Event Streams Service plugin.

	```console
	$ ibmcloud es init -i "$ES_SVC_NAME"

	API Endpoint: 	https://123abc4d5efgh67i.svc01.us-south.eventstreams.cloud.ibm.com
	OK
	```


## Basic Event Streams CLI Commands

You run couple of Kafka CLI commands in this section.

1. List `topics`

	```shell
	$ ibmcloud es topics

	OK
	No topics found.
	```
	
1. Create a new topic called `greetings` with 1 partition

	```console
	$ ibmcloud es topic-create greetings --partitions 1

	Created topic greetings
	OK
	```

	> Note: It may take a little while before the topic is created.

1. Display the topic details of `greetings`

	```shell
	$ ibmcloud es topic greetings

	Details for topic greetings
	Topic name   Internal?   Partition count   Replication factor   
	greetings    false       1                 3   

	Partition details for topic greetings
	Partition ID   Leader   Replicas   In-sync   
	0              2        [2 0 1]    [2 0 1]   

	Configuration parameters for topic greetings
	Name                  Value   
	cleanup.policy        delete   
	min.insync.replicas   2   
	segment.bytes         536870912   
	retention.ms          86400000   
	retention.bytes       104857600   
	OK
	```

Additional information of using the Event Streams CLI is available in [Lab05](../Lab05/README.md).


## Install Spring Boot,

To install Spring Boot,

1. Downbload Spring Boot package.

	```
	cd ~
	curl -L https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.2.0.RELEASE/spring-boot-cli-2.2.0.RELEASE-bin.tar.gz -o spring-boot-cli-2.2.0.RELEASE-bin.tar.gz 
	```

1. Unzip the Spring Boot package.

	```
	tar -xvf spring-boot-cli-2.2.0.RELEASE-bin.tar.gz && rm spring-boot-cli-2.2.0.RELEASE-bin.tar.gz
	```

1. Make the Spring Boot package accessible.

	```
	export PATH=$PATH:~/spring-2.2.0.RELEASE/bin
	spring --version
	```

## Install vi Editor

To install vi editor,

	```
	sudo apt-get update
	sudo apt-get install vim
	```



