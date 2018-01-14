---
layout: post
title: kops Introduction
excerpt: "A hello world of kops, you can learn how to use kops and some kubernetes demo"
---
# Contents
{:.no_toc}

* Toc
{:toc}

# What is kops?

[kops](https://github.com/kubernetes/kops) can help create, destroy, upgrade and maintain production-grade, highly available, Kubernetes clusters from the command line.
it support following cloud platform:

* AWS
* GCE
* VMware vSphere

There is a similar tool called *kubeadm*

We will introcude it based on AWS

# How to install kops?

kops depend on *kubectl*, you should install *kubectl* first. 

* brew

  ~~~ bash
  $ brew install kubectl # if you have not installed
  $ sudo brew install kops
  ~~~

# How to deploy kubernetes cluster on AWS using kops?

* First you need a IAM user to do the deployment, it requires following permissions
  * AmazonEC2FullAccess
  * AmazonRoute53FullAccess
  * AmazonS3FullAccess
  * IAMFullAccess
  * AmazonVPCFullAccess

* Then you need a S3 bucket to store the cluster configuration
* And you need configure your DNS properly, there are four scenarios
  * A Domain purchased/hosted via AWS
  * A subdomain under a domain purchased/hosted via AWS
  * Setting up Route53 for a domain purchased with another registrar
  * Subdomain for clusters in route53, leaving the domain at another registrar

  you can following this [doc](https://github.com/kubernetes/kops/blob/master/docs/aws.md) to configure your DNS

* Then you can use the follwing commands to deploy a cluster
  * Public cluster

    The master and nodes in a public network and can be accessed by any ip

    ~~~ bash
    $ export KOPS_NAME="<cluster dns record>"
    $ export KOPS_STATE_STORE="s3://<bucket-name>"

    $ kops create cluster \
        --name $KOPS_NAME \
        --state $KOPS_STATE_STORE \
        --zones ap-northeast-2a \
        --yes
    ~~~

  * Private cluster

    The master and nodes in a private network, there will be a bastion node. user can only access the nodes using bastion

    ~~~ bash
    $ export KOPS_NAME="<cluster dns record>"
    $ export KOPS_STATE_STORE="s3://<bucket-name>"

    $ kops create cluster \
        --name $KOPS_NAME \
        --state $KOPS_STATE_STORE \
        --zones ap-northeast-2a \
        --bastion \
        --topology private \
        --networking weave \
        --yes
    ~~~

# How to modify the cluster configuration?

The cluster configuration was stored in the specified S3 bucket, you can execute the following command to edit the configuration file

~~~ bash
$ kops edit cluster --name <cluster name>
~~~

The configuration file looks like this

~~~ yaml
apiVersion: kops/v1alpha2
kind: Cluster
metadata:
  creationTimestamp: 2018-01-12T06:21:40Z
  name: 
spec:
  api:
    dns: {}
  authorization:
    alwaysAllow: {}
  channel: stable
  cloudProvider: aws
  configBase: s3://
  etcdClusters:
  - etcdMembers:
    - instanceGroup: master-ap-northeast-2a
      name: a
    name: main
  - etcdMembers:
    - instanceGroup: master-ap-northeast-2a
      name: a
    name: events
  iam:
    allowContainerRegistry: true
    legacy: false
  kubernetesApiAccess:
  - 0.0.0.0/0
  kubernetesVersion: 1.8.4
  masterInternalName: 
  masterPublicName: 
  networkCIDR: 172.20.0.0/16
  networking:
    kubenet: {}
  nonMasqueradeCIDR: 100.64.0.0/10
  sshAccess:
  - 0.0.0.0/0
  subnets:
  - cidr: 172.20.32.0/19
    name: ap-northeast-2a
    type: Public
    zone: ap-northeast-2a
  topology:
    dns:
      type: Public
    masters: public
    nodes: public
~~~

You can also change the instance configuration of nodes, just execute following command

~~~ bash
$ kops get instancegroups # get all instancegroups
$ kops edit instancegroups <instancegroups name>
~~~

The configuration looks like this

~~~ yaml
apiVersion: kops/v1alpha2
kind: InstanceGroup
metadata:
  creationTimestamp: 2018-01-12T06:21:42Z
  labels:
    kops.k8s.io/cluster: 
  name: master-ap-northeast-2a
spec:
  image: kope.io/k8s-1.8-debian-jessie-amd64-hvm-ebs-2017-12-02
  machineType: t2.micro
  maxSize: 1
  minSize: 1
  nodeLabels:
    kops.k8s.io/instancegroup: master-ap-northeast-2a
  role: Master
  subnets:
  - ap-northeast-2a
~~~

After save the configuration changes, you can use the following command to deploy the changes

~~~ bash
$ kops update cluster <cluster name>
~~~

# How to delete a cluster?

~~~ bash
$ kops delete cluster --name <cluster name> --yes
~~~

# Play kubernetes
After you deploy the cluster using *kops*, *kubectl* already set the context, you can use it directly

## How to install a dashboard?
We will install the [kubernetes/dashboard](https://github.com/kubernetes/dashboard)

* Install the dashboard

  ~~~ bash
  $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
  ~~~

* Create a service account to access the dashboard

  * create the service account

    ~~~ yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: admin-user
      namespace: kube-system
    ~~~

  * bind this account to admin role

    ~~~ yaml
    apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRoleBinding
    metadata:
      name: admin-user
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - kind: ServiceAccount
      name: admin-user
      namespace: kube-system
    ~~~

* Fetch the access token

  ~~~ bash
  $ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')

  Name:         admin-user-token-6gl6l
  Namespace:    kube-system
  Labels:       <none>
  Annotations:  kubernetes.io/service-account.name=admin-user
                kubernetes.io/service-account.uid=b16afba9-dfec-11e7-bbb9-901b0e532516

  Type:  kubernetes.io/service-account-token

  Data
  ====
  ca.crt:     1025 bytes
  namespace:  11 bytes
  token:      eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLTZnbDZsIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJiMTZhZmJhOS1kZmVjLTExZTctYmJiOS05MDFiMGU1MzI1MTYiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.M70CU3lbu3PP4OjhFms8PVL5pQKj-jj4RNSLA4YmQfTXpPUuxqXjiTf094_Rzr0fgN_IVX6gC4fiNUL5ynx9KU-lkPfk0HnX8scxfJNzypL039mpGt0bbe1IXKSIRaq_9VW59Xz-yBUhycYcKPO9RM2Qa1Ax29nqN
  ~~~

* Start the proxy

  ~~~ bash
  $ kubectl proxy
  ~~~

* Access the dashboard

  You can access the dashboard with [http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/)

  In the Login page, you should fill in the token got from the previous step

## How to deploy job on kubernetes?

We will deploy a simple container, it will output the PI value in stdout. its deployments looks like this

~~~ yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
~~~

We can deploy it using this command

~~~ bash
$ kubectl create -f <deployment file>
~~~

Then we can fetch the pod log

~~~ bash
$ kubectl logs job/pi

3.1415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679821480865132823066470938446095505822317253594081284811174502841027019385211055596446229489549303819644288109756659334461284756482337867831652712019091456485669234603486104543266482133936072602491412737245870066063155881748815209209628292540917153643678925903600113305305488204665213841469519415116094330572703657595919530921861173819326117931051185480744623799627495673518857527248912279381830119491298336733624406566430860213949463952247371907021798609437027705392171762931767523846748184676694051320005681271452635608277857713427577896091736371787214684409012249534301465495853710507922796892589235420199561121290219608640344181598136297747713099605187072113499999983729780499510597317328160963185950244594553469083026425223082533446850352619311881710100031378387528865875332083814206171776691473035982534904287554687311595628638823537875937519577818577805321712268066130019278766111959092164201989380952572010654858632788659361533818279682303019520353018529689957736225994138912497217752834791315155748572424541506959508295331168617278558890750983817546374649393192550604009277016711390098488240128583616035637076601047101819429555961989467678374494482553797747268471040475346462080466842590694912933136770289891521047521620569660240580381501935112533824300355876402474964732639141992726042699227967823547816360093417216412199245863150302861829745557067498385054945885869269956909272107975093029553211653449872027559602364806654991198818347977535663698074265425278625518184175746728909777727938000816470600161452491921732172147723501414419735685481613611573525521334757418494684385233239073941433345477624168625189835694855620992192221842725502542568876717904946016534668049886272327917860857843838279679766814541009538837863609506800642251252051173929848960841284886269456042419652850222106611863067442786220391949450471237137869609563643719172874677646575739624138908658326459958133904780275898
~~~

## How to deploy application on kubernetes?
We will deploy a http proxy container, it will allow us to access a website through the http proxy. its deployments looks like this

~~~ yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    run: http-proxy
  name: http-proxy-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      run: http-proxy
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        run: http-proxy
    spec:
      containers:
      - image: wernight/dante
        imagePullPolicy: Always
        name: http-proxy
        ports:
        - containerPort: 1080
          protocol: TCP
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
~~~

We can deploy it using this command

~~~ bash
$ kubectl create -f <deployment file>
~~~

Not like the previous demo, we want to access this container from outside, so we must expose this container to the world.

We need to create a service, its configuration looks like this

~~~ yaml
apiVersion: v1
kind: Service
metadata:
  name: http-proxy-service
spec:
  ports:
  - port: 1080
    protocol: TCP
    targetPort: 1080
  selector:
    run: http-proxy
  type: LoadBalancer
~~~

Then we can fetch the service information to get the exposed endpoint

~~~ bash
$ kubectl describe service http-proxy-service
Name:                     http-proxy-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 run=http-proxy
Type:                     LoadBalancer
IP:                       100.66.125.105
LoadBalancer Ingress:     a994bbf86a11e7b85d02c3e17c5ff-156353552.ap-northeast-2.elb.amazonaws.com
Port:                     <unset>  1080/TCP
TargetPort:               1080/TCP
NodePort:                 <unset>  31590/TCP
Endpoints:                100.96.1.10:1080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
~~~

Now we can try the http proxy, the endpoint is *LoadBalancer Ingress*, the port is *Port*

~~~ bash
$ curl --proxy socks5://a994bbf86a11e7b85d02c3e17c5ff-156353552.ap-northeast-2.elb.amazonaws.com:1080 https://www.baidu.com

<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Compatible content=IE=Edge><meta content=always name=referrer><link rel=stylesheet type=text/css href=https://ss1.bdstatic.com/5eN1bjq8AAUYm2zgoY3K/r/www/cache/bdorz/baidu.min.css><title>百度一下，你就知道</title></head> <body link=#0000cc> <div id=wrapper> <div id=head> <div class=head_wrapper> <div class=s_form> <div class=s_form_wrapper> <div id=lg> <img hidefocus=true src=//www.baidu.com/img/bd_logo1.png width=270 height=129> </div> <form id=form name=f action=//www.baidu.com/s class=fm> <input type=hidden name=bdorz_come value=1> <input type=hidden name=ie value=utf-8> <input type=hidden name=f value=8> <input type=hidden name=rsv_bp value=1> <input type=hidden name=rsv_idx value=1> <input type=hidden name=tn value=baidu><span class="bg s_ipt_wr"><input id=kw name=wd class=s_ipt value maxlength=255 autocomplete=off autofocus=autofocus></span><span class="bg s_btn_wr"><input type=submit id=su value=百度一下 class="bg s_btn" autofocus></span> </form> </div> </div> <div id=u1> <a href=http://news.baidu.com name=tj_trnews class=mnav>新闻</a> <a href=https://www.hao123.com name=tj_trhao123 class=mnav>hao123</a> <a href=http://map.baidu.com name=tj_trmap class=mnav>地图</a> <a href=http://v.baidu.com name=tj_trvideo class=mnav>视频</a> <a href=http://tieba.baidu.com name=tj_trtieba class=mnav>贴吧</a> <noscript> <a href=http://www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1 name=tj_login class=lb>登录</a> </noscript> <script>document.write('<a href="http://www.baidu.com/bdorz/login.gif?login&tpl=mn&u='+ encodeURIComponent(window.location.href+ (window.location.search === "" ? "?" : "&")+ "bdorz_come=1")+ '" name="tj_login" class="lb">登录</a>');
                </script> <a href=//www.baidu.com/more/ name=tj_briicon class=bri style="display: block;">更多产品</a> </div> </div> </div> <div id=ftCon> <div id=ftConw> <p用d=lh> <a href=http://home.baidu.com>关于百度</a> <a href=http://ir.baidu.com>About Baidu</a> </p> <p id=cp>&copy;2017&nbsp;Baidu&nbsp;<a href=http://www.baidu.com/duty/>使
  百度前必读</a>&nbsp; <a href=http://jianyi.baidu.com/ class=cp-feedback>意见反馈</a>&nbsp;京ICP证030173号&nbsp; <img src=//www.baidu.com/img/gs.gif> </p> </div> </div> </div> </body> </html>
~~~

Done!!
