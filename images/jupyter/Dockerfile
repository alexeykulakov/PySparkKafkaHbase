FROM python:3.7.0


SHELL ["/bin/bash", "-o", "pipefail", "-c"]
USER root

ARG spark_version="2.4.3"
ARG hadoop_version="2.7"
ARG openjdk_version="8"

ENV APACHE_SPARK_VERSION="${spark_version}" \
    HADOOP_VERSION="${hadoop_version}"

RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    "openjdk-${openjdk_version}-jre-headless" \
    ca-certificates-java && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

WORKDIR /tmp
RUN wget -q "https://archive.apache.org/dist/spark/spark-${spark_version}/spark-${spark_version}-bin-hadoop${hadoop_version}.tgz" && \
    tar xzf "spark-${spark_version}-bin-hadoop${hadoop_version}.tgz" -C /usr/local --owner root --group root --no-same-owner && \
    rm "spark-${spark_version}-bin-hadoop${hadoop_version}.tgz"

WORKDIR /usr/local

ENV SPARK_HOME=/usr/local/spark
ENV SPARK_OPTS="--driver-java-options=-Xms1024M --driver-java-options=-Xmx4096M --driver-java-options=-Dlog4j.logLevel=info" \
    PATH="${PATH}:${SPARK_HOME}/bin"

RUN ln -s "spark-${spark_version}-bin-hadoop${hadoop_version}" spark && \
    # Add a link in the before_notebook hook in order to source automatically PYTHONPATH
    mkdir -p /usr/local/bin/before-notebook.d && \
    ln -s "${SPARK_HOME}/sbin/spark-config.sh" /usr/local/bin/before-notebook.d/spark-config.sh

RUN cp -p "${SPARK_HOME}/conf/spark-defaults.conf.template" "${SPARK_HOME}/conf/spark-defaults.conf" && \
    echo 'spark.driver.extraJavaOptions -Dio.netty.tryReflectionSetAccessible=true' >> "${SPARK_HOME}/conf/spark-defaults.conf" && \
    echo 'spark.executor.extraJavaOptions -Dio.netty.tryReflectionSetAccessible=true' >> "${SPARK_HOME}/conf/spark-defaults.conf"


ENV PYSPARK_PYTHON=/usr/local/bin/python

RUN pip install --upgrade pip
RUN pip install jupyterlab && jupyter serverextension enable --py jupyterlab

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt


EXPOSE 8888
RUN mkdir -p /opt/app/data

CMD jupyter lab --ip=* --port=8888 --no-browser --notebook-dir=/opt/app/data --allow-root
