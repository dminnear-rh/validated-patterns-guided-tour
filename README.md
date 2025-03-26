# Chapter 1 - Helm Charts in Patterns

Congratulations! You've already deployed your first pattern—even though it's
currently an empty one. In this chapter, we'll build upon this foundation by
deploying our first real application using Helm charts.

This guide assumes you're already familiar with Helm basics. If you're new to
Helm or would like a quick refresher, check out the official
[Chart Template Guide](https://helm.sh/docs/chart_template_guide/getting_started/)
to get comfortable with the concepts we'll use in this chapter.

## Deploying Local Helm Charts

We'll start by deploying a very simple Helm chart stored locally within our
pattern repository. The chart we'll use for this example is called
[simple-chart](./simple-chart/). This chart has just one configurable value
(the container image to deploy) and creates a simple deployment with a single
replica in your OpenShift cluster.

Take a moment to examine the [values-hub.yaml](./values-hub.yaml) file in your
repository. Notice the resources labeled `pattern-git-repo`. These resources
include a namespace, project, and ArgoCD application that instruct ArgoCD how to
deploy the chart. Because our chart resides within our pattern Git repository,
the path provided to the ArgoCD application is relative to the root directory of
the repository itself.

## Deploying Helm Charts from Repositories

Helm charts don't always have to reside in your patterns's Git repository;
you can also deploy charts directly from Helm repositories. For example, in
the same [values-hub.yaml](./values-hub.yaml) file, you'll see an example of
deploying the popular Nginx chart directly from Bitnami's Helm repository. The
corresponding resources are named `bitnami-helm-repo`, clearly separating them
from our locally stored chart deployment.

## Deploying Helm Charts from External Git Repositories

You've learned how to deploy Helm charts directly from Helm repositories and
from the same Git repository as your pattern. But Validated Patterns also
support deploying Helm charts from external Git repositories, providing
additional flexibility.

Take a look at the resources named `external-git-repo` in
[values-hub.yaml](./values-hub.yaml). You'll notice that in addition to the
standard fields (such as namespace, project, and ArgoCD application), we specify:

- **`repoURL`**: The URL pointing directly to the external Git repository
- containing the Helm chart.
- **`chartVersion`**: Currently represents the branch name of the Git
  repository containing the chart. (Note: This naming can be slightly confusing,
  and we're planning to improve the clarity of this field in future releases of
  the clustergroup chart.)

Using external Git repositories for Helm charts is especially useful when
collaborating across multiple teams or when leveraging shared, reusable
infrastructure maintained externally.

## Updating and Redeploying Our Pattern

Now that you've seen the changes, let's redeploy our updated pattern. Run the
following command:

```sh
oc edit patt validated-patterns-guided-tour -n openshift-operators
```

In the editor, update the `spec.gitSpec.targetRevision` value to `chapter-1`.
Save and close the editor to trigger an update.

After a few minutes, you'll find a new link in the "nine dots" menu labeled
`Hub ArgoCD`. Click this link to view and monitor the deployment of your
applications. Note that the initial synchronization may take a couple of minutes.

The original `Example ArgoCD` instance won't be deleted automatically. You can
clean it up by running:

```sh
oc delete argocd -n validated-patterns-guided-tour-example example-gitops
oc delete ns validated-patterns-guided-tour-example
```

> **Note:** We're currently investigating the clean removal of old Argo
> instances from the "nine dots" menu.

## Understanding the `values-hub.yaml` File

Since this is your first application deployment through a pattern, let's briefly
discuss the role of the `values-hub.yaml` file.

When you initially created your pattern, you defined a `clusterGroupName`—in our
case, set as `hub`. The Validated Patterns Operator automatically looks for a
corresponding values file (`values-hub.yaml`) based on the `clusterGroupName`
you provided. This file determines what should be deployed to the matching
cluster group.

If, for instance, you had created your pattern with a `clusterGroupName` of
`mycluster`, the operator would instead look for a file named
`values-mycluster.yaml` to determine what to deploy.

The operator isn't performing magic behind the scenes. Instead, it leverages the
[clustergroup-chart](https://github.com/validatedpatterns/clustergroup-chart/tree/main)
and uses your values file (`values-hub.yaml`) to create the necessary resources
like ArgoCD instances, namespaces, projects, and applications. Later, we'll
explore more advanced usage, including deploying subscriptions and other complex
resources.

## Coming Up Next

In the next chapter, we'll explore how to configure and set custom values for
our Helm charts, further personalizing your deployments. You can find Chapter 2
[here](https://github.com/dminnear-rh/validated-patterns-guided-tour/tree/chapter-2).
