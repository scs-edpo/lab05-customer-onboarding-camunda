# lab05-customer-onboarding-camunda

This project illustrates how to use a process engine, namely the Camunda BPM Platform, programmatically within a Spring Boot application.

## Process Solution

The main process solution is the [process-solution-java](/process-solution-java) SpringBoot application.

The [pom file](/process-solution-java/pom.xml) specifies the corresponding dependencies for the Camunda Spring Boot project.

Running the main [Application](/process-solution-java/src/main/java/com/example/processsolutionjava/CustomerOnboardingCamundaApplication.java) starts
the SpringBoot application with an embedded version of the Camunda BPM platform locally on your computer. You can access its [user interface](http://localhost:8080/) via your web browser.
Use the credentials: *demo* / *demo* to login. After login you see the following overview:

![camunda.png](camunda.png)

### Modeling and Deploying Processes

To model processes in BPMN 2.0 we recommend using the [Camunda Modeler](https://camunda.com/de/download/modeler/) desktop application. The following is a screenshot
of the [onboarding](/process-solution-java/src/main/resources/onboarding.bpmn) process provided with this project opened in the Modeler. 

![modeler.png](modeler.png)

All *bpmn* files within
the [resources](/process-solution-java/src/main/resources) will be automatically deployed to the Camunda platform upon start of the application. You can
deploy additional processes from the Modeler via the "Deploy Current Diagram" button, specifying as endpoint:

``http://127.0.0.1:8080/engine-rest/deployment/create``

### Starting the Process
The [CustomerOnBoardingRestController](/process-solution-java/src/main/java/com/example/processsolutionjava/controller/CustomerOnboardingRestController.java) provides
a REST endpoint at 

``localhost:8080/customer``.

The SpringBoot application loads a small web page (see [onboarding.html](/process-solution-java/src/main/resources/static/onboarding.html)) for entering the name of a customer via a [web form](http://localhost:8080/onboarding.html) at

``http://localhost:8080/onboarding.html``,

which then creates a POST request with the provided name es payload to the REST controller.

This then starts a new process instance based on the name (key) of the process: *onboarding*. Alternatively you can start the
process via the Camunda Web UI: in the Tasklist, select ''Start Process'', chose the process and enter the customer name as process variable:

![start.png](start.png)

## External Task Worker
The second part of this lab is the [External Task Worker](/worker-java), which is called from the BPMN 2.0 process in the activity "Create customer order in CRM system".
The [application.yml](/worker-java/src/main/resources/application.yml) file defines the properties related to the Camunda BPM platform to use and the
subscriptions to topic names from Camunda to receive external tasks on (here: *crmEntry*) as specified in the process model.
The class [CRMEntryTaskHandler](/worker-java/src/main/java/com/example/worker/CRMEntryTaskHandler.java) shows the main logic to be executed for this type of external task.

To successfully execute the entire process via the Camunda BPM platform, the SpringBoot application for the worker-java project also needs to be started after the CustomerOnboardingCamundaApplication via its main class [CRMEntryApplication](/worker-java/src/main/java/com/example/CRMEntryApplication.java).

## Migrating to Operaton

- Camunda has discontinued support for the Community Edition of Camunda 7 in order to focus on Camunda 8, its SaaS-oriented platform. The open-source codebase of Camunda 7 has since been forked and is now maintained by the Operaton project (https://operaton.org/).

- Operaton is therefore a continuation of Camunda 7 under a different name, with ongoing maintenance and development provided by the Operaton community. From a technical perspective, the core concepts, APIs, and architecture remain largely the same, which makes migration straightforward in most cases.

- This section provides guidelines on how to migrate an existing Camunda 7 project to Operaton. These steps can, for example, be applied to this repository. In addition, we have created a separate branch of this lab that already uses Operaton, which can be found at: [https://github.com/scs-edpo/lab05-customer-onboarding-camunda/tree/operaton-version](https://github.com/scs-edpo/lab05-customer-onboarding-camunda/tree/operaton-version)


### 1. Migrating the Process Solution Project (`process-solution-java`)

#### Update Dependencies (`pom.xml`)
In the [`process-solution-java/pom.xml`](/process-solution-java/pom.xml) file, update the Spring Boot version, the Camunda version variable, and the dependencies to their Operaton equivalents.

First, update the Spring Boot and Operaton version properties:
```xml
<!-- Change Spring Boot version -->
<spring-boot.version>3.5.10</spring-boot.version>

<!-- Replace camunda.spring-boot.version with operaton.spring-boot.version -->
<operaton.spring-boot.version>1.0.0</operaton.spring-boot.version>
```

Then, replace the Camunda dependencies with Operaton dependencies:
```xml
<!-- Replace camunda-bpm-spring-boot-starter-webapp and camunda-bpm-spring-boot-starter-rest -->
<dependency>
    <groupId>org.operaton.bpm.springboot</groupId>
    <artifactId>operaton-bpm-spring-boot-starter-webapp</artifactId>
    <version>${operaton.spring-boot.version}</version>
    <exclusions>
        <exclusion>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.operaton.bpm.springboot</groupId>
    <artifactId>operaton-bpm-spring-boot-starter-rest</artifactId>
    <version>${operaton.spring-boot.version}</version>
    <exclusions>
        <exclusion>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

#### Update Java Imports
Change the package imports from `org.camunda.bpm` to `org.operaton.bpm` in the following files:
- [`CustomerOnboardingRestController.java`](/process-solution-java/src/main/java/com/example/processsolutionjava/controller/CustomerOnboardingRestController.java)
- [`ScoreCustomerAdapter.java`](/process-solution-java/src/main/java/com/example/processsolutionjava/process/ScoreCustomerAdapter.java)

#### Update Application Properties
Update the configuration properties to use the Operaton prefix instead of Camunda. Change `camunda.bpm` to `operaton.bpm` in:
- [`process-solution-java/src/main/resources/application.yaml`](/process-solution-java/src/main/resources/application.yaml)


### 2. Migrating the Worker Project (`worker-java`)

#### Update Dependencies (`pom.xml`)
In the [`worker-java/pom.xml`](/worker-java/pom.xml) file, update the Spring Boot version and the external task client dependency.

First, update the Spring Boot and Operaton version properties:
```xml
<!-- Change Spring Boot version -->
<spring-boot.version>3.5.10</spring-boot.version>

<!-- Replace camunda.spring-boot.version with operaton.spring-boot.version -->
<operaton.spring-boot.version>1.0.0</operaton.spring-boot.version>
```

Then, replace the Camunda external task client dependency with the Operaton equivalent:
```xml
<!-- Replace camunda-bpm-spring-boot-starter-external-task-client -->
<dependency>
    <groupId>org.operaton.bpm.springboot</groupId>
    <artifactId>operaton-bpm-spring-boot-starter-external-task-client</artifactId>
    <version>${operaton.spring-boot.version}</version>
    <exclusions>
        <exclusion>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

#### Update Java Imports
Change the package imports from `org.camunda.bpm` to `org.operaton.bpm` in the following file:
- [`CRMEntryTaskHandler.java`](/worker-java/src/main/java/com/example/worker/CRMEntryTaskHandler.java)

#### Update Application Properties
Update the configuration properties to use the Operaton prefix instead of Camunda. Change `camunda.bpm` to `operaton.bpm` in:
- [`worker-java/src/main/resources/application.yml`](/worker-java/src/main/resources/application.yml)


### 3. Rebuild the Projects

Finally, clean and rebuild both projects to ensure all changes are applied correctly:

```bash
# In process-solution-java directory
mvn clean install

# In worker-java directory
mvn clean install
```

