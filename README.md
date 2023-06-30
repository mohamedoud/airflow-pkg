# Airflow Packing
Dependency archive for Airflow 2.6.2 with Python 3.9.

## Prerequisite

To install Python 3.9 and airflow system dependencies, you can modify the existing [TDP build ](https://github.com/TOSIT-IO/TDP/tree/main/build-env) Dockerfile by adding the following section before the entrypoint:

```dockerfile
# System dependencies
RUN apt-get -q install -y libffi-dev libldap2-dev
WORKDIR /tmp
# Download and install Python 3.9 source
RUN wget https://www.python.org/ftp/python/3.9.6/Python-3.9.6.tgz \
    && tar xzf Python-3.9.6.tgz
RUN cd Python-3.9.6 \
    && ./configure --enable-optimizations \
    && make altinstall
```

## Create dependency archive

Run the following commands from the TDP build container:

```bash
TDP_VERSION=TDP-1.0.0
AIRFLOW_VERSION=2.6.2
AIRFLOW=airflow-$AIRFLOW_VERSION-$TDP_VERSION
ARCHIVE=$AIRFLOW.tar.gz
AIRFLOW_EXTRA=[flower,redis,kerberos,celery,apache.hdfs,apache.hive,apache.spark]
cd /tdp
mkdir -p target/$AIRFLOW/dependencies
python3.9 -m venv airflow_venv
source airflow_venv/bin/activate
python3.9 -m pip install --upgrade pip wheel
python3.9 -m pip install psycopg2-binary setuptools python-ldap
python3.9 -m pip install apache-airflow${AIRFLOW_EXTRA}==$AIRFLOW_VERSION --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-$AIRFLOW_VERSION/constraints-3.9.txt"
python3.9 -m pip freeze > target/${AIRFLOW}/requirements.txt
python3.9 -m pip wheel --no-cache-dir --wheel-dir target/${AIRFLOW}/dependencies -r target/${AIRFLOW}/requirements.txt
deactivate
cd target 
tar -czvf $ARCHIVE ${AIRFLOW}
sha256sum $ARCHIVE | sed 's| .*/| |' > $ARCHIVE.sha256
```

