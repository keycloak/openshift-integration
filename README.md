# Openshift Integration

## Overview

This demo starts an OpenShift cluster configured to authenticate with Keycloak. It also starts Keycloak on the OpenShift cluster.

## Config

There is some configuration of the demo available in the `config` file.

## OCP 3.10 client

Recently, a new version of `oc` client has been released - `3.10.0.rc0`. The `openshift-start-configured-cluster` downloads
it automatically and stores in local `./oc` path.

NOTE: You still may use a different version, just put the `oc` binary into this directory.

## Start the demo

To start the demo simply run:

    ./openshift-start-configured-cluster

This will use oc cluster to write configuration to a temporary directory. It will then configure webhookTokenAuthenticators for kube-apiserver, openshift-apiserver and openshift-controller-manager. Finally it will start the OpenShift cluster that should now be secured with Keycloak.

## Try it out

Before trying it out make sure Keycloak is fully running. You can check this by running `token`. If it returns a bearer token it's up and running.

To try things out run (this will run `oc get sa` with a token obtained from Keycloak using the simple utility `token`):

    ./openshift-api-try

You should see the following:

    Error from server (Forbidden): serviceaccounts is forbidden: User "admin" cannot list serviceaccounts in the namespace "myproject": User "admin" cannot list serviceaccounts in project "myproject"

Run the following to give access to the user:

    ./oc adm policy add-cluster-role-to-user system:master <username>

If you get an `Unauthorized` message instead something is wrong. To debug what's going on run:

    ./openshift-api-logs

This will show the logs from the OpenShift API, there should be some information here to help debug the problem.

To allow the `admin` user from Keycloak to run `oc get sa` run the following:

    ./oc --token=$(kcinit token) get sa

You should now be able to run `./openshift-api-try` and get a list of service accounts from OpenShift.

## Notes

* The demo is currently not working with secure routes. Currently if using secured routes for Keycloak the OpenShift API complains about the certificate not being valid. Keycloak is using edge termination and the certificate should be signed by the default OpenShift CA, but this is still not valid.

* The demo is not using a released version of Keycloak, but rather an image on Docker Hub built from the `openshift-integration` branch. This image will be updated regularly until the required features are included in a Keycloak release.

* Improve how the token is retrieved to invoke `oc`. Currently tokens are retrieved with a small utility, this should most likely be replaced with `kcinit`.

* Can't delete temporary OpenShift configuration directory. This results in some temporary files in `tmp` being left after doing `oc cluster down`. These should be cleared by the OS when restarted though.

* Replace OSIN fully. This would allow `oc login` without using the token util and also allow using Keycloak to login to the OpenShift web console.
