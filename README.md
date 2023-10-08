# Crossplane Function: Loop

A [Crossplane] Composition Function template, for Go.

## What is this?

This is a Crossplane Composition Function.

Composition Functions let you extend Crossplane with new ways to 'do
Composition' - i.e. new ways to produce composed resources given a claim or XR.
You use Composition Functions instead of the `resources` array of templates.

This function generates resources through a loop.
This functions works with Crossplane v1.14 or later.

Here's an example of a Composition that uses this Function.

```yaml
---
apiVersion: nopexample.org/v1alpha1
kind: XNopResource
metadata:
  name: test-resource
spec:
  namespaces:
    - dev
    - production
---
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: xnopresources.nop.example.org
spec:
  compositeTypeRef:
    apiVersion: nop.example.org/v1alpha1
    kind: XNopResource
  mode: Pipeline
  pipeline:
  - step: loop
    functionRef:
      name: loop
    input:
      valuesXrPath: "spec.namespaces"
      namePrefix: "ns-"
      paths:
        - "spec.forProvider.manifest.metadata.name"
      resources:
        - base:
            apiVersion: kubernetes.crossplane.io/v1alpha1
            kind: Object
            spec:
              forProvider:
                manifest:
                  apiVersion: v1
                  kind: Namespace
```

The `XNopResource` is a Composition that defines the `namespaces` field as an array.
The function used in the Composition will iterate through those values and create a resource for each.

The `valuesXrPath` in the Composition points to the path in the Composition's XR where the field with array of values.

The `namePrefix` is an optional value that is used to (optionally) add the prefix to the name of the generated resources.

The `paths` field is an array used to replace the values of the paths in the generated resources.

The output of the resources generated through those examples is as follows.

```yaml
apiVersion: kubernetes.crossplane.io/v1alpha1
kind: Object
metadata:
  ...
spec:
  forProvider:
    manifest:
      apiVersion: v1
      kind: Namespace
      metadata:
        name: dev
---
apiVersion: kubernetes.crossplane.io/v1alpha1
kind: Object
metadata:
  ...
spec:
  forProvider:
    manifest:
      apiVersion: v1
      kind: Namespace
      metadata:
        name: production
```

## How To Build This?

```bash
go generate ./...

export TAG=v0.0.1

docker image build --tag c8n.io/vfarcic/crossplane-function-loop:$TAG .

docker image push c8n.io/vfarcic/crossplane-function-loop:$TAG

yq --inplace ".spec.package = \"c8n.io/vfarcic/crossplane-function-loop:$TAG\"" examples/functions.yaml
```

## How To Test This?

You can try your function out locally using [`xrender`][https://github.com/crossplane-contrib/xrender]. With `xrender`
you can run a Function pipeline on your laptop.

Run the Function locally:

```shell
go run . --insecure --debug
```

Once your Function is running, in another window you can use `xrender`.

```bash
go install github.com/crossplane-contrib/xrender@latest

xrender examples/xr.yaml examples/composition.yaml examples/functions-dev.yaml
```
