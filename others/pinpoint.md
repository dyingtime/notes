## Introduction

**Pinpoint** is an APM (Application Performance Management) tool for large-scale distributed systems written in Java / [PHP](https://github.com/pinpoint-apm/pinpoint-c-agent). Inspired by [Dapper](http://research.google.com/pubs/pub36356.html), Pinpoint provides a solution to help analyze the overall structure of the system and how components within them are interconnected by tracing transactions across distributed applications.

You should definitely check **Pinpoint** out If you want to

- understand your application topology at a glance
- monitor your application in *Real-Time*
- gain code-level visibility to every transaction
- install APM Agents without changing a single line of code
- have minimal impact on the performance (approximately 3% increase in resource usage)

## install pinpoint server

```shell
git ctone https://github.com/headless-dev/pinpoint-kubernetes 
cd pinpoint-kubernetes
# create namespace
kubectl create ns pinpoint
# install pinpoint
helm install pinpoint ./pinpoint -n pinpoint
```

default service type of pinpoint collector was `ClusterIp`, it exposes the service on a cluster-internal IP. Choosing this value makes the service only reachable from within the cluster. But sometime, service many be need deployment in a anther cluster, so we need to change service type to `NodeType`

```yaml
spec:
  # change type to NodePort
  type: NodePort
  ports:
  - port: {{ .Values.receiver.grpc.agentPort}}
    targetPort: 9991
    name: grpc-agent
    # default port range 30000-32767
    nodePort: 31991
  - port: {{ .Values.receiver.grpc.statPort}}
    targetPort: 9992
    name: grpc-stat
    nodePort: 31992
  - port: {{ .Values.receiver.grpc.spanPort}}
    targetPort: 9993
    name: grpc-span
    nodePort: 31993
  - port: {{ .Values.receiver.thrift.basePort}}
    targetPort: 9994
    name: thrift-base
    nodePort: 31994
  - port: {{ .Values.receiver.thrift.statPort}}
    targetPort: 9995
    name: thrift-stat
    nodePort: 31995
  - port: {{ .Values.receiver.thrift.spanPort}}
    targetPort: 9996
    name: thrift-span
    nodePort: 31996
```

## agent installtion

### when using released binary

Pinpoint Agent runs as a java agent attached to an application to be profiled (such as Tomcat).

To wire up the agent, pass `$AGENT_PATH/pinpoint-bootstrap-$VERSION.jar* to the *-javaagent` JVM argument when running the application:

```shell
-javaagent:$AGENT_PATH/pinpoint-bootstrap-$VERSION.jar
```

Additionally, Pinpoint Agent requires 2 command-line arguments in order to identify itself in the distributed system

- `-Dpinpoint.agentId` - uniquely identifies the application instance in which the agent is running on
- `-Dpinpoint.applicationName` - groups a number of identical application instances as a single service

Note that `pinpoint.agentId` must be globally unique to identify an application instance, and all applications that share the same `pinpoint.applicationName` are treated as multiple instances of a single service.

If you're launching the agent in a containerized environment, you might have set your *agent id* to be auto-generated every time the container is launched. With frequent deployment and auto-scaling, this will lead to the Web UI being cluttered with all the list of agents that were launched and destroyed previously. For such cases, you might want to add `-Dpinpoint.container` in addition to the 2 required command-line arguments above when launching the agent

#### quickstart

Download Pinpoint from [Latest Release](https://github.com/pinpoint-apm/pinpoint/releases/latest).

Extract the downloaded file.

```shell
tar xvzf pinpoint-agent-2.2.1.tar.gz
```

Run the JAR file, as follows:

```shell
java -jar -javaagent:$AGENT_PATH/pinpoint-bootstrap-2.4.2.jar
    -Dpinpoint.applicationName=<your_application_name>
    -Dpinpoint.agentId=<your_agent_id>
    -Dpinpoint.profiler.profiles.active=release
    -Dprofiler.collector.ip=<collector_ip>
    -Dprofiler.transport.grpc.collector.ip=<collector_ip>
    -Dprofiler.transport.grpc.agent.collector.port=31991
    -Dprofiler.transport.grpc.metadata.collector.port=31991
    -Dprofiler.transport.grpc.stat.collector.port=31992
    -Dprofiler.transport.grpc.span.collector.port=31993
    -Dprofiler.collector.tcp.port=31994
    -Dprofiler.collector.stat.port=31995
    -Dprofiler.collector.span.port=31996
    pinpoint-quickstart-testapp-2.2.1.jar
```

### when using helm

init a demo application

```shell
helm init demo
```

- add `initContainers` to your pods

```yaml
initContainers:
  - name: pinpoint-agent
    securityContext:
        {{- toYaml .Values.securityContext | nindent 12 }}
    imagePullPolicy: IfNotPresent
    image: "pinpointdocker/pinpoint-agent:2.4.2"
    volumeMounts:
    - mountPath: /pinpoint-agent2
      name: pinpoint-agent-volume
    env:
      - name: SPRING_PROFILES
        value: release
      - name: COLLECTOR_IP
        value: aqa01-i01-k8s02.lab.nordigy.ru
      - name: PROFILER_TRANSPORT_AGENT_COLLECTOR_PORT
        value: "31991"
      - name: PROFILER_TRANSPORT_METADATA_COLLECTOR_PORT
        value: "31991"
      - name: PROFILER_TRANSPORT_STAT_COLLECTOR_PORT
        value: "31992"
      - name: PROFILER_TRANSPORT_SPAN_COLLECTOR_PORT
        value: "31993"
      - name: COLLECTOR_TCP_PORT
        value: "31994"
      - name: COLLECTOR_STAT_PORT
        value: "31995"
      - name: COLLECTOR_SPAN_PORT
        value: "31996"
      - name: PROFILER_SAMPLING_TYPE
        value: COUNTING
      - name: PROFILER_SAMPLING_COUNTING_SAMPLING_RATE
        value: "1"
      - name: PROFILER_SAMPLING_PERCENT_SAMPLING_RATE
        value: "100"
      - name: PROFILER_SAMPLING_NEW_THROUGHPUT
        value: "0"
      - name: PROFILER_SAMPLING_CONTINUE_THROUGHPUT
        value: "0"
    command:
      - sh
      - '-c'
      - /usr/local/bin/configure-agent.sh;cp -R /pinpoint-agent /pinpoint-agent2;
```

- Add this to pod volumes

```yaml
volumes:
  - emptyDir: {}
    name: pinpoint-agent-volume
```

- Add this to your main container volume mounts

```yaml
volumeMounts:
  - mountPath: /pinpoint-agent2
    name: pinpoint-agent-volume
```

- Add this to JVM args

```yaml
env:
  - name: JAVA_TOOL_OPTIONS
    value: >|
      -javaagent:/pinpoint-agent2/pinpoint-agent/pinpoint-bootstrap-2.4.2.jar
      -Dpinpoint.applicationName=<your_application_name>
      -Dpinpoint.profiler.profiles.active=release
```

## when using swck

Earlier, we explained how to integrate with pinpoint agent for a single application.One way for implementing this is that developers to modify their code by themselves. However, services nowadays often consist of many different components and it could be a burden to modify code even though such functionality is useful to developers.

But Unfortunately, pinpoint didn't provide a solution for this. but we can use [swck](https://github.com/apache/skywalking-swck) to achieve the purpose.

For more details about swck, please refer to [java agent injector](https://github.com/apache/skywalking-swck/blob/master/docs/java-agent-injector.md)

#### quickstart

add annotations and labels to target deployments as follows:

```yaml
template:
    metadata:
      annotations:
        sidecar.skywalking.apache.org/env.Value: >-
          -javaagent:/pinpoint-agent2/pinpoint-agent/pinpoint-bootstrap-2.4.2.jar
          -Dpinpoint.applicationName=<your_application_name>
          -Dpinpoint.profiler.profiles.active=release
          -Dprofiler.collector.ip=<collector_ip>
          -Dprofiler.transport.grpc.collector.ip=<collector_ip>
          -Dprofiler.transport.grpc.agent.collector.port=31991
          -Dprofiler.transport.grpc.metadata.collector.port=31991
          -Dprofiler.transport.grpc.stat.collector.port=31992
          -Dprofiler.transport.grpc.span.collector.port=31993
          -Dprofiler.collector.tcp.port=31994
          -Dprofiler.collector.stat.port=31995
          -Dprofiler.collector.span.port=31996
        sidecar.skywalking.apache.org/initcontainer.Image: 'pinpointdocker/pinpoint-agent:2.4.2'
        sidecar.skywalking.apache.org/initcontainer.Name: inject-pinpoint-agent
        sidecar.skywalking.apache.org/initcontainer.args.Command: cp -R /pinpoint-agent /pinpoint-agent2;
        sidecar.skywalking.apache.org/sidecarVolume.Name: pinpoint-agent
        sidecar.skywalking.apache.org/sidecarVolumeMount.MountPath: /pinpoint-agent2
      labels:
        # set value to 'false' when you wang to uninstall pinpoint
        swck-java-agent-injected: 'true'
```

or add `podAnnotations` in `values.yaml` file

```yaml
# .....
podAnnotations:
  -javaagent:/pinpoint-agent2/pinpoint-agent/pinpoint-bootstrap-2.4.2.jar
  -Dpinpoint.applicationName=<your_application_name>
  -Dpinpoint.profiler.profiles.active=release
  -Dprofiler.collector.ip=<collector_ip>
  -Dprofiler.transport.grpc.collector.ip=<collector_ip>
  -Dprofiler.transport.grpc.agent.collector.port=31991
  -Dprofiler.transport.grpc.metadata.collector.port=31991
  -Dprofiler.transport.grpc.stat.collector.port=31992
  -Dprofiler.transport.grpc.span.collector.port=31993
  -Dprofiler.collector.tcp.port=31994
  -Dprofiler.collector.stat.port=31995
  -Dprofiler.collector.span.port=31996
```
