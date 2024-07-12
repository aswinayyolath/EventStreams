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
