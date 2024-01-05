---
title: 'Couchbase' 
date: 2023-02-18 11:30:00 
permalink: /v5/driver/couchbase/index.html
toc: true
eleventyNavigation:
  version: v5
  order: 100 
  parent: driver
  key: driver couchbase 
  title: 'Couchbase'
---
<h1 class="title">Couchbase driver</h1>

[[TOC]]

## Introduction
This section explains the Mongock Driver for Couchbase and how to use it.
<br />

-------------------------------------------

## Couchbase driver options and compatibility

Mongock provides the `CouchbaseDriver`, which is compatible with official couchbase Java SDK `com.couchbase.client:java-client` 3.x.x.
`CouchbaseDriver` is tested against Couchbase Server version 6 and Couchbase Server version 7+, but should be working even with older versions.

You can also use the Mongock spring extension to get advantage from the autoconfigure approach with Springboot.

<br />

-------------------------------------------

## CouchbaseDriver common configuration

<p class="tipAlt">When setting configuration via properties file, it must be prefixed by <b>mongock.couchbase</b></p>

### Properties


| Property       | Description                                                                                                                                                   | Type   | Default value |
|----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|--------|---------------|
| **scope**      | The custom scope to be used to store the lock and changeset information in couchbase. This value should only be set if you are using Couchbase server 7+      | String | _default      |  
| **collection** | The custom collection to be used to store the lock and changeset information in couchbase. This value should only be set if you are using Couchbase server 7+ | String | _default      |

<br />

------------------------------------------- 


## Get started 
Following the [get started section](/v5/get-started#steps-to-run-mongock), this covers steps 3 and 5 and 6.

### Add maven dependency for the driver (step 2)

#### Standalone 
```xml
<!--Standalone use-->
<dependency>
  <groupId>io.mongock</groupId>
  <artifactId>couchbase-driver</artifactId>
</dependency>

```

#### With Springboot 
```xml
<dependency>
  <groupId>io.mongock</groupId>
  <artifactId>couchbase-springboot-driver</artifactId>
</dependency>
```

#### With Springboot 3
```xml
<dependency>
  <groupId>io.mongock</groupId>
  <artifactId>couchbase-springboot-v3-driver</artifactId>
</dependency>
```

### Build the driver (setps 5)

<p class="successAlt"><b>This step is only required for builder approach.</b> Mongock handles it when autoconfiguration is enabled.</p>
These classes provide the same two static initializers

- **withDefaultLock**(Cluster cluster, Collection collection)
- **withLockStrategy**(Cluster cluster, Collection collection, long lockAcquiredForMillis, long lockQuitTryingAfterMillis,long lockTryFrequencyMillis)

```java
CouchbaseDriver driver = CouchbaseDriver.withDefaultLock(cluster, collection);
```

## Examples 
<p class="successAlt">Please visit our example projects in <a href="https://github.com/mongock/mongock-examples/tree/master/couchbase">this repo</a> for more information</p>



#### Example autoconfiguration with Springboot

```yaml
spring:
  data:
    couchbase:
      bucket-name: bucket
  couchbase:
    connection-string: couchbase://localhost:11210
    username: Administrator
    password: password

mongock:
  enabled: true
  migration-scan-package: io.mongock.examples.couchbase.springboot.migration
```

```java
@EnableMongock
@SpringBootApplication
public class QuickStartApp {
    
    public static void main(String[] args) {
        SpringApplicationBuilder().sources(QuickStartApp.class)().run(args);
    }
    
}
```

### Example: Couchbase standalone
```java
Cluster cluster = Cluster.connect(
        "couchbase://127.0.0.1",
        "Administrator",
        "password"
        );
Collection collection = cluster.bucket("bucketName").defaultCollection();
CouchbaseDriver driver = CouchbaseDriver.withDefaultLock(cluster, collection);
```

### Example: Couchbase Springboot Java Config
```java
@EnableMongock
@SpringBootApplication
public class QuickStartApp {
    
  public static void main(String[] args) {
    SpringApplication.run(QuickStartApp.class, args);
  }

  @Configuration
  public static class Config extends AbstractCouchbaseConfiguration {

    @Override
    public String getConnectionString() {
      return "couchbase://127.0.0.1";
    }

    @Override
    public String getUserName() {
      return "Administrator";
    }

    @Override
    public String getPassword() {
      return "password";
    }

    @Override
    public String getBucketName() {
      return "travel-sample";
    }
  }

}
```
