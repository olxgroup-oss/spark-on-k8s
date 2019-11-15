# Spark on K8s

## Objective

A practical example on how to run Spark on kubernetes

Reference: <https://spark.apache.org/docs/latest/running-on-kubernetes.html>

## Pre-requisites

- [docker](https://docs.docker.com/install/)
- [direnv](https://direnv.net/docs/installation.html)
- [make](https://www.gnu.org/software/make/)
- [curl](https://curl.haxx.se/)
- [tar](https://www.gnu.org/software/tar/)

A hypervisor for running minikube. Check possibilities [here](https://minikube.sigs.k8s.io/docs/reference/drivers/). The recommended one is [VirtualBox](https://www.virtualbox.org/wiki/Downloads).

## Instructions

```bash
# this will install k8s tooling locally, start minikube, initialize helm and deploy a docker registry chart to your minikube
make

# if everything goes well, you should see a message like this: Registry successfully deployed in minikube. Make sure you add 192.168.99.105:30000 to your insecure registries before continuing. Check https://docs.docker.com/registry/insecure/ for more information on how to do it in your platform.

# build the spark images
make docker-build

# push the spark images our private docker registry
make docker-push
# HINT: if you see "Get https://192.168.99.105:30000/v2/: http: server gave HTTP response to HTTPS client" go back and check whether you have it listed in your insecure registries

# once your images are pushed, let's run a sample spark job (first on client mode)
$SPARK_HOME/bin/spark-submit \
    --master k8s://https://$(minikube ip):8443 \
    --deploy-mode client \
    --conf spark.kubernetes.container.image=$(./get_image_name.sh spark) \
    --class org.apache.spark.examples.SparkPi \
    $SPARK_HOME/examples/jars/spark-examples_2.11-2.4.4.jar

# ... and now, the same job but from within a pod in cluster mode
./generate_clustermode_podspec.sh
./bin/kubectl apply -f clustermode-podspec-with-rbac.yaml # make sure you check the contents of this file to understand better how it works

# in case you want to rerun the example above, make sure you delete the pod first
./bin/kubectl delete pod spark-submit-example

# check the executor pods in another terminal window while running
./bin/kubectl get pods -w

# ...

# deletes minikube and clean up downloaded tools
make clean
```
