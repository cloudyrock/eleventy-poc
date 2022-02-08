---
title: 'MongoDB reactive' 
date: 2014-04-18 11:30:00 
permalink: /v5/driver/mongodb-reactive/index.html
toc: true
eleventyNavigation:
  version: v5
  order: 87 
  parent: driver
  key: driver mongodb 
  title: 'MongoDB reactive'
---
[[TOC]]
## Introduction
The concept of driver and how it works is already explained in the [technical-overview](/v5/technical-overview) and [driver](/v5/driver) pages.

Here we explain how to use a driver with MongoDB and the different drivers Mongock provides.

<br />

-------------------------------------------

## MongoDB driver options and compatibility
**There are 4 drivers in the MongoDB family driver:**

|     Mongock driver      |                  Driver library              | Version compatibility |
|-------------------------|----------------------------------------------|-----------------------|
| SpringDataMongoV3Driver | org.springframework.data:spring-data-mongodb | 3.X.X                 |
| SpringDataMongoV2Driver | org.springframework.data:spring-data-mongodb | 2.X.X                 |
|    MongoSync4Driver     |        org.mongodb:mongodb-driver-sync       | 4.X.X                 |
|    MongoSync3Driver     |         org.mongodb:mongo-java-driver        | 3.X.X                 |

<br />

-------------------------------------------

## MongoDB common configuration
All the MongoDB drivers share the same configuration. 

<p class="tipAlt">When setting configuration via properties file, it must be prefixed by <b>mongock.mongo-db</b></p>

### Properties


| Property           | Description                                                                                  | Type                | Default value |
| -------------------|----------------------------------------------------------------------------------------------|---------------------|---------------|
| **writeConcern**   | Exactly the same MongoDB parameter **write concern**. For more information, visit the official MongoDB documentation for [write concern](https://docs.mongodb.com/manual/reference/write-concern/).  | Object      |{w:`majority`,<br />wTimeoutMs: null,<br />j:true} |  
| **readConcern**    | Exactly the same MongoDB parameter **read concern**. For more information, visit the official MongoDB documentation for [read concern](https://docs.mongodb.com/manual/reference/read-concern/).  | String      | `majority` |
| **readPreference** | Exactly the same MongoDB parameter **read preference**. For more information, visit the official MongoDB documentation for [read preference](https://docs.mongodb.com/manual/reference/read-preference/).  | String      | `primary` |

<br />

-------------------------------------------

## MongoDB native drivers
Mongock offers two  drivers for MongoDB native drivers. The latest, version Sync 4.x, and the previous major version 3.x, just for those who haven't upgraded yet.

- MongoSync4Driver
- MongoCore3Driver

### Get started 
Following the [get started section](/v5/get-started#steps-to-run-mongock), this covers steps 3 and 5 and 6.

#### Add maven dependency for the driver (step 2)

```xml
<dependency>
  <groupId>io.mongock</groupId>
  <artifactId>mongodb-driver-sync</artifactId>
  <!--<artifactId>mongodb-v3-driver</artifactId> for MongoDB driver v3-->
</dependency>
```

#### Build the driver (setps 5)
These classes provide the same two static initializers

- **withDefaultLock**(MongoClient mongoClient, String databaseName)
- **withLockStrategy**(MongoClient mongoClient, String databaseName, long lockAcquiredForMillis, long lockQuitTryingAfterMillis,long lockTryFrequencyMillis)

```java
MongoSync4Driver driver = MongoSync4Driver.withDefaultLock(mongoClient, databaseName);
```

#### Driver extra configuration (step 6)

##### Transactions
Due to the MongoDB API design, to work with transactions the [ClientSession](https://mongodb.github.io/mongo-java-driver/4.3/apidocs/mongodb-driver-sync/com/mongodb/client/ClientSession.html) object is required in every operation and then managed the transaction.
Mongock make this very simple. The developer only needs to specify a `ClientSession` parameter in the contructor or method of the `@ChangeUnit` and use in the MongoDB operations. **Mongock takes care of everything else.**
<br /><br />
The following code shows how to save documents inside the transaction using the `ClientSession` object.
```java
  @Execution
  public void execution(ClientSession clientSession, MongoDatabase mongoDatabase) {
  
    mongoDatabase.getCollection(CLIENTS_COLLECTION_NAME, Client.class)
            .insertMany(clientSession, IntStream.range(0, INITIAL_CLIENTS)
                    .mapToObj(ClientInitializerChangeLog::getClient)
                    .collect(Collectors.toList()));
  }
```

<br />

-------------------------------------------

## Examples 
<p class="successAlt">Please visit out example projects in [this repo](https://github.com/mongock/mongock-examples/tree/master/mongodb) for more information</p>



#### Example autoconfiguration with Springboot

```yaml
mongock:
  mongo-db:
    write-concern:
      w: majority
      wTimeoutMs: 1000
      journal: true
    read-concern: majority
    read-preference: primary
```

```java
@EnableMongock
@SpringBootApplication
public class QuickStartApp {

    /**
     * Be wared MongoTemplate needs to be injected
     */
    public static void main(String[] args) {
        SpringApplicationBuilder().sources(QuickStartApp.class)().run(args);
    }

    /**
     * Transaction Manager.
     * Needed to allow execution of changeSets in transaction scope.
     */
    @Bean
    public MongoTransactionManager transactionManager(MongoTemplate mongoTemplate) {
        return new MongoTransactionManager(mongoTemplate.getMongoDbFactory());
    }

}
```


#### Example with builder

```java
//this could be the SpringDataMongoV2Driver passing the same paremeter or MongoSync4Driver/MongoCore3Driver passing the MongoClient and databaseName
SpringDataMongoV3Driver driver = SpringDataMongoV3Driver.withDefaultLock(mongoTemplate);
driver.setWriteConcern(WriteConcern.MAJORITY.withJournal(true).withWTimeout(1000, TimeUnit.MILLISECONDS));
driver.setReadConcern(ReadConcern.MAJORITY);
driver.setReadPreference(ReadPreference.primary());
```
