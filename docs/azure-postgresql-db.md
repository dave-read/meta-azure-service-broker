﻿# Azure Database for PostgreSQL Service

[Azure Database for PostgreSQL](https://azure.microsoft.com/en-us/services/postgresql) is a relational database service based on the open source Postgres database engine. It is a fully managed database as a service offering capable of handling mission-critical workloads with predictable performance, security, high availability, and dynamic scalability.  Develop applications with Azure Database for PostgreSQL leveraging the open source tools and platform of your choice.

## Behaviors

### Provision
  
  1. Create a server.
  
### Provision-Poll
  
  1. Check whether creating server succeeds or not.
  
  2. Configure firewall rules
  
### Bind
  
  1. Collect [credentials](./azure-postgresql-db.md#format-of-credentials).
  
### Unbind

  Do nothing
  
### Deprovision

  1. Delete the server.

### Deprovision-Poll

  1. Check whether deleting server succeeds or not.

## Create an Azure Database for PostgreSQL service

1. Get the service name and plans

  ```
  cf marketplace
  ```

  Sample output:

  ```
  service              plans                                                                         description
  azure-postgresqldb   basic50*, basic100*, standard100*, standard200*, standard400*, standard800*   Azure PostgreSQL Database Service

  ```

  If you can not find the service name, please use the following command to make the plans public.

  ```
  cf enable-service-access azure-postgresqldb
  ```

2. Create a service instance

  Configuration parameters are supported with the provision request. These parameters are passed in a valid JSON object containing configuration parameters, provided either in-line or in a file.

  ```
  cf create-service azure-postgresqldb $service_plan $service_instance_name -c $path_to_parameters
  ```

  Supported configuration parameters:

  ```
  {
      "resourceGroup": "<resource-group>",        // [Required] Unique. Only allow up to 90 characters
      "location": "<azure-region-name>",          // [Required] support westus and northeurope only
      "postgresqlServerName": "<server-name>",    // [Required] Unique. Server name cannot be empty or null. It can contain only lowercase letters, numbers and '-', but can't start or end with '-' or have more than 63 characters. 
      "postgresqlServerParameters": {
          "allowPostgresqlServerFirewallRules": [ // [Optional] If present, ruleName, startIpAddress and endIpAddress are mandatory in every rule.
              {
                  "ruleName": "<rule-name-0>",    // The rule name can only contain 0-9, a-z, A-Z, -, _, and cannot exceed 128 characters
                  "startIpAddress": "xx.xx.xx.xx",
                  "endIpAddress": "xx.xx.xx.xx"
              },
              {
                  "ruleName": "<rule-name-1>",
                  "startIpAddress": "xx.xx.xx.xx",
                  "endIpAddress": "xx.xx.xx.xx"
              }
          ],
          "properties": {
              "version": "9.5" | "9.6",
              "sslEnforcement": "Enabled" | "Disabled",
              "storageMB": 51200 | 179200 | 307200 | ... | 1075200, // 51200, 51200+128000*1, 51200+128000*2 ... 51200+128000*8
              "administratorLogin": "<server-admin-name>",
              "administratorLoginPassword": "<server-admin-password>"
          }
      },
      "postgresqlDatabaseName": "<database-name>" // [Required] Unique. Database name cannot be empty or null. It can contain only lowercase letters, numbers and '-', but can't start or end with '-' or have more than 63 characters. 
  }
  ```

  For example:

  ```
  cf create-service azure-postgresqldb basic100 postgresqldb -c examples/postgresqldb-example-config.json
  ```

  The contents of `examples/postgresqldb-example-config.json`:

  ```
  {
      "resourceGroup": "azure-service-broker",
      "location": "eastus",
      "postgresqlServerName": "generated-string",
      "postgresqlServerParameters": {
          "allowPostgresqlServerFirewallRules": [
              {
                "ruleName": "all",
                "startIpAddress": "0.0.0.0",
                "endIpAddress": "255.255.255.255"
              }
          ],
          "properties": {
              "version": "9.6",
              "sslEnforcement": "Disabled",
              "storageMB": 51200,
              "administratorLogin": "generated-string",
              "administratorLoginPassword": "generated-string"
          }
      },
      "postgresqlDatabaseName": "generated-string"
  }
  ```

  >**NOTE:** Please remove the comments in the JSON file before you use it.
  
  Above parameters are also the defaults if the broker operator doesn't change broker default settings. You can just run the following command to create a service instance without the json file:
  
  ```
  cf create-service azure-postgresqldb basic100 postgresqldb
  ```

3. Check the operation status of creating the service instance

  The creating operation is asynchronous. You can get the operation status after the creating operation.

  ```
  cf service $service_instance_name
  ```

  For example:

  ```
  cf service postgresqldb
  ```

[More information](http://docs.cloudfoundry.org/devguide/services/managing-services.html#create).

## Using the services in your application

### Binding

  ```
  cf bind-service $app_name $service_instance_name
  ```

  For example:

  ```
  cf bind-service demoapp postgresqldb
  ```

### Format of Credentials

  Verify that the credentials are set as environment variables

  ```
  cf env $app_name
  ```

  The credentials have the following format:

  ```
  "credentials": {
    "postgresqlServerName": "postgresqlservera",
    "postgresqlDatabaseName": "postgresqldba",
    "postgresqlServerFullyQualifiedDomainName": "postgresqlservera.postgres.database.azure.com",
    "administratorLogin": "ulrich",
    "administratorLoginPassword": "u1r8chP@ss",
    "jdbcUrl": "jdbc:postgresql://postgresqlservera.postgres.database.azure.com:5432/postgresqldba?user=ulrich@fake-server&password=u1r8chP@ss&ssl=true",
    "hostname": "postgresqlservera.postgres.database.azure.com",
    "port": 5432,
    "name": "postgresqldba",
    "username": "ulrich", 
    "password": "u1r8chP@ss",
    "uri": "postgres://ulrich@postgresqlservera:u1r8chP@ss@postgresqlservera.postgres.database.azure.com:5432/postgresqldba"
  }

  ```
  
  >**NOTE:** The part `hostname` - `uri` is compatible with [Cloud Foundry MySQL Release](https://github.com/cloudfoundry/cf-mysql-release).
  
## Unbinding

  ```
  cf unbind-service $app_name $service_instance_name
  ```

  For example:

  ```
  cf unbind-service demoapp postgresqldb
  ```

## Delete the service instance

  ```
  cf delete-service $service_instance_name -f
  ```

  For example:

  ```
  cf delete-service postgresqldb -f
  ```
