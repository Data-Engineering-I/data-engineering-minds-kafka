# SSL
This repository contains the configuration files for zookeeper, zookeeper clients, Kafka brokers, producers, and consumers for launching a basic 3-node kafka cluster with SSL security. Please check the step-by-step [video tutorial](https://www.youtube.com/watch?v=hR_OuiqLgOo) on my Youtube channel.


### Steps to generate CA, Truststore and Keystore 

**Please Note** - For Truststores and Keystores, I have shown the steps only to generate *kafka.zookeeper.truststore.jks* and *kafka.zookeeper.keystore.jks*. The procedure is the same for generating such files for brokers, producers, and consumers.

**1. Generate CA** <br />
openssl req -new -x509 -keyout ca-key -out ca-cert -days 3650

**2. Create Truststore** <br />
keytool -keystore kafka.zookeeper.truststore.jks -alias ca-cert -import -file ca-cert

**3. Create Keystore** <br />
keytool -keystore kafka.zookeeper.keystore.jks -alias zookeeper -validity 3650 -genkey -keyalg RSA -ext SAN=dns:localhost

**4. Create certificate signing request (CSR)** <br />
keytool -keystore kafka.zookeeper.keystore.jks -alias zookeeper -certreq -file ca-request-zookeeper

**5. Sign the CSR** <br />
openssl x509 -req -CA ca-cert -CAkey ca-key -in ca-request-zookeeper -out ca-signed-zookeeper -days 3650 -CAcreateserial

**6. Import the CA into Keystore** <br />
keytool -keystore kafka.zookeeper.keystore.jks -alias ca-cert -import -file ca-cert

**7. Import the signed certificate from step 5 into Keystore** <br />
keytool -keystore kafka.zookeeper.keystore.jks -alias zookeeper -import -file ca-signed-zookeeper


### To check all the brokers connected to the Zookeeper running in 2-way SSL mode
`
zookeeper-shell.sh localhost:2182 -zk-tls-config-file zookeeper-client.properties
ls /brokers/ids
`

### To check all the topics available on Kafka
`
kafka-topics.sh --zookeeper localhost:2181 --list
`

*For latest versions of Apache Kafka,*<br/>
`
kafka-topics.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --list
`
### To create a topic inside Kafka
`
kafka-topics.sh --zookeeper localhost:2181 --create --topic mytopic --partitions 2 --replication-factor 3
`

*For latest versions of Apache Kafka,*<br/>
`
kafka-topics.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --create --topic mytopic --partitions 2 --replication-factor 3
`
#### To create a topic with additional configurations
`
kafka-topics.sh --zookeeper localhost:2181 --create --topic my-topic --partitions 2 --replication-factor 3 --config min.insync.replicas=2
`

*For latest versions of Apache Kafka,*<br/>
`
kafka-topics.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --create --topic my-topic --partitions 2 --replication-factor 3 --config min.insync.replicas=2
`
### To describe a topic
`
kafka-topics.sh --zookeeper localhost:2181 --describe --topic my-topic
`

*For latest versions of Apache Kafka,*<br/>
`
kafka-topics.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --describe --topic my-topic
`
### from comments on Youtube videos
Bootstrap.servers for producers/consumers= specifying one node is always fine and it will find the other 2 nodes part of the cluster. But, i would always recommend to include more than 1 because if the first node is down, second node helps to discover the cluster........ TS and KS for Producers/consumers => Keystore depends highly depends on the machine where u execute ur app... so, u can of course use same KS if they are on the same machine... For TS, u can have one common CA imported into this TS and use the same across everywhere!!

=========================================

https://www.youtube.com/watch?v=hR_OuiqLgOo&list=PLlBQ_B5-H_xhEHUC6OWI0eide_4wbt8Eo&index=4
----------------------------------------------------------------------------------------------
 Hichem Belhocine  I tested with Kafka2.7 and PEM and it works fine for me. Here is the steps (for all 3 brokers): 
1. 𝐆𝐞𝐧𝐞𝐫𝐚𝐭𝐞 𝐂𝐀 
echo 01 > serial.txt
touch index.txt
openssl req -x509 -config openssl-ca.cnf -newkey rsa:4096 -sha256 -nodes -out cacert.pem -outform PEM


2. 𝐂𝐫𝐞𝐚𝐭𝐞 𝐓𝐫𝐮𝐬𝐭𝐬𝐭𝐨𝐫𝐞
keytool -keystore kafka.broker1.truststore.jks -alias CARoot -import -file cacert.pem
keytool -keystore kafka.broker0.truststore.jks -alias CARoot -import -file cacert.pem
keytool -keystore kafka.broker2.truststore.jks -alias CARoot -import -file cacert.pem


𝟑. 𝐂𝐫𝐞𝐚𝐭𝐞 𝐊𝐞𝐲𝐬𝐭𝐨𝐫𝐞 ==  (whiile creating keystore it did not ask for keypassword. I think it is because of pkcs12 format. So, inside server.properties, give ssl.keystore.password and ssl.key.passoword the same value. else it will fail with SSL handshake)
keytool -keystore kafka.broker1.keystore.jks -alias broker1 -validity 3650 -genkey -keyalg RSA -storetype pkcs12 -ext SAN=dns:localhost
keytool -keystore kafka.broker0.keystore.jks -alias broker0 -validity 3650 -genkey -keyalg RSA -storetype pkcs12 -ext SAN=dns:localhost
keytool -keystore kafka.broker2.keystore.jks -alias broker2 -validity 3650 -genkey -keyalg RSA -storetype pkcs12 -ext SAN=dns:localhost


𝟒. 𝐂𝐫𝐞𝐚𝐭𝐞 𝐜𝐞𝐫𝐭𝐢𝐟𝐢𝐜𝐚𝐭𝐞 𝐬𝐢𝐠𝐧𝐢𝐧𝐠 𝐫𝐞𝐪𝐮𝐞𝐬𝐭 (𝐂𝐒𝐑) == 
keytool -keystore kafka.broker1.keystore.jks -alias broker1 -certreq -file ca-request-broker1 -ext SAN=DNS:localhost
keytool -keystore kafka.broker0.keystore.jks -alias broker0 -certreq -file ca-request-broker0 -ext SAN=DNS:localhost
keytool -keystore kafka.broker2.keystore.jks -alias broker2 -certreq -file ca-request-broker2 -ext SAN=DNS:localhost

𝟓. 𝐒𝐢𝐠𝐧 𝐭𝐡𝐞 𝐂𝐒𝐑 == 
openssl ca -config openssl-ca.cnf -policy signing_policy -extensions signing_req -out ca-signed-broker1 -infiles ca-request-broker1
openssl ca -config openssl-ca.cnf -policy signing_policy -extensions signing_req -out ca-signed-broker0 -infiles ca-request-broker0
openssl ca -config openssl-ca.cnf -policy signing_policy -extensions signing_req -out ca-signed-broker2 -infiles ca-request-broker2


𝟔. 𝐈𝐦𝐩𝐨𝐫𝐭 𝐭𝐡𝐞 𝐂𝐀 𝐢𝐧𝐭𝐨 𝐊𝐞𝐲𝐬𝐭𝐨𝐫𝐞 == 
keytool -keystore kafka.broker1.keystore.jks -alias CARoot -import -file cacert.pem
keytool -keystore kafka.broker1.keystore.jks -alias broker1 -import -file ca-signed-broker1

keytool -keystore kafka.broker0.keystore.jks -alias CARoot -import -file cacert.pem
keytool -keystore kafka.broker0.keystore.jks -alias broker0 -import -file ca-signed-broker0

keytool -keystore kafka.broker2.keystore.jks -alias CARoot -import -file cacert.pem
keytool -keystore kafka.broker2.keystore.jks -alias broker2 -import -file ca-signed-broker2
