[![Build Status](https://travis-ci.com/morsby/openfaas-templates.svg?branch=main)](https://travis-ci.com/morsby/openfaas-templates)

# OpenFaaS Templates

A collection of templates for use in OpenFaaS:

- `golang-http`: Originally developed by Martin Heinz at
  [this repo](https://github.com/MartinHeinz/openfaas-templates). Modified to use Go 1.16.
- `golang-middleware`: Originally developed
  [here](https://github.com/openfaas/golang-http-template). Modified to use Go 1.16.

This repository is used both for development of _OpenFaaS_ templates and well as a
_OpenFaaS Template Store_.

## Usage

Before deploying functions, the Docker image needs to first be push to remote registry
repository, this repository is specified using `PREFIX` variable in `Taskfile.yml`. You
need to change its value to your _Docker Hub_ username.

- **Verify**:

  You can verify that all templates work by running:

  ```bash
  task verify
  ```

  This will build all templates in `template` directory. Ideally test should be included
  in Docker build, so that test don't need to be ran separately.

- **Build**:

  You can build the Docker image of the template using:

  ```bash
  task build FUNC=golang-mod
  ```

- **Run**:

  You can run template by building and running function created from it:

  ```bash
  task run FUNC=golang-mod
  ```

  The function is ran in Docker container on local accessible on port _8081_.

- **Debug**:

  You can debug template by building and running function created from it and attaching to
  its `watchdog` process logs:

  ```bash
  task debug FUNC=golang-mod
  ```

  When the function is running you can hit it with _cURL_:

  ```bash
  curl -vvv --header "Content-Type: application/json" \
            --request POST \
            --data '{"key":"value"}' \
            127.0.0.1:8081
  ```

  You might also want to change timeouts of the function when debugging (`template.yml`):

  ```yaml
  functions:
      func_name:
        ...
        environment:
            read_timeout: 20
            write_timeout: 20
  ```

- **Clean**:

  To remove artifacts created during build you can run:

  ```bash
  task clean
  ```

## Using Local Docker Images

Images need to be pushed to remote registry, because _OpenFaaS_ doesn't recognize local
repositories. That's why, you need to specify prefix, which is a username + repository in
remote registry.

If you don't want to push images, then you can use `helm` and pass it
`openfaasImagePullPolicy` and `faasnetesd.imagePullPolicy` parameters to use local docker
images:

```shell
helm upgrade openfaas chart/openfaas --install \
  --set "faasnetesd.imagePullPolicy=IfNotPresent" \
  --set "openfaasImagePullPolicy=IfNotPresent" \
  --namespace openfaas  \
  --set functionNamespace=openfaas-fn \
  --set operator.create=true
```

## Create New Template

How you create new templates depends on language you use, therefore please refer to guide
at <https://github.com/openfaas/faas-cli/blob/master/guide/TEMPLATE.md>

For the _Golang_ based templates please use _Golang_ module system.

You can use `./template/golang-mod` as a base or example.

Before building image, you first need to download all dependencies with `go mod tidy`.

## Troubleshooting Functions

Apart from `task debug` you can also use following commands for troubleshooting:

- View functions and their logs:

  ```console
  $ kubectl get deploy -n openfaas-fn
  NAME             READY     UP-TO-DATE   AVAILABLE    AGE
  <FUNCTION_NAME>   0/1       1            0           11m
  nodeinfo          1/1       1            1           7h3m

  kubectl logs -n openfaas-fn deploy/<FUNCTION_ANME>
  ```

- See if function failed to start:

  ```shell
  kubectl describe -n openfaas-fn deploy/<FUNCTION_NAME>
  ```

### Resources

- <https://github.com/openfaas/templates/blob/master/template/dockerfile/function/Dockerfile>
- <https://rancher.com/docs/k3s/latest/en/configuration/>
- <https://blog.alexellis.io/test-drive-k3s-on-raspberry-pi/>
- <https://github.com/openfaas-incubator/ofc-bootstrap>
- <https://github.com/openfaas/faas-cli/blob/master/guide/TEMPLATE.md>
- <https://docs.openfaas.com/deployment/troubleshooting/#function-execution-logs>
- <https://blog.alexellis.io/serverless-golang-with-openfaas/>
