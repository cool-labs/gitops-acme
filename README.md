# Create CD workflow on RH ACM using Tekton

Procedure:

```console
> oc new-project acme-cicd

Now using project "acme-cicd" on server "https://api.acmhub.rhbr-labs.com:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app ruby~https://github.com/sclorg/ruby-ex.git

to build a new example application in Python. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=gcr.io/hello-minikube-zero-install/hello-node

>  oc get serviceaccount pipeline
NAME       SECRETS   AGE
pipeline   2         29s

> cd tekton

> oc create -f tasks/01_create_namespaces.yaml
task.tekton.dev/create-namespaces created

> oc create -f tasks/02_create_app.yaml
task.tekton.dev/acme-deployment created

> oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:acme-cicd:pipeline
clusterrole.rbac.authorization.k8s.io/cluster-admin added: "system:serviceaccount:acme-cicd:pipeline"

> oc apply -f pipeline-acm.yaml
pipeline.tekton.dev/deploy-using-acm created

> oc apply -f prepare/tekton-source-pvc.yaml
persistentvolumeclaim/source-pvc created

> oc apply -f pipelinerun.yaml
pipelinerun.tekton.dev/deploy-using-acm-run-1 created

> tkn pipelinerun logs deploy-using-acm-run-1 -f
[fetch-repository : clone] + CHECKOUT_DIR=/workspace/output/
[fetch-repository : clone] + '[[' true '==' true ]]
[fetch-repository : clone] + cleandir
[fetch-repository : clone] + '[[' -d /workspace/output/ ]]
[fetch-repository : clone] + rm -rf /workspace/output//onecluster-multiple-envs
[fetch-repository : clone] + rm -rf /workspace/output//.git
[fetch-repository : clone] + rm -rf '/workspace/output//..?*'
[fetch-repository : clone] + test -z 
[fetch-repository : clone] + test -z 
[fetch-repository : clone] + test -z 
[fetch-repository : clone] + /ko-app/git-init -url https://github.com/giofontana/rhacm-pipelines.git -revision master -refspec  -path /workspace/output/ '-sslVerify=true' '-submodules=true' -depth 1
[fetch-repository : clone] {"level":"info","ts":1597794549.2025642,"caller":"git/git.go:136","msg":"Successfully cloned https://github.com/giofontana/rhacm-pipelines.git @ f111d89c17e5647d3cd4dedca6968f20b73cfb0d (grafted, HEAD, origin/master) in path /workspace/output/"}
[fetch-repository : clone] {"level":"info","ts":1597794549.240191,"caller":"git/git.go:177","msg":"Successfully initialized and updated submodules in path /workspace/output/"}
[fetch-repository : clone] + cd /workspace/output/
[fetch-repository : clone] + git rev-parse HEAD
[fetch-repository : clone] + tr -d '\n'
[fetch-repository : clone] + RESULT_SHA=f111d89c17e5647d3cd4dedca6968f20b73cfb0d
[fetch-repository : clone] + EXIT_CODE=0
[fetch-repository : clone] + '[' 0 '!=' 0 ]
[fetch-repository : clone] + echo -n f111d89c17e5647d3cd4dedca6968f20b73cfb0d

[create-namespaces : creating-namespaces] **** Creating namespaces ****
[create-namespaces : creating-namespaces] namespace/acme-demo created

[acme-deployment : acme-deployment] **** DEV ENVIRONMENT: Creating application on RH ACM ****
[acme-deployment : acme-deployment] application.app.k8s.io/acm-demo-acme-insurance created
[acme-deployment : acme-deployment] **** DEV ENVIRONMENT: Creating channel on RH ACM ****
[acme-deployment : acme-deployment] channel.apps.open-cluster-management.io/acm-demo-channel-dockerhub created
[acme-deployment : acme-deployment] **** DEV ENVIRONMENT: Creating placementrule on RH ACM ****
[acme-deployment : acme-deployment] placementrule.apps.open-cluster-management.io/acm-demo-placement-ui created
[acme-deployment : acme-deployment] placementrule.apps.open-cluster-management.io/acm-demo-placement-dev created
[acme-deployment : acme-deployment] **** DEV ENVIRONMENT: Creating subscription on RH ACM ****
[acme-deployment : acme-deployment] subscription.apps.open-cluster-management.io/acm-demo-subscription-ui created
[acme-deployment : acme-deployment] subscription.apps.open-cluster-management.io/acm-demo-subscription-middleware created
[acme-deployment : acme-deployment] subscription.apps.open-cluster-management.io/acm-demo-subscription-database-connector created

```


# Adding an Event Listener to sync pipeline with Git repository

## NOT WORKING :( Getting params.git-url command not found

1. Manually creating Event Listener resources (event listener, trigger template & trigger binding).

```console

> cd tekton/event-listener

> oc project acme-cicd
Now using project "acme-cicd" on server "https://api.cluster-509e.509e.sandbox1047.opentlc.com:6443".

> oc create -f trigger-binding.yaml
triggerbinding.triggers.tekton.dev/acme-cicd-acm-trigger-binding created

> oc create -f trigger-template.yaml
triggertemplate.triggers.tekton.dev/acme-cicd-acm-trigger-template created

> oc create -f event-listener.yaml
eventlistener.triggers.tekton.dev/acme-cicd-acm created

```

2. Then creating a route for event-listener service. You can do it from command line or easily from OpenShift dashboard. Once you get a route for the event listener (https://el-acme-cicd-acme-cicd.apps.cluster-509e.509e.sandbox1047.opentlc.com/) copy it.

3. Go to your git repository > settings > webhooks. Add a new webhook for push command with event listener route.

4. Push any change to the repo to test if it works.


Webhook working?
