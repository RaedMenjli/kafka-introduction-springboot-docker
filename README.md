##  Installing Kafka, Creating Topics, and Managing Partitions with Kafka (Spring Boot Edition)

###  **Objective**
The goal of this guide is to install Kafka, create topics, send and consume messages via the CLI, and explore partition management to observe message distribution.

---

###  **Prerequisites**
- Docker and Docker Compose installed
- Java 17+ installed
- Maven installed
- An IDE (IntelliJ IDEA, VS Code, or Eclipse)
- Apache Kafka
- Spring Boot project setup

---

### 1️⃣ **Create the Spring Boot Project**

- Go to Spring Initializr and generate a project with the following setup:
  - **Group:** com.kafka
  - **Artifact:** kafka-introduction-springboot-docker
  - **Dependencies:** Spring Web, Spring for Apache Kafka 

- Download, extract the project, and open it in your IDE.

---

### 2️⃣ **Setting Up the Docker Environment**
Create a `docker-compose.yml` file:

```yaml
version: '3.8'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"

  kafka:
    image: wurstmeister/kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
```

Start the services with:

```bash
docker-compose up -d
```

Check that the containers are running:

```bash
docker ps
```

---

### 3️⃣ **Create a Topic and Send Messages**

#### **Create a topic**

Create a topic named `demo-topic`:

```bash
docker exec -it <container_kafka> kafka-topics.sh \
  --create \
  --topic demo-topic \
  --bootstrap-server localhost:9092 \
  --partitions 1 \
  --replication-factor 1
```

#### **Produce messages**

Start a producer and send messages:

```bash
docker exec -it <container_kafka> kafka-console-producer.sh \
  --broker-list localhost:9092 \
  --topic demo-topic
```

Type some messages:

```
> Kafka is awesome!
> Let's test the consumer!
```

#### **Consume messages**

Open a consumer:

```bash
docker exec -it <container_kafka> kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic demo-topic \
  --from-beginning
```

You should see:

```
Kafka is awesome!
Let's test the consumer!
```

---

### 4️⃣ **Create Multiple Partitions and Observe Message Distribution**

#### **Create a topic with multiple partitions**

Let's create a new topic with 3 partitions:

```bash
docker exec -it <container_kafka> kafka-topics.sh \
  --create \
  --topic multi-partitions-topic \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1
```

#### **Send messages to this topic**

```bash
docker exec -it <container_kafka> kafka-console-producer.sh \
  --broker-list localhost:9092 \
  --topic multi-partitions-topic
```

Send some messages:

```
> Message 1
> Message 2
> Message 3
> Message 4
```

#### **Observe message distribution**

Open 3 consumers (1 per partition):

```bash
docker exec -it <container_kafka> kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic multi-partitions-topic \
  --partition 0 --from-beginning

# For partition 1
# Replace --partition 0 with --partition 1

# For partition 2
# Replace --partition 0 with --partition 2
```

You'll see that each partition gets a share of the messages.

---

### 5️⃣ **Integrate Kafka with Spring Boot**

#### **Add dependencies to `pom.xml`**

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.kafka</groupId>
  <artifactId>spring-kafka</artifactId>
</dependency>
```

#### **Create a producer service**

```java
@Service
public class KafkaProducerService {
    private final KafkaTemplate<String, String> kafkaTemplate;

    public KafkaProducerService(KafkaTemplate<String, String> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void sendMessage(String message) {
        kafkaTemplate.send("demo-topic", message);
    }
}
```

#### **Create a consumer service**

```java
@Component
public class KafkaConsumerService {
    @KafkaListener(topics = "demo-topic", groupId = "group_id")
    public void consume(String message) {
        System.out.println("Consumed message: " + message);
    }
}
```
 
---
### 6️⃣ **Testing and Sending Messages via Spring Boot**

#### **Create a Test Controller to Trigger Message Sending**

You can add an endpoint in your Spring Boot application to send messages via Kafka.

```java
@RestController
@RequestMapping("/kafka")
public class KafkaController {

    private final KafkaProducerService producerService;

    public KafkaController(KafkaProducerService producerService) {
        this.producerService = producerService;
    }

    @PostMapping("/send")
    public ResponseEntity<String> sendMessage(@RequestParam String message) {
        producerService.sendMessage(message);
        return ResponseEntity.ok("Message sent: " + message);
    }
}
```
 ### Test Sending Messages

Now you can test sending messages by calling the `/kafka/send` endpoint using Postman or cURL:

```bash
curl -X POST "http://localhost:8080/kafka/send?message=Hello%20Kafka"
```
This will trigger the producer to send the message to Kafka.

 ### Testing Consumer
To test the consumer, you can simply run your Spring Boot application and watch the logs. The consumer service will automatically listen for new messages and log them.

