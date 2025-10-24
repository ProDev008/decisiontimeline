
HR Performance Analytics Framework

Decision timelines for HR performance analysis related to employee onboarding — essentially, how long it takes before HR decides whether to onboard (hire permanently, extend probation, or reject) an employee, depending on performance insights.

Let’s break this down systematically for an HR Performance Analytics framework — covering the decision checkpoints across those timeframes.

Objective

Evaluate and decide whether to onboard, retain, promote, or release an employee based on quantifiable and behavioral performance metrics within defined timeframes.


Time-Based Decision Framework

Timeframe          HR Focus Area
  2 Weeks          Initial Fit & Adaptability
  1 Month          Learning Curve & Engagement
  3 Months         Probation Evaluation
  6 Months         Stabilization & Growth
  12 Months        Retention & Value Addition
  18 Months        Leadership Potential
  24 Months        Sustainability & Strategic Fit

  
Performance Metrics
     2 Weeks - Attendance & punctuality- Cultural adaptability- Training completion rate- Task response time- Peer feedback (initial collaboration)
     1 Month - Task quality & completion- Learning KPIs- Supervisor feedback- Work ethics & attitude- Communication & team behavior
     3 Months - Goal achievement % (vs. assigned)- Attendance & behavior trend- Quality vs. quantity ratio- Feedback 360 (manager, peers)- Self-assessment alignment
     6 Months - Productivity trend, Client feedback (if applicable), Innovation or initiative, Consistency in delivery, Engagement in team goals
     12 Months - KPI alignment with department, Mentorship participation, Career goal clarity, Organizational impact index, Performance rating average
     18 Months - Cross-functional contribution, Decision-making autonomy, Leadership behavior metrics, Peer mentoring, Upskilling participation
     24 Months - Long-term performance consistency, Employee satisfaction score, Attrition risk prediction, Innovation index, Cultural alignment score



Decision Actions
  2 Weeks - Continue onboarding- Extend training- Flag early risk (if red indicators appear)
  1 Month - Approve probation continuation- Trigger coaching plan- Evaluate role suitability
  3 Months - Confirm employment (onboard fully)- Extend probation- Discontinue onboarding
  6 Months - Onboard to long-term plan, Assign to independent role, Trigger performance improvement plan (if needed) 
  12 Months - Promote / Role expansion, Continue in role, Re-evaluate compensation & benefits
  18 Months - Prepare for leadership track, Evaluate for lateral or upward movement
  24 Months - Confirm for strategic/leadership role, Re-assess for new challenges, Plan retention incentives


  HR Performance Index (HPI) = 
(0.25 * Productivity Score) +
(0.20 * Learning & Adaptability) +
(0.20 * Collaboration & Communication) +
(0.15 * Attendance & Reliability) +
(0.10 * Manager Feedback) +
(0.10 * Self & Peer Review)


Then define thresholds:

HPI ≥ 80 → Confirm Onboarding

HPI 60–79 → Extend Probation / Mentorship

HPI < 60 → Re-evaluate or Reject Onboarding


let’s build a runnable Spring Boot microservice that automatically triggers HR performance evaluation events at specific onboarding checkpoints: 2 weeks, 1 month, 3 months, 6 months, 12 months, 18 months, 24 months.

This will use:

Spring Boot 3 (Java 21)

Kafka (Spring Kafka)

Scheduler (@Scheduled)

REST + GraphQL endpoints (for querying employee evaluation data)

DTO/Event classes for publishing HR evaluation events

Configurable timeframes (via application.yml)


1. Project Structure

hr-performance-evaluator/
├── src/
│   ├── main/
│   │   ├── java/com/example/hr/
│   │   │   ├── HrPerformanceApplication.java
│   │   │   ├── config/
│   │   │   │   └── KafkaConfig.java
│   │   │   ├── scheduler/
│   │   │   │   └── PerformanceScheduler.java
│   │   │   ├── service/
│   │   │   │   └── PerformanceEvaluationService.java
│   │   │   ├── model/
│   │   │   │   ├── Employee.java
│   │   │   │   └── PerformanceEvent.java
│   │   │   ├── kafka/
│   │   │   │   └── PerformanceEventProducer.java
│   │   │   └── controller/
│   │   │       └── PerformanceController.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── schema.graphqls
└── pom.xml


2. pom.xml

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example.hr</groupId>
    <artifactId>hr-performance-evaluator</artifactId>
    <version>1.0.0</version>
    <properties>
        <java.version>21</java.version>
        <spring.boot.version>3.3.3</spring.boot.version>
    </properties>

    <dependencies>
        <!-- Spring Boot Core -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <!-- Kafka -->
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>

        <!-- GraphQL -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-graphql</artifactId>
        </dependency>

        <!-- Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- JSON -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>

        <!-- Dev Tools -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>




3. Main Application


package com.example.hr;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HrPerformanceApplication {
    public static void main(String[] args) {
        SpringApplication.run(HrPerformanceApplication.class, args);
    }
}



package com.example.hr.config;

import org.apache.kafka.clients.admin.NewTopic;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.config.TopicBuilder;

@Configuration
public class KafkaConfig {

    @Bean
    public NewTopic performanceEvaluationTopic() {
        return TopicBuilder.name("hr-performance-evaluation")
                .partitions(3)
                .replicas(1)
                .build();
    }
}



package com.example.hr.model;

import java.time.LocalDateTime;

public record PerformanceEvent(
        String employeeId,
        String employeeName,
        String evaluationStage,  // e.g. "2W", "1M", "3M", "6M", ...
        LocalDateTime evaluationDate
) {}



package com.example.hr.kafka;

import com.example.hr.model.PerformanceEvent;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Service
public class PerformanceEventProducer {
    private final KafkaTemplate<String, PerformanceEvent> kafkaTemplate;

    public PerformanceEventProducer(KafkaTemplate<String, PerformanceEvent> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void publishEvent(PerformanceEvent event) {
        kafkaTemplate.send("hr-performance-evaluation", event.employeeId(), event);
        System.out.println("📤 Published Kafka Event: " + event);
    }
}


package com.example.hr.service;

import com.example.hr.kafka.PerformanceEventProducer;
import com.example.hr.model.PerformanceEvent;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.List;

@Service
public class PerformanceEvaluationService {

    private final PerformanceEventProducer producer;

    public PerformanceEvaluationService(PerformanceEventProducer producer) {
        this.producer = producer;
    }

    public void evaluateEmployee(String employeeId, String employeeName, String stage) {
        PerformanceEvent event = new PerformanceEvent(employeeId, employeeName, stage, LocalDateTime.now());
        producer.publishEvent(event);
    }

    public List<String> getAllEvaluationStages() {
        return List.of("2W", "1M", "3M", "6M", "12M", "18M", "24M");
    }
}




package com.example.hr.scheduler;

import com.example.hr.service.PerformanceEvaluationService;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
@EnableScheduling
public class PerformanceScheduler {

    private final PerformanceEvaluationService evaluationService;

    public PerformanceScheduler(PerformanceEvaluationService evaluationService) {
        this.evaluationService = evaluationService;
    }

    // 2 Weeks
    @Scheduled(cron = "0 0 9 * * *") // every day 9am; logic inside decides whom to evaluate
    public void triggerEvaluations() {
        // (Mock data, in reality you’d fetch employee data from DB)
        evaluationService.evaluateEmployee("E001", "Alice", "2W");
        evaluationService.evaluateEmployee("E002", "Bob", "1M");
        evaluationService.evaluateEmployee("E003", "Clara", "3M");
        evaluationService.evaluateEmployee("E004", "David", "6M");
        evaluationService.evaluateEmployee("E005", "Eve", "12M");
        evaluationService.evaluateEmployee("E006", "Frank", "18M");
        evaluationService.evaluateEmployee("E007", "Grace", "24M");
    }
}



GraphQL Schema (schema.graphqls)

type Query {
  evaluationStages: [String]
}



GraphQL Controller

package com.example.hr.controller;

import com.example.hr.service.PerformanceEvaluationService;
import org.springframework.graphql.data.method.annotation.QueryMapping;
import org.springframework.stereotype.Controller;
import java.util.List;

@Controller
public class PerformanceController {

    private final PerformanceEvaluationService service;

    public PerformanceController(PerformanceEvaluationService service) {
        this.service = service;
    }

    @QueryMapping
    public List<String> evaluationStages() {
        return service.getAllEvaluationStages();
    }
}



application.yml

spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer

logging:
  level:
    root: INFO



Run Instructions

# Terminal 1 - Start Kafka
zookeeper-server-start.sh config/zookeeper.properties
kafka-server-start.sh config/server.properties

# Terminal 2 - Start app
mvn spring-boot:run



How It Works

Scheduler runs daily (9 AM).

Evaluates employees based on their joining period → maps to a stage (2W, 1M, etc.).

Publishes a Kafka Event → "hr-performance-evaluation" topic.

HR Analytics or downstream consumers can process the event and record the result.

GraphQL endpoint /graphql lets HR query evaluation stages or metrics.




Everything below is ready-to-copy into the hr-performance-evaluator project structure you already have. I’ll highlight only the new/changed files and the run instructions (including a docker-compose to run Kafka + Zookeeper + Mongo).


What you’ll get

PerformanceEventConsumer — Kafka consumer that listens to hr-performance-evaluation, saves to Mongo.

EmployeeEvaluation — Mongo document storing last evaluation per employee.

EmployeeRepository — Spring Data Mongo repository.

EvaluationService — saves events + broadcasts real-time updates.

DashboardController — REST endpoints and SSE stream for real-time updates.

docker-compose.yml — local Kafka + Zookeeper + Mongo for quick testing.

application.yml — updated for consumer + mongo properties.

1) Add Maven dependencies (pom.xml changes)

Add these dependencies to your existing pom.xml:


<!-- Spring for MongoDB -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>

<!-- Spring Kafka (already present for producer, but ensure consumer support) -->
<dependency>
  <groupId>org.springframework.kafka</groupId>
  <artifactId>spring-kafka</artifactId>
</dependency>

<!-- For SSE support (web already included) -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>



package com.example.hr.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

import java.time.LocalDateTime;

@Document(collection = "employee_evaluations")
public class EmployeeEvaluation {

    @Id
    private String employeeId;
    private String employeeName;
    private String evaluationStage; // e.g., "2W", "1M", ...
    private LocalDateTime evaluationDate;
    private LocalDateTime lastUpdated;

    public EmployeeEvaluation() {}

    public EmployeeEvaluation(String employeeId, String employeeName, String evaluationStage, LocalDateTime evaluationDate) {
        this.employeeId = employeeId;
        this.employeeName = employeeName;
        this.evaluationStage = evaluationStage;
        this.evaluationDate = evaluationDate;
        this.lastUpdated = LocalDateTime.now();
    }

    // getters + setters omitted for brevity — include in real code
    // ...
    
    public String getEmployeeId() { return employeeId; }
    public void setEmployeeId(String employeeId) { this.employeeId = employeeId; }
    public String getEmployeeName() { return employeeName; }
    public void setEmployeeName(String employeeName) { this.employeeName = employeeName; }
    public String getEvaluationStage() { return evaluationStage; }
    public void setEvaluationStage(String evaluationStage) { this.evaluationStage = evaluationStage; }
    public LocalDateTime getEvaluationDate() { return evaluationDate; }
    public void setEvaluationDate(LocalDateTime evaluationDate) { this.evaluationDate = evaluationDate; }
    public LocalDateTime getLastUpdated() { return lastUpdated; }
    public void setLastUpdated(LocalDateTime lastUpdated) { this.lastUpdated = lastUpdated; }
}



package com.example.hr.repository;

import com.example.hr.model.EmployeeEvaluation;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface EmployeeRepository extends MongoRepository<EmployeeEvaluation, String> {
    // findById uses employeeId as @Id
}



Consumer config (optional) KafkaConsumerConfig.java

This config uses JsonDeserializer so the consumer will deserialize performance events into a map or POJO. Because our PerformanceEvent is a record, we'll consume as Map then map fields, which is safe and avoids needing the exact class on the consumer classpath.


package com.example.hr.config;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Bean;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.support.serializer.JsonDeserializer;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class KafkaConsumerConfig {

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Object> kafkaListenerContainerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "${spring.kafka.bootstrap-servers:localhost:9092}");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "hr-performance-consumers");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
        props.put(JsonDeserializer.TRUSTED_PACKAGES, "*");

        DefaultKafkaConsumerFactory<String, Object> consumerFactory =
                new DefaultKafkaConsumerFactory<>(props, new StringDeserializer(), new JsonDeserializer<>());

        ConcurrentKafkaListenerContainerFactory<String, Object> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory);
        return factory;
    }
}



Evaluation service that saves to Mongo and broadcasts SSE:


package com.example.hr.service;

import com.example.hr.model.EmployeeEvaluation;
import com.example.hr.repository.EmployeeRepository;
import org.springframework.stereotype.Service;
import org.springframework.web.servlet.mvc.method.annotation.SseEmitter;

import java.time.LocalDateTime;
import java.util.Map;
import java.util.concurrent.CopyOnWriteArrayList;

@Service
public class EvaluationService {

    private final EmployeeRepository repository;
    private final CopyOnWriteArrayList<SseEmitter> emitters = new CopyOnWriteArrayList<>();

    public EvaluationService(EmployeeRepository repository) {
        this.repository = repository;
    }

    public EmployeeEvaluation saveOrUpdateFromEvent(Map<String, Object> eventPayload) {
        String employeeId = (String) eventPayload.get("employeeId");
        String employeeName = (String) eventPayload.get("employeeName");
        String evaluationStage = (String) eventPayload.get("evaluationStage");
        String evaluationDateStr = String.valueOf(eventPayload.get("evaluationDate")); // may be ISO string

        LocalDateTime evaluationDate = LocalDateTime.now();
        // try to parse if ISO format; otherwise fallback to now
        try {
            evaluationDate = LocalDateTime.parse(evaluationDateStr);
        } catch (Exception ignored) {}

        EmployeeEvaluation doc = repository.findById(employeeId)
                .map(existing -> {
                    existing.setEmployeeName(employeeName);
                    existing.setEvaluationStage(evaluationStage);
                    existing.setEvaluationDate(evaluationDate);
                    existing.setLastUpdated(LocalDateTime.now());
                    return existing;
                })
                .orElseGet(() -> new EmployeeEvaluation(employeeId, employeeName, evaluationStage, evaluationDate));

        EmployeeEvaluation saved = repository.save(doc);

        // broadcast to SSE clients
        broadcast(saved);
        return saved;
    }

    public void broadcast(EmployeeEvaluation saved) {
        for (SseEmitter emitter : emitters) {
            try {
                emitter.send(SseEmitter.event()
                        .name("evaluation")
                        .data(saved));
            } catch (Exception e) {
                emitters.remove(emitter);
            }
        }
    }

    public SseEmitter createEmitter() {
        SseEmitter emitter = new SseEmitter(0L); // never timeout (0)
        emitters.add(emitter);
        emitter.onCompletion(() -> emitters.remove(emitter));
        emitter.onTimeout(() -> emitters.remove(emitter));
        emitter.onError((ex) -> emitters.remove(emitter));
        return emitter;
    }
}



package com.example.hr.kafka;

import com.example.hr.service.EvaluationService;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

import java.util.Map;

@Service
public class PerformanceEventConsumer {

    private final EvaluationService evaluationService;

    public PerformanceEventConsumer(EvaluationService evaluationService) {
        this.evaluationService = evaluationService;
    }

    // Consume as a generic Map (keys: employeeId, employeeName, evaluationStage, evaluationDate)
    @KafkaListener(topics = "hr-performance-evaluation", groupId = "hr-performance-consumers")
    public void consume(Map<String, Object> event) {
        // event will be deserialized to a Map by JsonDeserializer
        System.out.println("📥 Consumed event: " + event);
        evaluationService.saveOrUpdateFromEvent(event);
    }
}




package com.example.hr.controller;

import com.example.hr.model.EmployeeEvaluation;
import com.example.hr.repository.EmployeeRepository;
import com.example.hr.service.EvaluationService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.mvc.method.annotation.SseEmitter;

import java.util.List;

@RestController
@RequestMapping("/api")
public class DashboardController {

    private final EmployeeRepository repository;
    private final EvaluationService evaluationService;

    public DashboardController(EmployeeRepository repository, EvaluationService evaluationService) {
        this.repository = repository;
        this.evaluationService = evaluationService;
    }

    @GetMapping("/employees")
    public ResponseEntity<List<EmployeeEvaluation>> listAll() {
        return ResponseEntity.ok(repository.findAll());
    }

    @GetMapping("/employees/{id}")
    public ResponseEntity<EmployeeEvaluation> getById(@PathVariable String id) {
        return repository.findById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // SSE endpoint for real-time updates
    @GetMapping("/stream/evaluations")
    public SseEmitter streamEvaluations() {
        return evaluationService.createEmitter();
    }
}



Clients (dashboard) can call /api/stream/evaluations to receive real-time updates as evaluations are persisted.



application.yml (updated)


spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: hr-performance-consumers
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: "*"

  data:
    mongodb:
      uri: mongodb://mongo:27017/hrdb

logging:
  level:
    root: INFO



For local testing with Docker compose we point Mongo to mongo:27017. If you run the app locally without docker, change the URI to mongodb://localhost:27017/hrdb.


Docker Compose for local dev: docker-compose.yml

This brings up Zookeeper, Kafka, and MongoDB.



version: '3.8'
services:
  zookeeper:
    image: bitnami/zookeeper:latest
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
    ports:
      - "2181:2181"

  kafka:
    image: bitnami/kafka:latest
    depends_on:
      - zookeeper
    environment:
      - KAFKA_BROKER_ID=1
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
      - KAFKA_LISTENERS=PLAINTEXT://:9092
      - KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092
      - ALLOW_PLAINTEXT_LISTENER=yes
    ports:
      - "9092:9092"

  mongo:
    image: mongo:6.0
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:



10) Run instructions

Start services:

docker compose up -d (in the directory with docker-compose.yml)

Start the Spring Boot app:

mvn spring-boot:run

or build mvn package && java -jar target/hr-performance-evaluator-1.0.0.jar

Confirm Kafka topic exists (optional), or let the app create it (if your KafkaConfig creates the topic).

Trigger events:

If you're using the scheduler from the earlier code (the PerformanceScheduler that publishes mock events), the producer will create events and the consumer will persist them to Mongo.

Or produce a test event:


# Use kafka-console-producer (from Kafka) or any HTTP Kafka tool.
# Example JSON event:
{"employeeId":"E001","employeeName":"Alice","evaluationStage":"2W","evaluationDate":"2025-10-10T09:00:00"}



REST calls:

GET http://localhost:8080/api/employees → list last evaluation per employee

GET http://localhost:8080/api/employees/E001 → get one

Stream real-time:

Open http://localhost:8080/api/stream/evaluations from a browser or use curl:


curl -N http://localhost:8080/api/stream/evaluations



Example: Minimal JS front-end to receive SSE

Paste this in a simple index.html and open it in a browser (CORS may need enabling if served from a different origin):


<!doctype html>
<html>
<head><meta charset="utf-8"><title>HR Dashboard</title></head>
<body>
  <h1>Live Evaluations</h1>
  <ul id="list"></ul>

  <script>
    const evtSource = new EventSource('/api/stream/evaluations');
    evtSource.addEventListener('evaluation', e => {
      const data = JSON.parse(e.data);
      const li = document.createElement('li');
      li.textContent = `${data.employeeId} | ${data.employeeName} | ${data.evaluationStage} | ${data.evaluationDate}`;
      document.getElementById('list').prepend(li);
    });
    evtSource.onerror = err => console.error("EventSource error:", err);
  </script>
</body>
</html>



Notes & Considerations

Json serialization: I consume events as Map<String,Object> for flexibility. If you want strong typing, place the same PerformanceEvent class on the consumer side and set JsonDeserializer to that target type.

Idempotency: The consumer currently uses employeeId as the Mongo _id. Re-processing the same event updates the last evaluation. If you need event history, store events in a separate collection and keep the latest in this document.

Security/CORS: For production, secure Kafka, Mongo, and add authentication / CORS config for REST and SSE.

Scaling: Consumer group ID allows horizontal scaling of consumers. Use compacted topics if you intend to compact latest state at Kafka level.

Backpressure: SSE with many clients may require different streaming approach (WebSocket or reactive streams) in heavy traffic systems.

Time parsing: The service attempts to parse evaluationDate if provided as ISO; fallback to now.





