FROM quay.io/jupyter/all-spark-notebook:latest
ARG SPARK_VERSION=4.0.1
ARG HADOOP_VERSION=3.4
ARG OPENJDK_VERSION=17

USER root
RUN apt-get update && apt-get install -y openjdk-${OPENJDK_VERSION}-jdk && \
    rm -rf /var/lib/apt/lists/*

# Supprimer l'ancien hadoop-aws et telecharger les JARs compatibles Hadoop 3.4.2
# Hadoop 3.4.x utilise AWS SDK v2
RUN cd /usr/local/spark/jars && \
    rm -f hadoop-aws-*.jar aws-java-sdk-bundle-*.jar bundle-*.jar && \
    wget -q https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/3.4.2/hadoop-aws-3.4.2.jar && \
    wget -q https://repo1.maven.org/maven2/software/amazon/awssdk/bundle/2.29.51/bundle-2.29.51.jar && \
    wget -q https://repo1.maven.org/maven2/software/amazon/awssdk/url-connection-client/2.29.51/url-connection-client-2.29.51.jar && \
    wget -q https://repo1.maven.org/maven2/software/amazon/awssdk/s3-transfer-manager/2.29.51/s3-transfer-manager-2.29.51.jar && \
    wget -q https://repo1.maven.org/maven2/org/postgresql/postgresql/42.7.4/postgresql-42.7.4.jar && \
    echo "JARs installes: AWS S3A (Hadoop 3.4.2 + AWS SDK v2) + PostgreSQL JDBC"

USER $NB_UID
