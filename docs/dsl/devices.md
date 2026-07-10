# Defining Device Node Applications

An **Application** is a Kubernetes custom resource (`kind: Application`, API group `hyper.ai/v1`) that describes a deployable workload:

- an **Android** APK
- a **Docker image**
- an **ESP32 firmware** binary

along with the device it should run on and the constraints that govern its deployment.

---

## Field Reference
Below we provide all ***mandatory*** (✅) and ***optional*** (❌) fields that are registered in the Hyper-AI platform. The user can also provide additional fields and attributes, which will be forwarded and be accessible by their application.

### `spec` — Device Targeting
The attributes below are not required. Specifically it is possible to select a Device Node from the IDE UI, and this will fill the `device_name` attribute. Regarding the `device_uuid`, **you should not alter or set it**.

| Field | Type | Required | Example |
|---|---|---|---|
| `device_name` | string | ❌ | `"edge-device-01"` — if omitted, the scheduler selects the best available device |
| `device_uuid` | string | ❌ | `"40637429-b8ed-4817-b7fd-f32bc6834d9b"` — set automatically by the platform |

---

### `spec.app` — Application Identity

| Field | Type | Required | Example |
|---|---|---|---|
| `app.type` | string, enum: `device` | ✅ | `device` |
| `app.schemaVersion` | string | ✅ | `"1.0.0"` |
| `app.name` | string | ✅ | `"vision-capture"` |
| `app.version` | string | ✅ | `"2.3.0"` |
| `app.owner` | string | ✅ | `"Edge Vision Team"` |
| `app.lifecyclePhase` | string, enum: `development` `testing` `production` | ✅ | `"production"` |
| `app.description` | string | ❌ | `"Captures frames and emits detections."` |
| `app.parentApp.name` | string | ❌ | `"parent-app"` |
| `app.parentApp.uid` | string | ❌ | `"abc-123"` |

---

### `spec.workload` — Workload Definition

The `workload.kind` field determines which parameter block must be provided: exactly one of `androidApk`, `dockerImage`, or `esp32Binary`, matching the selected kind. Each block has its own set of fields, described below.

| Field | Type | Required | Example |
|---|---|---|---|
| `workload.kind` | string, enum: `AndroidApk` `DockerImage` `esp32Binary` | ✅ | `"DockerImage"` |

#### AndroidApk

| Field | Type | Required | Example |
|---|---|---|---|
| `workload.androidApk.apkUrl` | string (URI) | ✅ | `"https://example.com/app.apk"` |
| `workload.androidApk.packageName` | string | ✅ | `"com.example.myapp"` |
| `workload.androidApk.sha256` | string | ❌ | `"a3f5c..."` |
| `workload.androidApk.installMode` | string, enum: `install` `update` | ❌ | `"update"` |
| `workload.androidApk.launch.activity` | string | ❌ | `"com.example.myapp/.MainActivity"` |
| `workload.androidApk.launch.action` | string | ❌ | `"android.intent.action.MAIN"` |
| `workload.androidApk.launch.category` | string | ❌ | `"android.intent.category.LAUNCHER"` |

#### DockerImage

| Field | Type | Required | Example |
|---|---|---|---|
| `workload.dockerImage.image` | string | ✅ | `"nginx:latest"` |
| `workload.dockerImage.imagePullPolicy` | string, enum: `Always` `IfNotPresent` `Never` | ❌ | `"IfNotPresent"` |
| `workload.dockerImage.imagePullSecretRef` | string | ❌ | `"registry-credentials"` |

#### esp32Binary

| Field | Type | Required | Example |
|---|---|---|---|
| `workload.esp32Binary.binaryUrl` | string (URI) | ✅ | `"https://example.com/firmware.bin"` |
| `workload.esp32Binary.chip` | string, enum: `esp32` `esp32s2` `esp32s3` `esp32c3` `esp32c6` `esp32h2` | ✅ | `"esp32s3"` |
| `workload.esp32Binary.flash.method` | string, enum: `serial` `ota` | ✅ | `"ota"` |
| `workload.esp32Binary.sha256` | string | ❌ | `"b2f4a..."` |
| `workload.esp32Binary.flash.port` | string | ❌ | `"/dev/ttyUSB0"` |
| `workload.esp32Binary.flash.baudRate` | integer (≥1200) | ❌ | `115200` |
| `workload.esp32Binary.flash.offset` | string (hex) | ❌ | `"0x10000"` |
| `workload.esp32Binary.flash.partition` | string | ❌ | `"app0"` |
| `workload.esp32Binary.flash.eraseFlash` | boolean | ❌ | `true` |

---

### `spec.exec` — Execution Parameters
These attributes can be used to pass environmental variables inside the application deployed.

| Field | Type | Required | Example |
|---|---|---|---|
| `exec.parameters` | map[string]string | ❌ | `STREAM_URL: "rtsp://10.0.0.5/live"` |
| `exec.command` | array of strings | ❌ | `["/app/start", "--verbose"]` |
| `exec.env` | map[string]string | ❌ | `LOG_LEVEL: "info"` |
| `exec.workingDir` | string | ❌ | `"/app"` |

---

### `spec.resources` — Resource Requirements
These are additional attributes for required describing resources. They are important and taken into consideration, in case **the user does not opt for a specific Device Node during submission from the IDE**s (they are used for scheduling the application to the appropriate Device Node).

| Field | Type | Required | Example |
|---|---|---|---|
| `resources.cpu.value` | number | ❌ | `250` |
| `resources.cpu.unit` | string, enum: `cores` `millicores` | ❌ | `"millicores"` |
| `resources.memory.value` | number | ❌ | `256` |
| `resources.memory.unit` | string, enum: `MiB` `GiB` etc. | ❌ | `"MiB"` |
| `resources.storage.value` | number | ❌ | `1` |
| `resources.storage.unit` | string, enum: `GiB` etc. | ❌ | `"GiB"` |
| `resources.gpu` | integer (≥0) | ❌ | `1` |
| `resources.tpu` | integer (≥0) | ❌ | `0` |
| `resources.accelerators` | array of strings | ❌ | `["cuda", "tensorrt"]` |

---

### `spec.network` — Network Requirements

| Field | Type | Required | Example |
|---|---|---|---|
| `network.ports[].port` | integer (1–65535) | ✅ | `8080` |
| `network.ports[].protocol` | string | ✅ | `"HTTP"` |
| `network.networkBandwidthMin.value` | number | ✅ | `10` |
| `network.networkBandwidthMin.unit` | string, enum: `bps` `Kbps` `Mbps` `Gbps` | ✅ | `"Mbps"` |

---

### `spec.qos` — Quality of Service

| Field | Type | Required | Example |
|---|---|---|---|
| `qos.latencyToleranceMax.value` | number | ✅ | `150` |
| `qos.latencyToleranceMax.unit` | string, enum: `ms` `s` | ✅ | `"ms"` |
| `qos.energyCost.value` | number | ✅ | `5` |
| `qos.energyCost.unit` | string, enum: `mW` `W` | ✅ | `"W"` |
| `qos.monetaryCost.value` | number | ✅ | `0.05` |
| `qos.monetaryCost.currency` | string | ✅ | `"USD"` |
| `qos.monetaryCost.per` | string, enum: `second` `minute` `hour` `day` | ✅ | `"hour"` |
| `qos.resilience` | string | ✅ | `"auto-restart"` |
| `qos.availability.value` | number (0–1) | ✅ | `0.95` |
| `qos.availability.unit` | string, enum: `fraction` | ✅ | `"fraction"` |
| `qos.startupTime.value` | number | ✅ | `3` |
| `qos.startupTime.unit` | string, enum: `ms` `s` | ✅ | `"s"` |

---

### `spec.constraints` — Scheduling Constraints

| Field | Type | Required | Example |
|---|---|---|---|
| `constraints.schedulingPriority` | integer | ✅ | `5` |
| `constraints.supportedArchitectures` | array of strings | ✅ | `["arm64-v8a", "amd64"]` |
| `constraints.geoLocationRequirement` | string | ✅ | `"LocalZone"` |
| `constraints.isHighlyAvailable` | boolean | ✅ | `false` |
| `constraints.faultTolerance` | string | ✅ | `"graceful-degradation"` |
| `constraints.dataClassification` | string | ✅ | `"private"` |
| `constraints.trustScore` | number (≥0) | ❌ | `90.0` |
| `constraints.batteryLevelMin` | integer (0–100) | ❌ | `20` |
| `constraints.securityLevel` | string | ❌ | `"high"` |

---

### `spec.sensors` — Sensor Requirements

| Field | Type | Required | Example |
|---|---|---|---|
| `sensors[].sensorType` | string | ✅ | `"temperature"` |
| `sensors[].isActive` | boolean | ✅ | `true` |
| `sensors[].taskStatus` | string, enum: `IDLE` `RUNNING` `SCHEDULED` | ✅ | `"IDLE"` |
| `sensors[].sensorIDs` | array of strings | ❌ | `["sensor-001"]` |
| `sensors[].platformInfo` | string | ❌ | `"i2c-bus-1"` |

---

### `spec.runtime` — Runtime Requirements

| Field | Type | Required | Example |
|---|---|---|---|
| `runtime.baseOS.name` | string | ❌ | `"Ubuntu"` |
| `runtime.baseOS.version` | string | ❌ | `"22.04"` |
| `runtime.acceleratorRuntime[].name` | string | ❌ | `"cuda"` |
| `runtime.hypervisor` | string | ❌ | `"kvm"` |

---

### `spec` — Miscellaneous

| Field | Type | Required | Example |
|---|---|---|---|
| `logs_url` | string (URI) | ❌ | `"https://logs.example.com/my-app"` |
| `metrics_url` | string (URI) | ❌ | `"https://metrics.example.com/my-app"` |
| `config` | free-form object | ❌ | see [Custom Configuration Fields](#custom-configuration-fields) |

---

### `status` — Deployment Status (Read-only)

Unlike the `spec` fields above, `status` is not set by the user. The platform updates `status.phase` automatically as the application progresses through its lifecycle:

| Phase | Meaning |
|---|---|
| `pending` | Deployment requested, waiting for device ACK |
| `scheduled` | Device selected, workload being sent |
| `deployed` | Device acknowledged successful deployment |
| `failed` | Deployment failed or timed out |
| `removed` | Application was deleted and device acknowledged removal |

---

## Custom Configuration Fields

> **Note:** The `spec.config` field accepts any key-value structure, allowing you to pass arbitrary configuration to your application without modifying the schema. This is useful for application-specific settings that don't fit into the standard fields.

Examples:

```yaml
spec:
  config:
    mqtt_broker: "mqtt://edge-broker:1883"
    sampling_rate: "100ms"
    model_path: "/models/person-v3.onnx"
    debug: "true"
```

```yaml
spec:
  config:
    camera_index: "0"
    resolution: "1920x1080"
    output_topic: "detections/camera-0"
```

---

## Examples

See [Cookbook & Examples](../cookbook.md#device-applications) for minimal device app examples.
