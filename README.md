# Deployment configurations for grafana and graphite

This repository contains OpenShift templates for grafana and graphite. Templates are a set of OpenShift/K8S objects which can be deployed together and configured through parameters.

## Basic usage

To deploy the template, by processing the template and piping the output to `oc apply`:

```
oc process -f <template_file> | oc create -f -
```

Parameters can be added either through command line arguments or a parameter file:

```
oc process -f <template_file> -p PARAM_NAME=value | oc create -f -

oc process -f <template_file> --param-file <paramfile> | oc create -f -
```

The format of the paramfile is as follows:

```
PARAM_NAME=value
ANOTHER_PARAM_NAME=another_value
```

The possible parameters can be listed with:

```
oc process --parameters -f <template_file>
```

More information on templates [Documentation](https://docs.openshift.com/container-platform/4.4/openshift_images/using-templates.html)
