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


===================
This is how my openssl-ca.cnf looks like:
HOME            = .
RANDFILE        = .rnd

####################################################################
[ ca ]
default_ca    = CA_default      # The default ca section

[ CA_default ]

base_dir      = /Users/vinod/mymac/kafka1/ssl
certificate   = $base_dir/cacert.pem   # The CA certifcate
private_key   = $base_dir/cakey.pem    # The CA private key
new_certs_dir = $base_dir              # Location for new certs after signing
database      = $base_dir/index.txt    # Database index file
serial        = $base_dir/serial.txt   # The current serial number

default_days     = 1000         # How long to certify for
default_crl_days = 30           # How long before next CRL
default_md       = sha256       # Use public key default MD
preserve         = no           # Keep passed DN ordering

x509_extensions = ca_extensions # The extensions to add to the cert

email_in_dn     = no            # Don't concat the email in the DN
copy_extensions = copy          # Required to copy SANs from CSR to cert

####################################################################
[ req ]
default_bits       = 4096
default_keyfile    = cakey.pem
distinguished_name = ca_distinguished_name
x509_extensions    = ca_extensions
string_mask        = utf8only

####################################################################
[ ca_distinguished_name ]
countryName         = Country Name (2 letter code)
countryName_default = DE

stateOrProvinceName         = State or Province Name (full name)
stateOrProvinceName_default = Berlin

localityName                = Locality Name (eg, city)
localityName_default        = Berlin

organizationName            = Organization Name (eg, company)
organizationName_default    = DEM

organizationalUnitName         = Organizational Unit (eg, division)
organizationalUnitName_default = DE

commonName         = Common Name (e.g. server FQDN or YOUR name)
commonName_default = ca-cert

emailAddress         = Email Address
emailAddress_default = test@test.com

####################################################################
[ ca_extensions ]

subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always, issuer
basicConstraints       = critical, CA:true
keyUsage               = keyCertSign, cRLSign

####################################################################
[ signing_req ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
basicConstraints       = CA:FALSE
keyUsage               = digitalSignature, keyEncipherment

####################################################################
[ signing_policy ]
countryName            = optional
stateOrProvinceName    = optional
localityName           = optional
organizationName       = optional
organizationalUnitName = optional
commonName             = supplied
emailAddress           = optional


==================================================
==================================================

And, when I executed openssl s_client -debug -connect localhost:9097 -tls1, I got below output as u which is fine as these are related to client (producer/consumer) connections. I was able to create topics as well and all the three brokers are able to connect without issue.
CONNECTED(00000003)
write to 0x7fdc20c25aa0 [0x7fdc21022e03] (200 bytes => 200 (0xC8))
0000 - 16 03 01 00 c3 01 00 00-bf 03 01 40 38 24 3f 3a   ...........@8$?:
0010 - 16 f5 f3 5e e9 5e c0 ca-87 1f ec a9 ae a5 c6 ff   ...^.^..........

---
no peer certificate available
---
No client certificate CA names sent
---
SSL handshake has read 5 bytes and written 7 bytes
---
New, (NONE), Cipher is (NONE)
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1
    Cipher    : 0000
    Session-ID: 
    Session-ID-ctx: 
    Master-Key: 
    Key-Arg   : None
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    Start Time: 1609362469
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
---





















