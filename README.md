# Validated Patterns Guided Tour

Welcome! This guided tour will introduce you to **Validated Patterns**,
demonstrating clearly and interactively what they are and how they're
effectively used. We'll start from scratch, initially deploying an empty
pattern, and then incrementally add complexity and functionality.
As we progress step-by-step, you'll gain practical insights into how
each component fits together, forming what's known as a "living architecture".

By the end of this guide, you'll have the skills and confidence to create,
customize, and deploy your own Validated Patterns in real-world scenarios.

## What are Validated Patterns?

The primary reference for Validated Patterns is always the official website,
[validatedpatterns.io](https://validatedpatterns.io/). According to the site,
Validated Patterns are:

- An evolution in how applications are deployed across hybrid cloud environments.
- Advanced reference architectures designed to simplify the deployment of
  complex business solutions.
- An opinionated GitOps environment that significantly lowers the barrier
  for creating repeatable, declarative deployments.

In practical terms, imagine your team regularly deploying a consistent set of
microservices across various cloud platforms. Without standardization, each
deployment might require custom scripts, extensive manual configuration, and
significant troubleshooting time. Validated Patterns address these issues by
providing standardized, repeatable, and easily manageable deployment templates.

To truly grasp what these descriptions mean—and more importantly, to understand
what a Validated Pattern fundamentally is—we'll walk through the entire process
of creating and deploying a pattern together. By the end of this guide, you'll
have the hands-on experience to confidently leverage Validated Patterns in your
own projects.

### Opinionated GitOps Environment

A Validated Pattern **is** an **opinionated GitOps environment**, meaning it
comes with pre-defined choices designed to simplify deployments, ensure
consistency, and reduce complexity.

The first opinionated requirement you should be aware of is:

> Patterns **must** be deployed to an OpenShift 4 cluster.

If you don’t already have access to an OpenShift 4 cluster, free trials are
readily available at
[redhat.com](https://www.redhat.com/en/technologies/cloud-computing/openshift/try-it).

The second critical requirement relates directly to GitOps itself:

> A pattern is fundamentally a Git repository.

This means that making updates to a pattern is as simple as updating the
contents of its source Git repository—allowing for clear versioning,
traceability, and streamlined deployments.

## Deploying Our First Pattern

With your OpenShift cluster set up, you're ready to deploy your first Validated
Pattern! Before we can begin deploying patterns, however, we'll first install
the Validated Patterns Operator, a key component that simplifies management and
deployment.

### Installing the Validated Patterns Operator

The quickest way to install the operator is through the OperatorHub in your
OpenShift cluster—simply search for the **Validated Patterns Operator** and
click **Install**.

However, manually installing the operator provides valuable insights into how it
works and what components are involved. Since this understanding will greatly
enhance your ability to use and troubleshoot Validated Patterns effectively,
we'll take this more educational approach.

The Validated Patterns Operator and related components are typically installed
via a Helm chart. You can find the Helm chart's source code in the
[pattern-install-chart GitHub repository](https://github.com/validatedpatterns/pattern-install-chart),
and the chart itself can be accessed from the [pattern-install quay.io repository](https://quay.io/repository/hybridcloudpatterns/pattern-install?tab=info).
This chart enables the "one-push deployments" you'll see later in the guided
tour.

For this initial deployment, however, we'll install each component separately.
Doing so will allow us to carefully inspect and understand each element
individually.

#### Install the Pattern CustomResourceDefinition (CRD)

First, we'll install the **Pattern CustomResourceDefinition (CRD)**, which
defines the structure and behavior of our patterns within OpenShift.

Save the following YAML content to a file named `pattern-crd.yaml`:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.14.0
  name: patterns.gitops.hybrid-cloud-patterns.io
spec:
  group: gitops.hybrid-cloud-patterns.io
  names:
    kind: Pattern
    listKind: PatternList
    plural: patterns
    shortNames:
    - patt
    singular: pattern
  scope: Namespaced
  versions:
  - additionalPrinterColumns:
    - jsonPath: .status.lastStep
      name: Step
      priority: 1
      type: string
    - jsonPath: .status.lastError
      name: Error
      priority: 2
      type: string
    name: v1alpha1
    schema:
      openAPIV3Schema:
        description: Pattern is the Schema for the patterns API
        properties:
          apiVersion:
            description: |-
              APIVersion defines the versioned schema of this representation of an object.
              Servers should convert recognized schemas to the latest internal value, and
              may reject unrecognized values.
              More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources
            type: string
          kind:
            description: |-
              Kind is a string value representing the REST resource this object represents.
              Servers may infer this from the endpoint the client submits requests to.
              Cannot be updated.
              In CamelCase.
              More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds
            type: string
          metadata:
            type: object
          spec:
            description: PatternSpec defines the desired state of Pattern
            properties:
              analyticsUUID:
                description: Analytics UUID. Leave empty to autogenerate a random
                  one. Not PII information
                type: string
              clusterGroupName:
                type: string
              experimentalCapabilities:
                description: Comma separated capabilities to enable certain experimental
                  features
                type: string
              extraParameters:
                description: |-
                  .Name is dot separated per the helm --set syntax, such as:
                    global.something.field
                items:
                  properties:
                    name:
                      type: string
                    value:
                      type: string
                  required:
                  - name
                  - value
                  type: object
                type: array
              extraValueFiles:
                description: URLs to additional Helm parameter files
                items:
                  type: string
                type: array
              gitOpsSpec:
                properties:
                  manualSync:
                    description: 'Require manual intervention before Argo will sync
                      new content. Default: False'
                    type: boolean
                type: object
              gitSpec:
                properties:
                  hostname:
                    description: Optional. FQDN of the git server if automatic parsing
                      from TargetRepo is broken
                    type: string
                  inClusterGitServer:
                    default: false
                    description: (EXPERIMENTAL) Enable in-cluster git server (avoids
                      the need of forking the upstream repository)
                    type: boolean
                  originRepo:
                    description: |-
                      Upstream git repo containing the pattern to deploy. Used when in-cluster fork to point to the upstream pattern repository.
                      Takes precedence over TargetRepo
                    type: string
                  originRevision:
                    description: (DEPRECATED) Branch, tag or commit in the upstream
                      git repository. Does not support short-sha's. Default to HEAD
                    type: string
                  pollInterval:
                    default: 180
                    description: 'Interval in seconds to poll for drifts between origin
                      and target repositories. Default: 180 seconds'
                    type: integer
                  targetRepo:
                    description: Git repo containing the pattern to deploy. Must use
                      https/http or, for ssh, git@server:foo/bar.git
                    type: string
                  targetRevision:
                    description: 'Branch, tag, or commit to deploy.  Does not support
                      short-sha''s. Default: HEAD'
                    type: string
                  tokenSecret:
                    description: |-
                      Optional. K8s secret name where the info for connecting to git can be found. The supported secrets are modeled after the
                      private repositories in argo (https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#repositories)
                      currently ssh and username+password are supported
                    type: string
                  tokenSecretNamespace:
                    description: Optional. K8s secret namespace where the token for
                      connecting to git can be found
                    type: string
                type: object
              multiSourceConfig:
                properties:
                  clusterGroupChartGitRevision:
                    default: main
                    description: |-
                      The git reference when deploying the clustergroup helm chart directly from a git repo
                      Defaults to 'main'. (Only used when developing the clustergroup helm chart)
                    type: string
                  clusterGroupChartVersion:
                    description: Which chart version for the clustergroup helm chart.
                      Defaults to "0.8.*"
                    type: string
                  clusterGroupGitRepoUrl:
                    description: |-
                      The url when deploying the clustergroup helm chart directly from a git repo
                      Defaults to '' which means not used (Only used when developing the clustergroup helm chart)
                    type: string
                  enabled:
                    default: true
                    description: (EXPERIMENTAL) Enable multi-source support when deploying
                      the clustergroup argo application
                    type: boolean
                  helmRepoUrl:
                    description: The helm chart url to fetch the helm charts from
                      in order to deploy the pattern. Defaults to https://charts.validatedpatterns.io/
                    type: string
                type: object
            required:
            - clusterGroupName
            - gitSpec
            type: object
          status:
            description: PatternStatus defines the observed state of Pattern
            properties:
              analyticsSent:
                default: 0
                type: integer
              analyticsUUID:
                type: string
              appClusterDomain:
                type: string
              applications:
                items:
                  description: |-
                    PatternApplicationInfo defines the Applications
                    Status for the Pattern.
                    This structure is part of the PatternStatus as an array
                    The Application Status will be included as part of the Observed state of Pattern
                  properties:
                    healthMessage:
                      type: string
                    healthStatus:
                      type: string
                    name:
                      type: string
                    namespace:
                      type: string
                    syncStatus:
                      type: string
                  type: object
                type: array
              clusterDomain:
                type: string
              clusterID:
                type: string
              clusterName:
                type: string
              clusterPlatform:
                type: string
              clusterVersion:
                type: string
              conditions:
                items:
                  properties:
                    lastTransitionTime:
                      description: Last time the condition transitioned from one status
                        to another.
                      format: date-time
                      type: string
                    lastUpdateTime:
                      description: The last time this condition was updated.
                      format: date-time
                      type: string
                    message:
                      description: A human readable message indicating details about
                        the transition.
                      type: string
                    status:
                      description: Status of the condition, one of True, False, Unknown.
                      type: string
                    type:
                      description: Type of deployment condition.
                      type: string
                  required:
                  - lastUpdateTime
                  - status
                  - type
                  type: object
                type: array
              lastError:
                description: Last error encountered by the pattern
                type: string
              lastStep:
                description: Last action related to the pattern
                type: string
              path:
                type: string
              version:
                description: Number of updates to the pattern
                type: integer
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
```

Apply it to your OpenShift cluster using the command:

```sh
oc apply -f pattern-crd.yaml
```

Feel free to review the details of this CRD if you're interested (you can run
`oc describe crd patterns.gitops.hybrid-cloud-patterns.io`), but for practical
purposes, the key fields we'll focus on later are `gitSpec.targetRepo` and
`gitSpec.targetRevision`.

#### Install the Pattern Operator Configmap

Next, we'll configure the Validated Patterns Operator by creating a ConfigMap,
which defines certain environment settings and defaults used by the operator.

Save the provided YAML content to a file named
`pattern-operator-configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: patterns-operator-config
  namespace: openshift-operators
data:
  gitops.catalogSource: redhat-operators
  gitops.channel: gitops-1.15
  gitea.chartName: gitea
  gitea.helmRepoUrl: https://charts.validatedpatterns.io/
  gitea.chartVersion: 0.0.*
```

Apply it to your OpenShift cluster using the command:

```sh
oc apply -f pattern-operator-configmap.yaml
```

You might notice references to `gitea` parameters here; we won't need those
right now since we're starting with a public GitHub repository. The important
part to note is that our GitOps deployment uses
[Red Hat OpenShift GitOps](https://www.redhat.com/en/technologies/cloud-computing/openshift/gitops),
which enforces our earlier-stated requirement: patterns must be Git repositories.

#### Install the Validated Patterns Operator Subscription

Finally, we'll create an Operator Subscription, which is OpenShift’s way of
installing and managing operators, including updates and lifecycle management.

Save the YAML snippet provided to a file named
`pattern-operator-subscription.yaml`:

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: patterns-operator
  namespace: openshift-operators
  labels:
    operators.coreos.com/patterns-operator.openshift-operators: ""
spec:
  channel: fast
  installPlanApproval: Automatic
  name: patterns-operator
  source: community-operators
  sourceNamespace: openshift-marketplace
```

Apply it to your OpenShift cluster using the command:

```sh
oc apply -f pattern-operator-subscription.yaml
```

For those familiar with Kubernetes but newer to OpenShift, the Subscription
resource handles automated installation and updates of operators. You can learn
more about Operator Subscriptions from the
[Operator Lifecycle Manager documentation](https://olm.operatorframework.io/docs/concepts/crds/subscription/).

If you're curious about the internal workings or would like to explore further,
you can always examine the operator’s source code directly in the
[patterns-operator GitHub repository](https://github.com/validatedpatterns/patterns-operator).
We'll reference this repository again as needed throughout this guide.

### Deploying Our First Pattern

Now that we've successfully installed the Patterns CRD, the Patterns Operator
ConfigMap, and the Patterns Operator itself, we're finally ready to deploy our
very first pattern!

Create a file named `pattern.yaml` containing the YAML manifest below:

```yaml
apiVersion: gitops.hybrid-cloud-patterns.io/v1alpha1
kind: Pattern
metadata:
  name: validated-patterns-guided-tour
  namespace: openshift-operators
spec:
  clusterGroupName: hub
  gitSpec:
    targetRepo: https://github.com/dminnear-rh/validated-patterns-guided-tour.git
    targetRevision: main
```

Apply this pattern manifest to your OpenShift cluster by running:

```sh
oc apply -f pattern.yaml
```

Interestingly, the pattern we're deploying right now is this very
repository—which currently contains nothing more than this README!
(Don't worry, there's no magic involved here—this simply demonstrates
that our repository, at this point, represents an "empty" pattern.)

### What Just Happened?

When you applied the pattern, the Validated Patterns Operator automatically
installed the **Red Hat OpenShift GitOps Operator**, initiating the syncing
process with your Git repository (which, again, is empty for now).

To see the effects, go to your OpenShift console and click the **"Nine Dots"**
menu button in the top-right corner. You’ll notice two new ArgoCD instances:

| ArgoCD Instance | Purpose |
| --- | --- |
| **Cluster ArgoCD** | Automatically created by the Validated Patterns Operator.It manages the setup and lifecycle of the pattern-specific ArgoCD instance (the actual GitOps controller). |
| **Example ArgoCD** | This is your actual GitOps instance. It handles syncing your pattern repository with your OpenShift cluster, deploying resources as defined in the Git repository. |

Currently, your `Example ArgoCD` instance is empty because we haven't yet
defined any resources to deploy. Remember: the repository itself is the complete
specification of your pattern. In the upcoming sections, we will populate this
repository with real, deployable resources, and you'll see the true power of
Validated Patterns!

## What's Next?

You've successfully deployed an empty Validated Pattern and learned about its
core components, such as the Patterns Operator and ArgoCD instances. Now, let's
start populating this pattern with real, deployable resources.

In [Chapter 1 - Helm Charts in Patterns](https://github.com/dminnear-rh/validated-patterns-guided-tour/tree/chapter-1),
you'll deploy your first application using Helm charts. This hands-on chapter
will guide you through both local and repository-based Helm chart deployments,
demonstrating how Validated Patterns simplify and streamline your GitOps
workflow in practical scenarios.
