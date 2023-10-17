# kubeconfig

aws eks --region us-east-1 update-kubeconfig --name meda-dev-yeti-eksdemotest


# Step 1: To Create a sample Application

## Creating Name space

kubectl create namespace otel-test-app

## Test the spring boot app

kubectl run my-app-pod springcommunity/spring-framework-petclinic:latest -n otel-test-app

kubectl expose pod my-app-pod --type=NodePort --port=80 --target-port=8080 --name=my-app-service -n otel-test-app


## Deployment

kubectl apply -f 1_depolyment.yaml -n otel-test-app

kubectl delete -f 1_depolyment.yaml -n otel-test-app


## Accessing the application

kubectl get node -o wide  -n otel-test-app  <external_ip>

kubectl get service -o wide -n otel-test-app  <cont_port>

 <ext_ip>:<cont_port>

# Step 2: OpenTelemetry Operator

Before we deploy the OpenTelemetry Operator, we will first start deploy another operator, namely the cert-manager.

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml 

kubectl delete -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml 

kubectl apply -f 2_cert_manager.yaml

kubectl apply -f opentelemetry-operator.yaml


# Step 3:  
  
Now that the cert-manager is deployed, we can begin installing the OpenTelemetry collector. This can be done by applying the operator manifest directly like this:

kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml

kubectl delete -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml

Once the operator is deployed, it will provide two custom resources:

1. Collector CRD (responsible for the collection, processing and transporting of the data)
2. Instrumentation CRD (responsible for auto instrumentation of the application)

 $ kubectl get crd

# Step 5: Deployment of central OpenTelemetry collector

In our strategy, we have chosen to use a central OpenTelemetry collector and have other OpenTelemetry agents send their data to this one. The data that is being received from the agents will be processed on this collector and sent over to the storage backend(s) via exporters.



 $ kubectl apply -f 4_central_collector.yaml


## Step 6: OpenTelemetry Agent to a Backend

In this scenario, the “Agent mode”; the OTEL instrumented application sends the telemetry data to a (collector) agent that resides together with the application or on the same host as the application, as a Sidecar or DaemonSet.

### Sidecar
Now we will deploy an OpenTelemetry agent using the Sidecar mode. This agent will send traces from the application to our central(gateway) OpenTelemetry collector.

Create a new file and call it sidecar.yaml

Deploy the sidecar with kubectl apply -f sidecar.yaml


## step 7: Auto Instrumentation
The operator can inject and configure OpenTelemetry auto-instrumentation libraries.

Currently DotNet, Java, NodeJS and Python are supported.

To use auto-instrumentation, configure an Instrumentation resource with the configuration for the SDK and instrumentation.

For the Java application, that will look like this.

vim instrumentation.yaml

 $ kubectl apply -f instrumentation.yaml

# Grafana and tempo install through helm

kubectl create namespace my-grafana

kubectl get namespace my-grafana

kubectl apply -f grafana.yaml --namespace=my-grafana

kubectl get pvc --namespace=my-grafana -o wide


kubectl get deployments --namespace=my-grafana -o wide


kubectl get svc --namespace=my-grafana -o wide

kubectl get all --namespace=my-grafana