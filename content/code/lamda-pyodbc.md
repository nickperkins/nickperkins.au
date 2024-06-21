---
title: "Using pyodbc in AWS Lambda functions"
date: 2019-07-21T12:57:05+10:00
draft: false

categories: [Serverless, AWS]
tags: [aws, lambda, serverless, docker, chalice]
toc: false
author: ""
---
<!-- cSpell:words pyodbc ODBC msodbsql -->
This week I was working on an AWS Lambda function that needed to read and write from a legacy Microsoft SQL database. It's written using the [AWS Chalice](https://github.com/aws/chalice) framework and in local testing everything looked great. Not so much when we needed to deploy it to AWS for testing.

## Why?

Most of the time that you include a python package for use in a lambda function, Chalice is able to package that into the deployment, and you're good to go. However, that's only true if the package doesn't rely on native libraries that aren't included in the Amazon Linux environment that lambda functions run on. Microsoft ODBC drivers are one of those things.

Thankfully you can compile these libraries yourself and include them in the package being deployed into lambda. With thanks to [this](https://gist.github.com/carlochess/658a98589709f46dbb3d20502e48556b) gist for the starting point, here is what I did to compile the drivers.

## Pre-requisites

* Docker

## Steps

First we start a docker container using an image that mimics the Amazon Linux environment your function will be running in. In my case I'm using python 3.7.

```bash
docker run -it --rm --entrypoint bash  -e ODBCINI=/var/task -e ODBCSYSINI=/var/task \
-v "$PWD":/var/task  lambci/lambda:build-python3.7
```

This sets the current working directory to be a volume within the docker container where all the compilation artifacts will end up. We then install the MS SQL ODBC drivers from Microsoft's rpm repository, copy them into our target directory and set some environment variables, so the system is looking in our target directory for any libraries or include files.

```bash
curl https://packages.microsoft.com/config/rhel/6/prod.repo > /etc/yum.repos.d/mssql-release.repo
# Disable the Amazon repos as the version provided isn't compatible with msodbcsql
yum -y install unixODBC --disablerepo=amzn*
ACCEPT_EULA=Y yum -y install msodbcsql
cp -r /opt/microsoft/msodbcsql .
export CFLAGS="-I/var/task/include"
export LDFLAGS="-L/var/task/lib"
```

When we install `msodbsql` it installs a number of dependency packages including `unixODBC`.  However, we need to include the `unixODBC` drivers the libraries in our lambda function package, so we download the source, compile it and install it into our target volume. Note the version number installed by the previous step and match it to the source package.

```bash
curl ftp://ftp.unixodbc.org/pub/unixODBC/unixODBC-2.3.7.tar.gz -O
tar xvzf unixODBC-2.3.7.tar.gz
cd unixODBC-2.3.7
./configure  --sysconfdir=/var/task  --disable-gui --disable-drivers --enable-iconv --with-iconv-char-enc=UTF8 \
--with-iconv-ucode-enc=UTF16LE --prefix=/var/task
make install
cd ..
rm -rf unixODBC-2.3.7 unixODBC-2.3.7.tar.gz
```

Install the `pyodbc` package into the current directory. We use the `-t` option to specify the target directory.

```bash
pip install pyodbc -t .
```

Create updated configuration files to point to the new location for the drives. You need to ensure that your `pyodbc` code is referencing the same driver version. Originally our code was referencing `ODBC Driver 17 for SQL Server` which was fine locally, but not so here.

```bash
cat <<EOF > odbcinst.ini
[ODBC Driver 13 for SQL Server]
Description=Microsoft ODBC Driver 13 for SQL Server
Driver=/var/task/msodbcsql/lib64/libmsodbcsql-13.1.so.9.2
UsageCount=1
EOF

cat <<EOF > odbc.ini
[ODBC Driver 13 for SQL Server]
Driver      = ODBC Driver 13 for SQL Server
Description = My ODBC Driver 13 for SQL Server
Trace       = No
EOF
```

You now have a directory containing all the libraries, drivers and packages needed to connect to an MS SQL database from an AWS Lambda function. If you are using AWS Chalice, you copy all this into the `vendors` directory and Chalice will include it into the deployment package.
