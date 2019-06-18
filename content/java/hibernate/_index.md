---
title: "Hibernate"
date: 2019-05-31T18:21:05-07:00
draft: false
weight: 1
---

![](/images/hibernate-logo.svg)

- [Introduction](#introduction)
- [Requirements](#requirements)
- [Using the integration](#using-the-integration)
    - [Imports](#imports)
    - [Maven imports](#imports#0)
- [End to end example](#end-to-end-example)
    - [Directory structure](#directory-structure)
    - [Source code](#source-code)
    - [Question.java](#0)
    - [ListQuestions.java](#1)
    - [hibernate.cfg.xml](#2)
    - [Question.hbm.xml](#3)
- [References](#references)

### Introduction
We provide an integration for Hibernate ORM that will inspect and augment your SQL with information about your
setup. It is best used when coupled with other frameworks such as:

{{<card-vendor href="/java/spring" src="/images/spring-logo.png">}}
<!-- {{<card-vendor href="/java/jetty" src="/images/jetty-logo.png">}}
{{<card-vendor href="/java/grpc" src="/images/grpc-logo.png">}}
{{<card-vendor href="/java/tomcat" src="/images/tomcat-logo.png">}} -->

### Requirements

- Java 8+
- Successfully installed [sqlcommenter-java](/java/#install)

### Using the integration

You can include this integration in your Java programs in 2 ways

#### Persistance XML file

By simply adding to your persistence XML file the property
`"hibernate.session_factory.statement_inspector"`

e.g. to your `hibernate.cfg.xml` file

{{<highlight xml>}}
<property name="hibernate.session_factory.statement_inspector"
          value="io.orijtech.integrations.sqlcommenter.schibernate.SCHibernate" />
{{</highlight>}}

#### In Java source code

When creating your Hibernate session factory, add our StatementInspector like this:

{{<highlight java>}}
import io.orijtech.integrations.sqlcommenter.schhibernate.SCHibernate;

...
        sessionFactoryBuilder.applyStatementInspector(new SCHibernate());
{{</highlight>}}

### End to end example

After following the example below you should be able to see in your PostgreSQL logs something that looks like the following
```shell
2019-06-12 15:43:23.260 PDT [34305] LOG:  execute <unnamed>:
select question0_.id as id1_0_, question0_.question_text as question2_0_,
question0_.pub_date as pub_date3_0_ from polls_question question0_
/*span_id='bcd07633524964ba',trace_id='e6588efe3fe58814f2afc0c63a2d9c55'*/
```

{{% notice tip %}}
Obviously, when augmented with other frameworks you'll see more information e.g. with Spring.
{{%/ notice %}}

#### Directory structure
Please ensure that at the end, your directory mirrors the structure below:

```shell
.
├── pom.xml
├── settings.gradle
├── src
│   └── main
│       ├── java
│       │   └── io
│       │       └── orijtech
│       │           └── quickstarts
│       │               └── sqlcommenter
│       │                   └── hibernate
│       │                       ├── ListQuestions.java
│       │                       └── Question.java
│       └── resources
│           ├── Question.hbm.xml
│           └── hibernate.cfg.xml
```

#### Source code
{{<tabs ListQuestions_Java Question_Java>}}
{{<highlight java>}}
// In file ListQuestions.Java
package io.orijtech.quickstarts.sqlcommenter.hibernate;

import java.util.ArrayList;
import java.util.List;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class ListQuestions {
  private static SessionFactory sessionFactory;

  public static void main(String[] args) throws Exception {
    sessionFactory = new Configuration().configure().buildSessionFactory();

    ListQuestions lister = new ListQuestions();
    List<Question> allQuestions = lister.listAll();

    for (int i = 1; i < 1000; i++) {
      for (Question qn : allQuestions) {
        System.out.println(
            String.format(
                "Question(%d) Text: %s PublishDate: %s",
                qn.getId(), qn.getText(), qn.getPublishDate()));
      }
      Thread.sleep(1000);
      System.out.println("Sleeping " + i);
    }

    System.exit(0);
  }

  public List<Question> listAll() throws Exception {
    List<Question> questions = new ArrayList<Question>();

    try (Session session = sessionFactory.openSession()) {
      questions = session.createQuery("FROM Question").list();
    } catch (Exception e) {
      throw e;
    }

    return questions;
  }
}
{{</highlight>}}

{{<highlight java>}}
// In file Question.Java
package io.orijtech.quickstarts.sqlcommenter.hibernate;

import java.sql.Date;

public class Question {
  private int id;
  private String text;
  private Date publishDate;

  public Question() {}

  public Question(String text, Date publishDate) {
    this.text = text;
    this.publishDate = publishDate;
  }

  public int getId() {
    return this.id;
  }

  public void setId(int id) {
    this.id = id;
  }

  public String getText() {
    return this.text;
  }

  public void setText(String text) {
    this.text = text;
  }

  public Date getPublishDate() {
    return this.publishDate;
  }

  public void setPublishDate(Date publishDate) {
    this.publishDate = publishDate;
  }
}
{{</highlight>}}
{{</tabs>}}

#### XML files
{{<tabs hibernate_cfg_xml Question_hbm_xml pom_xml>}}
{{<highlight xml>}}
<!-- Please place this file under: src/main/resources/hibernate.cfg.xml -->
<?xml version = "1.0" encoding = "utf-8"?>
<!DOCTYPE hibernate-configuration SYSTEM
"http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <property name = "hibernate.dialect">
            org.hibernate.dialect.PostgreSQLDialect
        </property>

        <property name = "hibernate.connection.driver_class">
            org.postgresql.Driver
        </property>


        <property name = "hibernate.connection.url">
            jdbc:postgresql://localhost:5432/quickstart_py
        </property>

        <property name = "hibernate.connection.password">
            $postgres$
        </property>

        <property name = "hibernate.connection.pool_size">1</property>

        <property name="hibernate.session_factory.statement_inspector">
            io.orijtech.integrations.sqlcommenter.schibernate.SCHibernate
        </property>

        <mapping resource = "Question.hbm.xml" />
    </session-factory>
</hibernate-configuration>
{{</highlight>}}

{{<highlight xml>}}
<!-- Please place this file under: src/main/resources/Question.cfg.xml -->
<?xml version = "1.0" encoding = "utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
"-//Hibernate/Hibernate Mapping DTD//EN"
"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping>
    <class name = "io.orijtech.quickstarts.sqlcommenter.hibernate.Question" table = "polls_question">
        <meta attribute = "class-description">
            Details about questions made in the poll.
        </meta>

        <id name = "id" type="int" column = "id">
            <generator class="native" />
        </id>

        <property name = "text" type="string" column = "question_text" />
        <property name = "publishDate" type="date" column = "pub_date" />
    </class>
</hibernate-mapping>
{{</highlight>}}

{{<highlight xml>}}
<!-- Please place this file under: pom.xml -->
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>io.orijtech</groupId>
    <artifactId>sqlcommenter-hibernate</artifactId>
    <packaging>jar</packaging>
    <version>0.0.1</version>
    <name>sqlcommenter-hibernate-app</name>
    <url>http://maven.apache.org</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <opencensus.version>0.21.0</opencensus.version>
        <hibernate.version>5.4.3.Final</hibernate.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>io.opencensus</groupId>
            <artifactId>opencensus-api</artifactId>
            <version>${opencensus.version}</version>
        </dependency>

        <dependency>
            <groupId>io.opencensus</groupId>
            <artifactId>opencensus-impl</artifactId>
            <version>${opencensus.version}</version>
        </dependency>

        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>${hibernate.version}</version>
        </dependency>

        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.2.5</version>
        </dependency>

        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>${hibernate.version}</version>
        </dependency>

        <dependency>
            <groupId>io.orijtech.integrations</groupId>
            <artifactId>sqlcommenter-java</artifactId>
            <version>0.0.1</version>
        </dependency>
    </dependencies>

    <build>
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>1.5.0.Final</version>
            </extension>
        </extensions>

        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.7.0</version>
                    <configuration>
                        <source>1.8</source>
                        <target>1.8</target>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>

        <plugins>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>appassembler-maven-plugin</artifactId>
                <version>1.10</version>
                <configuration>
                    <programs>
                        <program>
                            <id>ListQuestions</id>
                            <mainClass>io.opencensus.quickstarts.sqlcommenter.hibernate.ListQuestions</mainClass>
                        </program>
                    </programs>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
{{</highlight>}}
{{</tabs>}}

### References

Resource|URL
---|---
Hibernate ORM project|https://hibernate.org/orm/
sqlcommenter-java on Github|https://github.com/orijtech/sqlcommenter-java
