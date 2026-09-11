# Tuning pod admission selector

The optional `match-tuning-pods-only` setting restricts the Katib pod webhook to pods labelled `workload.viboo.cloud/katib-trial: "true"` in namespaces already enabled for Katib injection.
The setting defaults to false so an existing deployment can migrate custom trial templates before enabling it.
The built-in Job and PyTorchJob templates supply the label on every pod template automatically.

For a custom Job trial, add the marker under `spec.template.metadata.labels`:

```yaml
spec:
  template:
    metadata:
      labels:
        workload.viboo.cloud/katib-trial: "true"
```

For distributed training resources, add the label to each replica's pod template, including the primary replica and workers.
Apply the same requirement to SDK-generated trial specifications and push-based metrics collectors.
Labelling only the Experiment, Trial, or parent Job does not label its pods automatically.
Katib's own injected trial labels are too late for admission selection.

Before enabling the setting, inventory and migrate every supported custom and SDK submission path and run a tuning smoke test.
Leave the setting disabled while any required path lacks the marker.
An unlabelled custom trial will bypass the pod webhook when this setting is enabled and can therefore lose required metrics mutation.

The selector affects only `mutator.pod.katib.kubeflow.org`.
Required mutations retain `failurePolicy: Fail`; Experiment mutation and validation, the namespace selector, and the controller service-account exclusion remain in effect.
Disable the setting to restore the prior namespace-based matching without removing workload labels.

Run the charm's `tox -e lint,unit` checks before packing it with `charmcraft pack`.
