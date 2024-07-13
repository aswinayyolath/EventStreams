# Understanding Schemas and Schema Registry in Event Streams
In Apache Kafka, schemas play a crucial role in ensuring data consistency and structure across messages exchanged between producers and consumers. While Kafka itself can handle any data, schemas define the specific format and structure of the data within messages, ensuring that both producers and consumers interpret data correctly.

## Why Kafka Needs Schemas
Schemas provide a predefined structure for data in Kafka messages, facilitating efficient data handling and interpretation. They specify the fields and their types, ensuring consistency across data exchanges.

## What is a Schema Registry?
The Apicurio Registry, integrated into IBM Event Streams, serves as an open-source schema registry for managing and versioning schemas. It stores schemas in Kafka's internal topics, providing versioned history and an interface for retrieving schemas.

## Apicurio Schema Registry in Event Streams
Each Event Streams cluster hosts its own instance of the Apicurio Registry, enabling producers and consumers to validate data against registered schemas. This ensures data conformity without needing to transmit schemas within messages, thus optimizing message size and network usage.

## Using Apicurio with IBM Event Streams

![alt text](image-20.png)

Event Streams supports Apache Avro as the schema format, which offers efficient binary serialization. Producers serialize data according to a schema, embedding a unique identifier that references the schema in the registry. Consumers then use this identifier to fetch and deserialize data correctly.

## Serialization and Deserialization
Producers use serializers to encode data in Avro format, including the schema identifier. Consumers use deserializers to retrieve schemas from the registry and decode messages accordingly, ensuring data integrity and structure.

## Versions and Compatibility
Apicurio Registry manages schema versions, allowing schemas to evolve while maintaining backward compatibility. Producers and consumers can continue using older versions until they are updated to newer ones, preventing disruptions in data flow.

## Creating and Registering Schemas
Creating a schema for your Kafka messages ensures that both producers and consumers agree on the data structure. In IBM Event Streams, you can define and register schemas using the Apicurio Schema Registry. Here's a simplified process:

- **Define Your Schema**: Use Apache Avro to create a schema that specifies the structure of your messages, including fields and their data types.
- **Register the Schema**: Use the Event Streams UI or CLI to register your schema with the Apicurio Registry. This involves uploading the schema definition, which the registry then stores and assigns a unique identifier.

This process ensures that every message sent to a Kafka topic conforms to the defined schema, making data handling more efficient and reliable.

## Managing Schema Lifecycle
Managing the lifecycle of schemas involves creating, versioning, and deprecating schemas as your data requirements evolve. Here's a brief overview:

- **Versioning**: Whenever you update a schema, you create a new version in the Apicurio Registry. This ensures that older versions remain available for applications that still rely on them.
- **Compatibility**: The registry checks for compatibility between schema versions, ensuring that new versions do not break existing producers or consumers.
- **Deprecation**: Once a new schema version is stable and widely adopted, you can deprecate older versions. Deprecated schemas can still be used, but users are warned to update to the latest version.
- **Disabling and Deleting**: Eventually, you may disable or delete old schemas to streamline operations and reduce clutter in the registry.

## Configuring Apicurio Schema Registry

let's dive into a hands-on example to see schema registry with Event Streams in action.

In this section, we will create a Java application to produce and consume messages in Kafka, utilizing the Apicurio Schema Registry to manage and validate schemas. This step-by-step guide will walk you through setting up your project, writing the necessary code, and running the application.

### Step 1: Setting Up the Maven Project
First, we need to create a Maven project. Maven is a popular build automation tool used primarily for Java projects. It helps manage project dependencies and build processes.

### Create a new directory for your project

```sh
mkdir kafka-schema-registry-demo
cd kafka-schema-registry-demo
```

### Generate a Maven project
Use the following command to create a Maven project structure:

```bash
mvn archetype:generate -DgroupId=com.eventstreams.example -DartifactId=kafka-schema-registry -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

### Navigate to the project directory

```bash
cd kafka-schema-registry-demo
```

### Update the pom.xml file
Open the pom.xml file in your preferred text editor and add the necessary dependencies for Kafka, Avro, and Apicurio Schema Registry

```xml
<dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.kafka</groupId>
      <artifactId>kafka-clients</artifactId>
      <version>3.7.0</version>
    </dependency>
    <!-- Avro -->
    <dependency>
      <groupId>org.apache.avro</groupId>
      <artifactId>avro</artifactId>
      <version>1.10.2</version>
    </dependency>
    <!-- IBM Event Streams Schema Registry Client -->
    <dependency>
      <groupId>io.apicurio</groupId>
      <artifactId>apicurio-registry-serdes-avro-serde</artifactId>
      <version>2.5.11.Final</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>1.7.26</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-simple</artifactId>
      <version>1.7.26</version>
    </dependency>
</dependencies>
```

### Writing the Producer Code
Now, let's write the Java code to produce messages to a Kafka topic using a defined schema. We'll create a schema for our message and register it with Apicurio Schema Registry.

```java
package com.eventstreams.example;

import org.apache.kafka.clients.CommonClientConfigs;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.config.SaslConfigs;
import org.apache.kafka.common.config.SslConfigs;
import org.apache.kafka.common.serialization.StringSerializer;

import io.apicurio.registry.serde.SerdeConfig;
import io.apicurio.registry.serde.avro.AvroKafkaSerdeConfig;
import io.apicurio.registry.serde.avro.AvroKafkaSerializer;
import io.apicurio.rest.client.config.ApicurioClientConfig;

import org.apache.avro.generic.GenericData;
import org.apache.avro.generic.GenericRecord;
import org.apache.avro.Schema;
import java.io.File;
import java.util.Properties;

public class Producer {

    public static void main(String[] args) throws Exception {
        String user = "<SCRAM USERNAME>";
        String userPassword="<SCRAM USER PASSWORD>";
        String topic = "<TOPIC NAME>";
        String schemaPath="<SCHEMA PATH>";
        String REGISTRY_URL= "<SCHEMA REGISTRY URL>";
        String BOOTSTRAP_SERVERS_CONFIG = "<BOOTSTRAP_SERVER URL>";
        String TRUSTSTORE_LOCATION = "<TRUSTSTORE_LOCATION>";
        String TRUSTSTORE_PASSWORD="<TRUSTSTORE_PASSWORD>";

        Properties props = new Properties();

        // SSL Configuration
        props.put(SslConfigs.SSL_PROTOCOL_CONFIG, "TLSv1.2");
        props.put(SslConfigs.SSL_TRUSTSTORE_LOCATION_CONFIG, TRUSTSTORE_LOCATION);
        props.put(SslConfigs.SSL_TRUSTSTORE_PASSWORD_CONFIG, TRUSTSTORE_PASSWORD);
        
        // SASL Configuration
        props.put(CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, "SASL_SSL");
        props.put(SaslConfigs.SASL_MECHANISM, "SCRAM-SHA-512");
        String saslJaasConfig = String.format("org.apache.kafka.common.security.scram.ScramLoginModule required username=\"%s\" password=\"%s\";", user, userPassword);
        props.put(SaslConfigs.SASL_JAAS_CONFIG, saslJaasConfig);
        
        // Apicurio Registry Configuration
        props.put(SerdeConfig.AUTH_USERNAME, user);
        props.put(SerdeConfig.AUTH_PASSWORD, userPassword);
        props.put(ApicurioClientConfig.APICURIO_REQUEST_TRUSTSTORE_LOCATION, TRUSTSTORE_LOCATION);
        props.put(ApicurioClientConfig.APICURIO_REQUEST_TRUSTSTORE_PASSWORD, TRUSTSTORE_PASSWORD);
        props.put(ApicurioClientConfig.APICURIO_REQUEST_TRUSTSTORE_TYPE, "PKCS12");
        props.put(SerdeConfig.REGISTRY_URL, REGISTRY_URL);

        // Kafka Producer Configuration
        props.put(CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS_CONFIG);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, AvroKafkaSerializer.class.getName());
        props.put(AvroKafkaSerdeConfig.AVRO_ENCODING, AvroKafkaSerdeConfig.AVRO_ENCODING_JSON);
        props.putIfAbsent(SerdeConfig.AUTO_REGISTER_ARTIFACT, true);

        // Initialize Kafka Producer
        try (KafkaProducer<String, GenericRecord> producer = new KafkaProducer<>(props)) {
            // Load Avro Schema
            Schema.Parser schemaDefinitionParser = new Schema.Parser();
            Schema schema = schemaDefinitionParser.parse(new File(schemaPath));

            // Create GenericRecord
            GenericRecord genericRecord = new GenericData.Record(schema);
            genericRecord.put("name", "Aswin");
            genericRecord.put("age", 20);

            // Create and Send Producer Record
            ProducerRecord<String, GenericRecord> producerRecord = new ProducerRecord<>(topic, genericRecord);
            producer.send(producerRecord);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### Writing the Consumer Code
Next, we'll write the Java code to consume messages from a Kafka topic and deserialize them using the schema from the Apicurio Schema Registry.

```java
package com.eventstreams.example;

import org.apache.kafka.clients.CommonClientConfigs;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.common.config.SaslConfigs;
import org.apache.kafka.common.config.SslConfigs;
import org.apache.kafka.common.serialization.StringDeserializer;

import io.apicurio.registry.serde.SerdeConfig;
import io.apicurio.registry.serde.avro.AvroKafkaDeserializer;
import io.apicurio.registry.serde.avro.AvroKafkaSerdeConfig;
import io.apicurio.rest.client.config.ApicurioClientConfig;

import org.apache.avro.generic.GenericRecord;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

public class Consumer {

    public static void main(String[] args) {
        String user = "<SCRAM USERNAME>";
        String userPassword="<SCRAM USER PASSWORD>";
        String topic = "<TOPIC NAME>";
        String REGISTRY_URL= "<SCHEMA REGISTRY URL>";
        String BOOTSTRAP_SERVERS_CONFIG = "<BOOTSTRAP_SERVER URL>";
        String TRUSTSTORE_LOCATION = "<TRUSTSTORE_LOCATION>";
        String TRUSTSTORE_PASSWORD="<TRUSTSTORE_PASSWORD>";

        Properties props = new Properties();

        // SSL Configuration
        props.put(SslConfigs.SSL_PROTOCOL_CONFIG, "TLSv1.2");
        props.put(SslConfigs.SSL_TRUSTSTORE_LOCATION_CONFIG, TRUSTSTORE_LOCATION);
        props.put(SslConfigs.SSL_TRUSTSTORE_PASSWORD_CONFIG, TRUSTSTORE_PASSWORD);

        // SASL Configuration
        props.put(CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, "SASL_SSL");
        props.put(SaslConfigs.SASL_MECHANISM, "SCRAM-SHA-512");
        String saslJaasConfig = String.format("org.apache.kafka.common.security.scram.ScramLoginModule required username=\"%s\" password=\"%s\";", user, userPassword);
        props.put(SaslConfigs.SASL_JAAS_CONFIG, saslJaasConfig);

        // Apicurio Registry Configuration
        props.put(SerdeConfig.AUTH_USERNAME, user);
        props.put(SerdeConfig.AUTH_PASSWORD, userPassword);
        props.put(ApicurioClientConfig.APICURIO_REQUEST_TRUSTSTORE_LOCATION, TRUSTSTORE_LOCATION);
        props.put(ApicurioClientConfig.APICURIO_REQUEST_TRUSTSTORE_PASSWORD, TRUSTSTORE_PASSWORD);
        props.put(ApicurioClientConfig.APICURIO_REQUEST_TRUSTSTORE_TYPE, "PKCS12");
        props.put(SerdeConfig.REGISTRY_URL, REGISTRY_URL);

        // Kafka Consumer Configuration
        props.put(CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS_CONFIG);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, AvroKafkaDeserializer.class.getName());
        props.put(AvroKafkaSerdeConfig.AVRO_ENCODING, AvroKafkaSerdeConfig.AVRO_ENCODING_JSON);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "my_consumer_group");
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

        // Initialize Kafka Consumer
        try (KafkaConsumer<String, GenericRecord> consumer = new KafkaConsumer<>(props)) {
            consumer.subscribe(Collections.singletonList(topic));

            while (true) {
                ConsumerRecords<String, GenericRecord> records = consumer.poll(Duration.ofMillis(100));
                for (ConsumerRecord<String, GenericRecord> record : records) {
                    System.out.printf("Consumed record: key = %s, value = %s%n", record.key(), record.value());
                }
            }
        }
    }
}

```

Schema (`user.avsc`) looks like

```json 
{
  "type": "record",
  "name": "User",
  "namespace": "com.eventstreams.example",
  "fields": [
    {"name": "name", "type": "string"},
    {"name": "age", "type": "int"}
  ]
}
```

## Schema Registry

<img width="1722" alt="image" src="https://github.com/user-attachments/assets/a60409d4-3428-4d8e-a937-cd8c45959e71">

## Produced messages can be viewed from the IBM Event Streams Topic page

<img width="1728" alt="image" src="https://github.com/user-attachments/assets/f028a134-8f21-4fdb-8878-926f61de8516">


In this part of the blog series, we've explored the importance of schemas in Apache Kafka, the role of the Apicurio Schema Registry, and how it integrates seamlessly with IBM Event Streams. We covered the process of producing and consuming messages with Avro schemas, ensuring data consistency and structure across Kafka communications. By leveraging Apicurio and Avro, you can efficiently manage your data schemas, handle serialization and deserialization, and ensure smooth versioning and compatibility for evolving data requirements.

In the next part, we'll dive into the details of installing your own CA certificates and private keys for securing your Event Streams cluster, enhancing the security and integrity of your Kafka environment.
