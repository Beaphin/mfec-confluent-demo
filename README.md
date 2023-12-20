sudo su -
unalias cp
yum install java-11-openjdk -y
yum install git -y
mkdir mfec-confluent-demo
cd mfec-confluent-demo
curl -O http://packages.confluent.io/archive/7.3/confluent-7.3.2.tar.gz
curl -O https://d1i4a15mxbxib1.cloudfront.net/api/plugins/confluentinc/kafka-connect-datagen/versions/0.6.3/confluentinc-kafka-connect-datagen-0.6.3.zip
git clone https://github.com/confluentinc/jmx-monitoring-stacks.git
cd jmx-monitoring-stacks
git fetch
git checkout 7.3-post
cd ..

mkdir -p /kafka/confluent/
mkdir -p /kafka_log
mkdir -p /kafka_data

tar xzf confluent-7.3.2.tar.gz --directory /kafka/confluent/
cp -r /kafka/confluent/confluent-7.3.2/etc/ /kafka/confluent/
cp connect-distributed.properties /kafka/confluent/etc/kafka/connect-distributed.properties

keytool -genkeypair -noprompt -alias self -keyalg RSA -keysize 2048 -sigalg SHA256withRSA -dname "CN=localhost" -validity 365 -keypass confluent -keystore privatekey.jks -storepass confluent -storetype JKS
mkdir -p /kafka/ssl/private/
cp privatekey.jks /kafka/ssl/private/kafka.confluentserver.keystore.jks

mkdir -p /kafka/confluent/log4j
cp /kafka/confluent/confluent-7.3.2/etc/kafka/connect-log4j.properties /kafka/confluent/log4j/connect_distributed_log4j.properties

mkdir -p /kafka/prometheus
cp -r /root/mfec-confluent-demo/jmx-monitoring-stacks/shared-assets/jmx-exporter/* /kafka/prometheus
cp /root/mfec-confluent-demo/jmx-monitoring-stacks/shared-assets/jmx-exporter/jmx_prometheus_javaagent-0.20.0.jar /kafka/prometheus/jmx_prometheus_javaagent.jar

cat connect-distributed.properties | grep plugin
/kafka/confluent/confluent-7.3.2/bin/confluent-hub install confluentinc-kafka-connect-datagen-0.6.3.zip

useradd -u 1003 datalab
chown -R datalab:datalab /kafka
chown -R datalab:datalab /kafka_log
chown -R datalab:datalab /kafka_data

mkdir /etc/systemd/system/confluent-kafka-connect.service.d
cp override.conf /etc/systemd/system/confluent-kafka-connect.service.d/override.conf
cp confluent-kafka-connect.service /usr/lib/systemd/system/confluent-kafka-connect.service
systemctl daemon-reload
#systemctl status confluent-kafka-connect
#systemctl start confluent-kafka-connect
