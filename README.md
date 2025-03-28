# Chapter 3 - Subscriptions in Validated Patterns

In the previous chapters, we've taken a look at deploying Helm charts as
Argo CD applications. These applications are the bread and the butter of
Validated Patterns comes in the form of Operator subscriptions. These
subscriptions are what really make up the magic as you'll see with the simple
pattern demonstrated in this chapter.

## A Brief Introduction to our Pattern

Before we dive into subscriptions, let's discuss what this pattern is composed of.
This is our first real pattern, one that's more than just a deploy of a Helm
chart and the first we've created as part of this tutorial that accomplishes the
aim of being a "living architecture". The major components of our pattern are:

- **OpenShift Serverless** for scalable serverless deployments.
- **OpenShift Service Mesh 3** for traffic management without sidecars.
- **Kiali** for observability and service visualization.
- **Tempo** and **OpenTelemetry** for distributed tracing.

The overall purpose of this pattern is to demonstrate a holistic managed solution
for serverless architectures.

## Adding Subscriptions to our Pattern

If you take a look at [values-hub.yaml](./values-hub.yaml) at the `subscriptions`
block you can see how simple it is to create an Operator subscription. For Red Hat
operators like we are using, all you actually need to create a subscription is
the correct name. (You can always install the operator manually through Operator
Hub and check the name of the subscription that was created to be sure what
this name is.)

If you only provide the name, as we did for the subscription `servicemeshoperator3`
then some default values are assumed. The subscripion we created with just the name
is equivalent to:

```yaml
servicemesh:
  name: servicemeshoperator3
  namespace: openshift-operators
  source: redhat-operators
  sourceNamespace: openshift-marketplace
  installPlanApproval: {{ $.Values.global.options.installPlanApproval }} # defaults to Automatic if you don't set it
```

In addition, you may optionally set the `channel` field and the `csv` field to
specify exactly which release channel and version of the operator you intend to
install. In the case of the OpenShift Serverless Operator, you may have noticed
we explicitly chose a namespace other than the default of `openshift-operators`.
In this case, we are choosing to use the namespace recommended by the operator.
This is the recommended best practice when possible and another reason it's useful
to install operators from Operator Hub as you develop patterns-to reference the
recommendations of the creator of the operator.

## The Applications Included in this Pattern

While this chapter is meant to focus on subscriptions, we are deploying
a complete pattern to demonstrate the power included with our simple
subscription declaration. To that end, let's take a quick glance over
the applications we're deploying alongside the Operators of our pattern:

* [The `serverless` chart](./charts/serverless-demo/) consists of the components:
  * [knative-service.yaml](./charts/serverless-demo/templates/knative-service.yaml): a simple Knative Serving service instrumented for tracing
  * [opentelemetry-collector.yaml](./charts/serverless-demo/templates/opentelemetry-collector.yaml): a basic OpenTelemetry collector with Tempo backend for tracing
  * [traffic-generator.yaml](./charts/serverless-demo/templates/traffic-generator.yaml): a simple traffic-generator job to send requests automatically to our knatic service
* [The `servicemesh` chart](./charts/servicemesh/) consists of the components:
  * [servicemeshcontrolplane.yaml](./charts/servicemesh/templates/servicemeshcontrolplane.yaml): the configuration for the Istio control plane pods
  * [servicemeshcni.yaml](./charts/servicemesh/templates/servicemeshcni.yaml): Istio’s Container Network Interface (CNI) plugin
