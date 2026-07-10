# Cookbook & Examples

Recipes and examples for building complex workflows including native and device applications.

## Recipes

### Minimal device app

Below you can find a very minimal example of a workload targeted to a Device Node with name `my-device`. This device is a registered Device Node in the cluster and also supports docker image runtime. All possible attributes are explained in the [Device Apps reference](dsl/devices.md).

```yaml
apiVersion: hyper.ai/v1
kind: Application
metadata:
  name: my-app
  namespace: default
spec:
  device_name: my-device
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

