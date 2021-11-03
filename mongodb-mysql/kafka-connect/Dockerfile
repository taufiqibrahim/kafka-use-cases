FROM confluentinc/cp-kafka-connect-base:6.2.0

RUN echo "Installing connector plugins"

RUN confluent-hub install --no-prompt debezium/debezium-connector-mongodb:1.7.0
RUN confluent-hub install --no-prompt debezium/debezium-connector-mysql:1.7.0
