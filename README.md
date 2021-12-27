Kafka Training, Building a little project to have hands on the Kafka ecosystem
This is just a little project to understand Kafka, I post so it may help people understand the pipeline and the code.

Here is the context: We have a list of jobs that we can fill with description, and in the other side, one data source gathers people names with their job name. Theses two data are put into two separate kafka topics (person-job, job-desc) via a Producer (person-job) and a Connector (job-desc).
Then, we want to count how many people work for each jobs, and if we can, bind it with a description. We put the result on a topic that is consumed by a consumer. An UI can read this consumer to show the realtime data changes.

I will read here all the steps to launch all containers, creating topics, connector and schemas registration

I Implemented all of that: 
 - Person Name + Job: Java producer (localhost:8090) ... Generating Data (random)
 - Job description: MySQL + Debezium connector (localhost:8070, localhost:3307)
 - UI to write Job description: Angular (localhost:4205)
 - Stream Processing (Counting, join): Spring Cloud Stream (localhost:8050)
 - Consumer API: Java Consumer (localhost:8060)
 - Consumer UI: Angular (localhost:4210)
 - Confluent Cluster: (localhost:9021)

All of these containers are in my dockerhub (seyfa)
All of these apps code are in my github (ktr-code)

Here is a schema of what i Did


1. Prerequisites
  - Install Kafka cli
  - Docker + Docker-compose
  - Have enough RAM

2. Launch MYSQL Container
  - Open Terminal
  - cd app/lib/db
  - docker-compose up -d
  - docker exec -it <container_name> bash
  - mysl -u root -p + Enter + write 'example'
  - GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
  - FLUSH PRIVILEGES;

3. Launch Confluent Cluster
These step need time and compute power so don't be hurry
  - Open terminal
  - cd app/lib/confluent-cluster
  - docker-compose up -d

4. When the Cluster is loaded, then we create topics
  - kafka-topics.sh --create --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1 --topic person-job
  - kafka-topics.sh --create --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1 --topic person-job-stat

5. Add Schema registry for each of them's value
  - Open Control Center
  - Click on Topics -> person-job -> Schema -> Value -> Set a Schema
  - Copy the content of app/lib/resources/person-job.json into the editor
  - Do the same for person-job-stat

6. Add a Connector for our MySQL DB
  - Normally, CDC is enable, if it is not, sorry but find a way to do it
  - Open Control Center
  - Connect -> connect-default -> Add connector -> Upload connector config file
  - Choose file located in app/lib/resources/mysql_connector_debezium.json
  - Write your own IP Address on the field "HostName"
  - Click on Next and validate

7. Launch Consumer
  - Open Terminal
  - cd app/lib/consumer
  - docker-compose up -d

8. Launch Producers + Processing
  - Open terminal
  - cd app/lib/producers
  - docker-compose up -d

9. When you are finished, you can turnoff all containers (cluster, producers, consumer, db) Than remove all containers and image and maybe prune volumes

Wait less than 5 min then the pipeline is running
You can edit A description at http://localhost:4205
You can see realtime data at http://localhost:4210
You can also see data moving on topics using Control Center
