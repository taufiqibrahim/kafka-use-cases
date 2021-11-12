FROM confluentinc/cp-kafka-connect-base:6.2.0

RUN echo "Installing connector plugins"

RUN confluent-hub install --no-prompt debezium/debezium-connector-mongodb:1.7.0
RUN confluent-hub install --no-prompt debezium/debezium-connector-mysql:1.7.0
RUN confluent-hub install --no-prompt confluentinc/kafka-connect-jdbc:10.2.5

# JDBC Drivers
# ------------
# MySQL
RUN cd /usr/share/confluent-hub-components/confluentinc-kafka-connect-jdbc/lib
RUN curl -o mysql-connector-java-8.0.16.jar https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.16/mysql-connector-java-8.0.16.jar
RUN cp mysql-connector-java-8.0.16.jar /usr/share/confluent-hub-components/confluentinc-kafka-connect-jdbc/lib/
RUN ls /usr/share/confluent-hub-components/confluentinc-kafka-connect-jdbc/lib
