# wavefront-opentracing-bundle-java

Welcome to the Wavefront Java Tracing Agent! 

* [Prerequisites](#Prerequisites)
* [Configuring the Parameters](#Configuring-the-Parameters)
* [Parameters](#Parameters)
* [Using the SpecialAgent](#Using-the-SpecialAgent)
* [Instrumenting a Java Based Container](#Instrumenting-a-Java-Based-Container)

The Wavefront Java Tracing Agent builds a fat-jar containing the [Wavefront by VMware OpenTracing
 Java SDK](https://github.com/wavefrontHQ/wavefront-opentracing-sdk-java) and a `TracerFactory` 
 implementation. It provides application observability without having to do any code changes. 
 
 **Tip**: *In this document, SpecialAgent refers to the [Java Agent for Open Tracing](https://github.com/opentracing-contrib/java-specialagent)*.
 
## Prerequisites

* [Download](https://github.com/opentracing-contrib/java-specialagent#2111-stable) the latest version of the Java SpecialAgent to your application's directory. 
* Java 7 or above.

## Configuring the Parameters

You can configure the [Tracer parameters](#Parameters) using any of the following methods.

* Create a file named `tracer.properties` in the application's directory and use the following template to configure the properties.

  This file is used to configure your Wavefront Tracer instance for reporting and to configure tags specific to the application.
  
  Example:
  ```properties
  # Required application tags
  wf.application=myApp
  wf.service=myService

  # Optional application tags
  wf.cluster=us-west
  wf.shard=primary

  # Reporting through direct ingestion
  wf.reportingMechanism=direct
  wf.server=<replace-with-wavefront-url>
  wf.token=<replace-with-wavefront-api-token>

  # Reporting with a Wavefront proxy
  #wf.reportingMechanism=proxy
  #wf.proxyHost=<replace-with-wavefront-proxy-hostname>
  #wf.proxyMetricsPort=2878
  #wf.proxyDistributionsPort=2878
  #wf.proxyTracingPort=30000
  ```

* Configure the parameters by overriding the existing parameters using System properties.

  Example:
  ```bash
  java -cp:$MYCLASSPATH:wavefront-opentracing-bundle-java.jar \
      -Dwf.service=MyOtherService \
      com.mycompany.MyOtherService
  ```

* Configure the Tracer parameters via YAML files. You need two YAML files:
  * One to 
[configure application tags](https://github.com/wavefrontHQ/wavefront-jersey-sdk-java#1-configure-application-tags).
  * Another to [configure Wavefront reporting](https://github.com/wavefrontHQ/wavefront-jersey-sdk-java#2-configure-wavefront-reporting). 

  The paths to these YAML files need be specified in `tracer.properties` or as System properties:

  Example: Define the paths to the YAML files via System porperties.
  ```bash
  java -cp:$MYCLASSPATH:wavefront-opentracing-bundle-java.jar \
      -Dwf.applicationTagsYamlFile=application-tags.yaml \
      -Dwf.reportingConfigYamlFile=wf-reporting-config.yaml \
      com.mycompany.MyService
  ```

  **Note**: The parameters configured via `tracer.properties` or System properties override the parameters configured via YAML files.

## Parameters

Wavefront Tracer parameters use the prefix `wf.`:

| Parameter | Description |
| --------- | ----------- |
| `wf.application`              | Name that identifies your application. Use the same value for all microservices in the same application. |
| `wf.service`                  | Name that identifies the microservice within your application. Use a unique value for each microservice. |
| `wf.cluster`                  | Name of a group of related hosts that serves as a cluster or region in which the application will run. |
| `wf.shard`                    | Name of a subgroup of hosts within a cluster. |
| `wf.customTags`               | Tags specific to your application, formatted as a delimited string of key-values. For example, `tagKey1,tagVal1,tagKey2,tagVal2` |
| `wf.customTagsFromEnv`        | Environment variables to load as tags, formatted as a delimited string of environment variable names to load values from and tag keys to map to. For example, `envVarName1,tagKey1,envVarName2,tagKey2` |
| `wf.customTagsDelimiter`      | Delimiter for `wf.customTags`. Default is `,` |
| `wf.reportingMechanism`       | `direct` or `proxy`. Sending data directly to Wavefront is the simplest way to get up and running quickly, whereas using a Wavefront proxy is the recommended choice for a large-scale deployment. |
| `wf.server`                   | URL for your Wavefront instance, typically `https://myCompany.wavefront.com` |
| `wf.token`                    | String produced by [obtaining an API token](https://docs.wavefront.com/wavefront_api.html#generating-an-api-token). You must have Direct Data Ingestion permission when you obtain the token. |
| `wf.proxyHost`                | String name or IP address of the host on which you set up the [Wavefront proxy](https://docs.wavefront.com/proxies.html). |
| `wf.proxyMetricsPort`         | Proxy port to send metrics to. Recommended value is 2878. Must match the value set for `pushListenerPorts=` in `wavefront.conf`. |
| `wf.proxyDistributionsPort`   | Proxy port to send histograms to. Recommended value is 2878. Must match the value set for `histogramDistListenerPorts=` in `wavefront.conf`. |
| `wf.proxyTracingPort`         | Proxy port to send trace data to. Recommended value is 30000. Must match the value set for `traceListenerPorts=` in `wavefront.conf`. |
| `wf.source`                   | String that represents where the data originates -- typically, the host name of the machine running the microservice. |
| `wf.applicationTagsYamlFile`  | Optional. Path of the [YAML file that configures application tags](https://github.com/wavefrontHQ/wavefront-jersey-sdk-java#1-configure-application-tags). |
| `wf.reportingConfigYamlFile`  | Optional. Path of the [YAML file that configures Wavefront reporting](https://github.com/wavefrontHQ/wavefront-jersey-sdk-java#2-configure-wavefront-reporting). |

## Using the SpecialAgent

The [Java OpenTracing SpecialAgent](https://github.com/opentracing-contrib/java-specialagent) 
automatically instruments Java applications with traces. To send these traces to Wavefront, the 
Wavefront Java Tracing Agent can be attached as a Tracer Plugin to the SpecialAgent at runtime. 
To do so, add the `-Dsa.tracer=${TRACER_JAR}` property to specify the JAR path of the bundle:

```bash
java -javaagent:opentracing-specialagent-1.4.0.jar \
    -Dsa.tracer=wavefront-opentracing-bundle-java-0.9.0.jar \
    -Dwf.service=myService \
    -jar MyService.jar
```

Starting with `v1.4.1` of the SpecialAgent, the Wavefront Java Tracing Agent comes packaged with
the SpecialAgent so you no longer need the standalone bundle JAR. To send traces to Wavefront,
add `-Dsa.tracer=wavefront`:

```bash
java -javaagent:opentracing-specialagent-1.4.1.jar \
    -Dsa.tracer=wavefront \
    -Dwf.service=myService \
    -jar MyService.jar
```
## Instrumenting a Java Based Container

You can instrument a Java-based container using the OpenTracing Special Agent. For more information, see [Container Instrumentation](/docs/container.md).
