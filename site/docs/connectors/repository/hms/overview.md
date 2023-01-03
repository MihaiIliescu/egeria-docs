<!-- SPDX-License-Identifier: CC-BY-4.0 -->
<!-- Copyright Contributors to the Egeria project. -->


--8<-- "snippets/content-status/in-development.md"

# Hive Metastore Repository Connector

This repository has been created to manage artifacts and issues relating to integration with Hive Metastore (HMS).
This connector is based on the same polling pattern that the [File sample OMRS connector uses](/connectors/repository/file-sample-omrs-connector/overview/).

!!! info "Connector details"
- Connector Category:  [Repository and Event Mapper Connectors](/connectors/#repository-and-event-mapper-connectors)
- Hosting Service: [Local OMRS Repository Connector](/services/omrs/component-descriptions/local-repository-connector)
- Hosting Server: [repository proxy](/concepts/repository-proxy)
- Source Module: [Hive Metastore Repository Connector :material-github:](https://github.com/odpi/egeria-connector-hivemetastore){ target=gh }
- Jar File Name: `egeria-connector-hivemetastore-1.0-SNAPSHOT.jar`

__Important notice__

The gradle JAR step will include some of the dependencies into the connector JAR, making is a semi-Fat Jar. This makes sure that additional dependencies are automatically deployed together with the connector.

##  Configuration

### Repository Proxy Connector embedded configuration

Configure a [Repository proxy with an embedded native repository](/connectors/repository/repository-proxy-embedded-repository/)

### Configure the event mapper connector

Any open metadata repository that supports its own API **must** also implement an event mapper to ensure the [Open Metadata Repository Services (OMRS)](/services/omrs) is notified when metadata is added to the repository without going through the open metadata APIs.

The [event mapper](/concepts/event-mapper-connector) is a connector that listens for proprietary events from the repository and converts them into calls to the OMRS. The OMRS then distributes this new metadata.

For the Hive Metastore (HMS) Repository Proxy Connector this Event mapper currently polls for Hive Metastore content.
It may be enhanced in the future to also emit granular events to track the HMS metadata as it changes.


!!! post "POST - configure event mapper"
```
{{platformURLRoot}}/open-metadata/admin-services/users/{{adminUserId}}/servers/{{serverName}}/local-repository/event-mapper-details?connectorProvider={{fullyQualifiedJavaClassName}}&eventSource={{resourceName}}
```

The `connectorProvider` should be set to the fully-qualified Java class name for the [connector provider](/concepts/connector-provider), and the `eventSource` should give the details for how to access the events (for example, the hostname and port number of an Apache Kafka bootstrap server).

#### HMS connector configuration overview
![HMS connector configuration overview](./hms config.drawio.svg)
HMS connector configuration overview 
 
Event mapper Endpoint address should be defined with the url of the thrift endpoint. 

like this 
```
"endpoint": {
"class": "Endpoint",
"address": "thrift://catalog.eu-de.dataengine.cloud.ibm.com:9083"
},
```

`configurationProperties` parameters

| Event mapper configuration parameter name | default      | Description                                                                                                       |
|------------------------------------------|--------------|-------------------------------------------------------------------------------------------------------------------|
| qualifiedNamePrefix                      | empty string | This is a prefix for the qualifiedName. This prefix is used on every entity that is created using this connector. |
| refreshTimeInterval                      | null         | Poll interval in minutes. If null only poll once at connector start time.                                         |
| CatalogName                              | hive         | This is the HMS catalog name.                                                                                     |
| DatabaseName                             | default      | This is the HMS database name.                                                                                    |
| sendPollEvents                           |              | Set this to true to send events to the cohort every poll.                                                         |
| endpointAddress                          |              | url to access the data that this metadata describes                                                               |
| sendSchemaTypesAsEntities                | false        | Set this to true to use the old deprecated style of creating schematypes as entities.                             |
| cacheIntoCachingRepository               | true         | Set this to false to not cache the metadata content                                                               |

## Using with the IBM Cloud® Data Engine service.

To use this connector with [IBM Cloud® Data Engine service](https://cloud.ibm.com/catalog/services/data-engine-previously-sql-query), the code needs to be recompiled to bring in the IBM HMS Client library.
For more details see the [IBM documentation](https://cloud.ibm.com/docs/sql-query?topic=sql-query-hive_metastore#hive_compatible_client) on this.
There is a [bash script](https://github.com/odpi/egeria-connector-hivemetastore/blob/main/utilities/createIBMHMS.sh) that is supplied as-is and 
can be used in development on a Mac to build and run an Egeria platform that contains the [IBM Hive-compatible client](https://us.sql-query.cloud.ibm.com/download/catalog/hive-metastore-standalone-client-3.1.2-sqlquery.jar).

The following additional security parameters need to be specified in the `configurationProperties` as the IBM Hive-compatible client
uses a secure API to talk to the IBM Cloud®.


| Event mapper configuration parameter name | default | Description                                                                                           |
|-------------------------------------------|---------|-------------------------------------------------------------------------------------------------------|
| MetadataStoreUserId                       | null    | The Data Engine service [crn](https://cloud.ibm.com/docs/account?topic=account-crn).                  |
| MetadataStorePassword                     | null    | The [API key](https://www.ibm.com/docs/en/app-connect/container?topic=servers-creating-cloud-api-key) |
| useSSL                                    | false   | Set to true                                                                                     |

## Design

### Components
The high level architecture of the connector is:
![Caching Repository proxy components](HMS%20Connector.drawio.png)

It shows how the event mapper polling loop:

- Gets the Hive metastore information from the Hive metastore.
- Adds the appropriate reference entities and relationships to the repository connector
- Finds the entities and relationships per asset (Database)
- Sends a batched event per asset
- Waits for the length of time specified in the refreshTimeInterval configuration parameter.
- repeats

### Working with Hive Metastore and its APIs.

The Hive Metastore can be run as a standalone one. This standalone server jar file is also required for the Client
API. The HMSClient API used is
[https://github.com/apache/hive/blob/master/standalone-metastore/metastore-common/src/main/java/org/apache/hadoop/hive/metastore/HiveMetaStoreClient.java](https://github.com/apache/hive/blob/master/standalone-metastore/metastore-common/src/main/java/org/apache/hadoop/hive/metastore/HiveMetaStoreClient.java)
It uses the Thrift API to communicate with the Hive Metastore.
At this time (July 2022) the version 3.1.3 of this Hive Metastore has [vulnerabilities](https://mvnrepository.com/artifact/org.apache.hive/hive-standalone-metastore/3.1.3)
A number of excludes were required in the gradle build file to ensure the appropriate vulnerable libraries are not present - as reported by Sonarscan and lift.

HMS Client calls used:


| HMS Client call                                        | Description                                                                                                                         |
|--------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| client.getTables(&lt;catName&gt;,  "*")          | Get all the tables from the Catalog with name &lt;catName&gt; and database with name &lt;dbName&gt;                                 |
| client.getTable(&lt;catName&gt;, &lt;dbName&gt;, &lt;tableName&gt;) | Get the table details for table named &lt;tableName&gt; in Catalog &lt;catName&gt;, the table returned contains the column details |


### Hive Metastore mapping to Egeria OMRS open types


Egeria has an open type called Database we are mapping this to the Hive Database. Note that at Hive 3 there are
higher level concepts called catalogs that hold databases. This connector supports 2 catalog and one database,
these are specified by name in the Egeria configuration.

#### Entity Types

| HMS concept | Description                           | Egeria open Entity type | Comments                                                                 |
|-------------|---------------------------------------|-------------------------|--------------------------------------------------------------------------|
| Catalog     | Higher level of container within Hive | Not modeled             | The getCatalogs API is not always present in all HMS implementations     |
| Database    | Lives within a Catalog                | Database                |                                                                          |
| n/a         | n/a                                   | Connection              | This represents the connection to the instance data                      |
| n/a         | n/a                                   | ConnectionType          | This is the type of the connection                                       |
| n/a         | n/a                                   | Endpoint                | This is where the endpoint information is stored                         |
| n/a         | n/a                                   | DeployedDatabaseSchema  | Deployed Schema                                                          |
| Database    | Lives within a Catalog                | RelationalDBSchemaType  | Database schema type                                                     |
| Database    | Lives within a Catalog                | RelationalTable         | Relational Table                                                         |
| Database    | Lives within a Catalog                | RelationalColumn        | Relational Column                                                        |

#### Relationship Types

| Egeria open Relationship type | Comments                                                                |
|-------------------------------|-------------------------------------------------------------------------|
| ConnectionEndpoint            | Relationship between Connection and Endpoint                            |
| ConnectionConnectorType       | Relationship between Connection and Connector Type                      |
| ConnectionToAsset             | Relationship between Connection and Asset                               |
| AssetSchemaType               | Relationship between Database (the asset) and Schema Type               |
| AttributeForSchema            | Relationship between the RelationalTable and RelationalColumn           |
| DataContentForDataSet         | Relationship between DeloyedDatabaseSchema and RelationalDBSchemaType   |

#### Classification Types

| HMS concept                  | Description                                                      | Egeria open Classifation type | Comments                                                            |
|------------------------------|------------------------------------------------------------------|-------------------------------|---------------------------------------------------------------------|
| Hive table Type              | if this is VIRTUAL_VIEW then this is a view ratheer than a table | CalculatedValue               | The RelationalTable is classified with CalculatedValue if is a view |
| for columns fieldSchema Type | This is the type of the Hive column (e.g. String)                | TypeEmbeddedAttribute         | This contains the type of the column                                |
| for tables n/a               | n/a                                                              | TypeEmbeddedAttribute         | The type of the Table                                               |



You may also find these links in the Egeria documentation useful:

* [Repository Connectors](https://egeria-project.org/concepts/repository-connector/)
* [Integration Connector](http://egeria-project.org/concepts/integration-connector/)

During 2022 we have also had a number of Webinars relating to connector choices and design:

* [Webinar Program](https://egeria-project.org/education/webinar-program/overview/)


### Reference materials

* [https://github.com/odpi/egeria/blob/master/open-metadata-implementation/repository-services/README.md](https://github.com/odpi/egeria/blob/master/open-metadata-implementation/repository-services/README.md)
  and it's sub-pages are great resources for developers.
* [Egeria Webinars](https://wiki.lfaidata.foundation/display/EG/Egeria+Webinar+program) particularly the one on repository connectors.


----

License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/),
Copyright Contributors to the ODPi Egeria project.


