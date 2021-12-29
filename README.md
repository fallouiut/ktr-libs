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

All of these containers are in my dockerhub (https://hub.docker.com/r/seyfa/ktr/tags)
All of these apps code are in my github (https://github.com/fallouiut/ktr-code)

Here is a schema of what i Did: ![topic person-job](https://user-images.githubusercontent.com/23740922/147651888-721a3079-a7df-4f97-ae64-8d769cbe495e.png)

<h2>1. Prerequisites</h2>
  - Install Kafka cli <br/>
  - Docker + Docker-compose <br/>
  - Have enough RAM <br/>

<h2>2. Launch MYSQL Container</h2>
  - Open Terminal <br/>
  - cd app/lib/db <br/>
  - docker-compose up -d <br/>
  - docker exec -it <container_name> bash <br/>
  - mysl -u root -p + Enter + write 'example' <br/>
  - GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'; <br/>
  - FLUSH PRIVILEGES;

<h2>3. Launch Confluent Cluster</h2>
These step need time and compute power so don't be hurry
  - Open terminal
  - cd app/lib/confluent-cluster
  - docker-compose up -d

<h2>4. When the Cluster is loaded, then we create topics</h2>
  - kafka-topics.sh --create --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1 --topic person-job
  - kafka-topics.sh --create --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1 --topic person-job-stat

<h2>5. Add Schema registry for each of them's value
  - Open Control Center
  - Click on Topics -> person-job -> Schema -> Value -> Set a Schema
  - Copy the content of app/lib/resources/person-job.json into the editor
  - Do the same for person-job-stat

<h2>6. Add a Connector for our MySQL DB</h2>
  - Normally, CDC is enable, if it is not, sorry but find a way to do it
  - Open Control Center
  - Connect -> connect-default -> Add connector -> Upload connector config file
  - Choose file located in app/lib/resources/mysql_connector_debezium.json
  - Write your own IP Address on the field "HostName"
  - Click on Next and validate

<h2>7. Launch Consumer</h2>
  - Open Terminal
  - cd app/lib/consumer
  - docker-compose up -d

<h2>8. Launch Producers + Processing</h2>
  - Open terminal
  - cd app/lib/producers
  - docker-compose up -d

<h2>9. When you are finished, you can turnoff all containers (cluster, producers, consumer, db) Than remove all containers and image and maybe prune volumes</h2>

Wait less than 5 min then the pipeline is running
You can edit A description at http://localhost:4205
You can see realtime data at http://localhost:4210
You can also see data moving on topics using Control Center
