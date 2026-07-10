# Cookbook & Examples

Recipes and examples for building complex workflows including native and device applications.

## Native Applications

The examples below are native Applications — workloads defined through an application profile. All available attributes are explained in the [Native Apps reference](dsl/native-apps.md).

### Hello World Web Server — nginx

A workflow that deploys an nginx server. The user expresses their preferences through specific constraints and QoS requirements.

```yaml
applicationProfile:
  metadata:
    type: "native"
    schemaVersion: "1.1.0"
    name: "hello-world-webserver"
    version: "1.0.0"
    description: "Nginx web server for HyperAI platform assessment."
    owner: "UoA-Team"
    lifecyclePhase: "development"
    annotations:
      intent: "platform-demo"
      domain: "education"
      language: "Python 3.10"
      dependencies: ["fastapi", "uvicorn"]

  specs:
    runtime:
      executionType: "container"
      entryPoint: "uvicorn"
      args: ["main:app", "--host", "0.0.0.0", "--port", "8000"]
      baseOS:
        name: "python"
        version: "3.10-slim"
      containerImage:
        uri: "nginx"
        tag: "latest"

    resources:
      cpu: "2000m"
      memory: "10Gi"
      storage: "1Gi"

    network:
      ports:
        - port: 8000
          protocol: "TCP"
          publicExposure: true
      protocols: ["HTTP"]

    constraints:
      supportedArchitectures: ["x86_64", "arm64"]
      securityLevel: "high"
      dataClassification: "private"
      isHighlyAvailable: false

    qos:
      startupTime: "10s"
      availability: "99.0%"
```

## Device Applications

The examples below are device Applications — workloads deployed to Device Nodes across the edge. All available attributes are explained in the [Device Apps reference](dsl/devices.md).

### Hello World — Docker

A minimal Docker-based application for testing device registration and scheduling. No `device_name` is specified — the platform scheduler will automatically select the best available device with Docker support.

```yaml
apiVersion: hyper.ai/v1
kind: Application
metadata:
  name: hello-world
  namespace: default
  annotations:
    intent: "Hello World test"
    domain: "edge-testing"
spec:
  app:
    type: device
    schemaVersion: "1.0.0"
    name: "hello-world"
    version: "1.0.0"
    description: "Hello World Docker image for basic connectivity testing."
    owner: "HyperAI Team"
    lifecyclePhase: "testing"
  workload:
    kind: DockerImage
    dockerImage:
      image: "hello-world"
      imagePullPolicy: "IfNotPresent"
  exec:
    parameters:
      MODE: "test"
  network:
    ports:
      - port: 80
        protocol: "HTTP"
    networkBandwidthMin: { value: 1, unit: "Mbps" }
  qos:
    latencyToleranceMax: { value: 500, unit: "ms" }
    energyCost: { value: 1, unit: "W" }
    monetaryCost: { value: 0.01, currency: "USD", per: "hour" }
    resilience: "auto-restart"
    availability: { value: 0.90, unit: "fraction" }
    startupTime: { value: 5, unit: "s" }
  constraints:
    schedulingPriority: 1
    supportedArchitectures: ["amd64", "arm64"]
    geoLocationRequirement: "LocalZone"
    isHighlyAvailable: false
    faultTolerance: "graceful-degradation"
    dataClassification: "internal"
```
### Web Server (nginx) — Docker

Below you can find a very minimal example of a workload targeted to a Device Node with name `my-device`. This device is a registered Device Node in the cluster and also supports docker image runtime. All possible attributes are explained in the [Device Apps reference](dsl/devices.md).

```yaml
apiVersion: hyper.ai/v1
kind: Application
metadata:
  name: my-app
  namespace: default
spec:
  device_name: pi-node-1
  app:
    type: device
    schemaVersion: "1.0.0"
    name: "my-app"
    version: "1.0.0"
    owner: "My Team"
    lifecyclePhase: "production"
  workload:
    kind: DockerImage
    dockerImage:
      image: "nginx:latest"
  network:
    ports:
      - port: 80
        protocol: "HTTP"
    networkBandwidthMin: { value: 1, unit: "Mbps" }
  qos:
    latencyToleranceMax: { value: 500, unit: "ms" }
    energyCost: { value: 5, unit: "W" }
    monetaryCost: { value: 0.01, currency: "USD", per: "hour" }
    resilience: "auto-restart"
    availability: { value: 0.95, unit: "fraction" }
    startupTime: { value: 5, unit: "s" }
  constraints:
    schedulingPriority: 5
    supportedArchitectures: ["amd64", "arm64"]
    geoLocationRequirement: "LocalZone"
    isHighlyAvailable: false
    faultTolerance: "graceful-degradation"
    dataClassification: "internal"
```


### Telemetry Collector — Docker

A production-grade Dockerized telemetry collector that exports metrics and logs. Demonstrates the use of resource requirements, execution parameters, and environment variables.

```yaml
apiVersion: hyper.ai/v1
kind: Application
metadata:
  name: telemetry-collector
  namespace: default
  annotations:
    intent: "Telemetry collection"
    domain: "edge-platform"
spec:
  app:
    type: device
    schemaVersion: "1.0.0"
    name: "telemetry-collector"
    version: "1.4.7"
    description: "Dockerized collector that exports metrics and logs."
    owner: "Platform Team"
    lifecyclePhase: "production"
  workload:
    kind: DockerImage
    dockerImage:
      image: "registry.example.com/edge/collector:1.4.7"
      imagePullPolicy: "IfNotPresent"
      imagePullSecretRef: "registry-credentials"
  exec:
    command: ["/app/collector"]
    workingDir: "/app"
    env:
      LOG_LEVEL: "info"
    parameters:
      config: "/etc/collector/config.yaml"
      port: "9100"
      scrapeIntervalSeconds: "15"
  resources:
    cpu: { value: 250, unit: "millicores" }
    memory: { value: 256, unit: "MiB" }
    storage: { value: 1, unit: "GiB" }
    gpu: 0
    tpu: 0
  network:
    ports:
      - port: 9100
        protocol: "HTTP"
    networkBandwidthMin: { value: 5, unit: "Mbps" }
  qos:
    latencyToleranceMax: { value: 500, unit: "ms" }
    energyCost: { value: 2, unit: "W" }
    monetaryCost: { value: 0.02, currency: "USD", per: "hour" }
    resilience: "auto-restart"
    availability: { value: 0.99, unit: "fraction" }
    startupTime: { value: 2, unit: "s" }
  constraints:
    schedulingPriority: 3
    supportedArchitectures: ["amd64", "arm64"]
    geoLocationRequirement: "LocalZone"
    isHighlyAvailable: true
    faultTolerance: "auto-restart"
    dataClassification: "internal"
```

> **Note:** You have to set a valid docker image tag in the `image` attribute, of your choosing.

### Vision Capture — Android APK

An Android APK application that captures video frames and emits object detections. Demonstrates Android workload configuration including APK URL, package name, launch intent, and execution parameters. Targets a specific device by name.

```yaml
apiVersion: hyper.ai/v1
kind: Application
metadata:
  name: vision-capture-android
  namespace: default
  annotations:
    intent: "Computer vision capture"
    domain: "edge-vision"
spec:
  device_name: smartphone-android-1
  app:
    type: device
    schemaVersion: "1.0.0"
    name: "vision-capture-android"
    version: "2.3.0"
    description: "Android APK that captures frames and emits detections."
    owner: "Edge Vision Team"
    lifecyclePhase: "production"
  workload:
    kind: AndroidApk
    androidApk:
      apkUrl: "https://downloads.example.com/apk/vision-capture-android-2.3.0.apk"
      packageName: "com.example.visioncapture"
      installMode: "update"
      launch:
        activity: "com.example.visioncapture/.MainActivity"
        action: "android.intent.action.MAIN"
        category: "android.intent.category.LAUNCHER"
  exec:
    parameters:
      STREAM_URL: "rtsp://10.0.0.5/live"
      DETECTION_MODEL: "person-v3"
      MODE: "production"
  network:
    ports:
      - port: 8080
        protocol: "HTTP"
    networkBandwidthMin: { value: 10, unit: "Mbps" }
  qos:
    latencyToleranceMax: { value: 150, unit: "ms" }
    energyCost: { value: 5, unit: "W" }
    monetaryCost: { value: 0.05, currency: "USD", per: "hour" }
    resilience: "auto-restart"
    availability: { value: 0.95, unit: "fraction" }
    startupTime: { value: 3, unit: "s" }
  constraints:
    schedulingPriority: 5
    supportedArchitectures: ["arm64-v8a"]
    geoLocationRequirement: "LocalZone"
    isHighlyAvailable: false
    faultTolerance: "graceful-degradation"
    dataClassification: "private"
```

> **Note:** You have to set a correct url in the `apkUrl` attribute, to point to an APK, of your choosing.
