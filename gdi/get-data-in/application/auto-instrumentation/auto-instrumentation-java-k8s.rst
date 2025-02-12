.. include:: /_includes/gdi/zero-config-preview-header.rst

.. meta::
   :description: How to enable zero configuration automatic instrumentation for Kubernetes Java applications and thus collect and send traces to Splunk Application Performance Monitoring (APM) without altering your code.

.. _auto-instrumentation-java-k8s:

==================================================================================================================
Zero Configuration Auto Instrumentation for Java Applications on Kubernetes
==================================================================================================================

Zero Configuration Auto Instrumentation for Java enables automatic instrumentation for Kubernetes Java applications. When you enable automatic instrumentation, you do not have to recompile your applications, but you do have to restart any applications that are already running. 

.. _zero-config-k8s-prereqs:

Prerequisites
====================================

.. include:: /_includes/gdi/zero-conf-reqs.rst

- Install :ref:`the Splunk OpenTelemetry (OTel) Collector Kubernetes Operator<k8s-operator>` on a :new-page:`compatible version of Kubernetes <https://github.com/signalfx/splunk-otel-collector-operator#compatibility-matrix>`.

.. _enable-zero-conf-java-k8s:

Enable automatic instrumentation of Java applications on Kubernetes 
===============================================================================

Before deployment, you can enable automatic instrumentation for a Kubernetes Deployment or pod by adding the ``otel.splunk.com/inject-java`` annotation.

When you enable instrumentation, the Collector operator injects the Splunk OTel Java agent into Java applications to capture telemetry data.

To enable automatic instrumentation, add this annotation to the ``spec`` for a deployment or pod: ``otel.splunk.com/inject-java: "true"``.
If you add the annotation to a pod, restarting the pod removes the annotation.

You can also enable automatic instrumentation on a running workload.

.. _enable-zero-conf-java-yaml:

Enable or disable automatic instrumentation before runtime
-------------------------------------------------------------

If the deployment is not deployed, add the ``otel.splunk.com/inject-java`` annotation to the application deployment YAML file.

For example, given the following deployment YAML:

.. code-block:: yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-java-app
    spec:
      template:
        spec:
          containers:
          - name: my-java-app
            image: my-java-app:latest


Enable auto instrumentation by adding ``otel.splunk.com/inject-java: "true"`` to the ``spec``:

.. code-block:: yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-java-app
    spec:
      template:
        metadata:
          annotations:
            otel.splunk.com/inject-java: "true"
        spec:
          containers:
          - name: my-java-app
            image: my-java-app:latest
    

The Collector operator enables automatic instrumentation for any Java applications in the deployment.

To disable automatic instrumentation, remove the annotation or set its value to ``false``.

.. _enable-zero-conf-java-patch:

Enable or disable automatic instrumentation on a running workload
-----------------------------------------------------------------

If the application is already running, patch the deployment using ``kubectl patch`` to enable instrumentation. 

.. caution:: 

    Patching a deployment restarts the pods in the deployment.


Use the following snippet as an example. Replace ``<my-deployment>`` with your deployment's name.

.. code-block:: bash
    
    kubectl patch deployment <my-deployment> -p '{"spec": {"template":{"metadata":{"annotations":{"otel.splunk.com/inject-java":"true"}}}} }'

To disable automatic instrumentation, run the same command but change the value of the annotation to ``false``:

.. code-block:: bash
    
    kubectl patch deployment <my-deployment> -p '{"spec": {"template":{"metadata":{"annotations":{"otel.splunk.com/inject-java":"false"}}}} }'


.. _k8s-zero-conf-java-verify:

Check the status of automatic instrumentation 
-------------------------------------------------

When you successfully enable instrumentation for a deployment, the metadata for every pod in the deployment includes the annotation ``otel.splunk.com/injection-status:success``. 

Use the following command to check for the ``injection-status`` annotation. Replace ``<POD_NAME>`` with the name of your pod.

.. code-block:: bash

    kubectl get pod  <POD_NAME> -o yaml | grep inject

The command's result is similar to the following:

.. code-block:: bash

    otel.splunk.com/inject-java: "true"
    otel.splunk.com/injection-status: success


If the ``injection-status`` annotation is not present or is not set to ``success``, auto instrumentation is not enabled. See the troubleshooting section for next steps.

If the ``injection-status`` annotation is set to ``success``, you have enabled instrumentation correctly. You can :ref:`verify-apm-data` or :ref:`optionally configure instrumentation settings<configure-java-zeroconf-k8s>`. 

.. _configure-java-zeroconf-k8s:

Optionally configure instrumentation
-----------------------------------------

The default settings for auto instrumentation are sufficient for most basic cases. You can add advanced configuration like enabling custom sampling and including custom data in the reported spans with environment variables and Java system properties.

For example, if you want every span to include the key-value pair ``build.id=feb2023_v2``, set the ``OTEL_RESOURCE_ATTRIBUTES`` environment variable.

  .. code-block:: bash
    
     kubectl set env deployment/<my-deployment> OTEL_RESOURCE_ATTRIBUTES=build.id=feb2023_v2

See :ref:`advanced-java-otel-configuration` for the full list of supported environment variables.

.. include:: /_includes/gdi/next-steps.rst

.. _k8s-zero-conf-troubleshooting:

Troubleshooting
=======================

If you enable auto instrumentation and you do not see any telemetry data in Observability Cloud APM, try the following steps:

- Check the Collector operator logs. Look for the pods in the ``splunk-otel-operator-system`` namespace, and then examine their logs:

.. code-block:: bash

   kubectl get pods  --namespace=splunk-otel-operator-system

   NAME                                                      READY   STATUS    RESTARTS   AGE
   splunk-otel-agent-7cspj                                   1/1     Running   0          31h
   splunk-otel-agent-gkmts                                   1/1     Running   0          31h
   splunk-otel-agent-xbnpm                                   1/1     Running   0          31h
   splunk-otel-cluster-receiver-8cd9874c8-6jlz6              1/1     Running   0          31h
   splunk-otel-operator-controller-manager-8455c8bc7-m8f24   1/1     Running   0          31h

   kubectl logs  --namespace=splunk-otel-operator-system  splunk-otel-operator-controller-manager-8455c8bc7-m8f24 

Run this command to see the logs for one of the pods:

.. code-block:: bash

   kubectl logs  --namespace=splunk-otel-operator-system  <pod-name>

- You can also follow the :ref:`steps to troubleshoot the Java agent<basic-java-troubleshooting>`.

.. include:: /_includes/troubleshooting-steps.rst