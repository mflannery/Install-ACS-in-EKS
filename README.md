# Install Red Hat Advanced Cluster Security in AWS EKS

Mike Flannery 

Staff Solutions Architect

Red Hat

The following instructions are based on the official documentation: [https://docs.openshift.com/acs/3.66/installing/install-quick-roxctl.html](https://docs.openshift.com/acs/3.66/installing/install-quick-roxctl.html)

**First make sure the EKS Cluster meets the prerequisites**

[https://docs.openshift.com/acs/3.66/installing/prerequisites.html](https://docs.openshift.com/acs/3.66/installing/prerequisites.html)

The document above describes the prerequisites for installing Central, Scanner and Sensor so make sure to meet those minimum requirements. The install will check that the minimums are met and will throw an error if they are not met.

**Next create a StackRox account**

[https://signup.stackrox.io](https://signup.stackrox.io)

And then let us know the email address and username for the account so we can get it authenticated.

**Installing ACS**

Once the minimum requirements are confirmed as met the installation can proceed. In this document the installation will be done via the roxctl client. You can see the official documentation [here](https://docs.openshift.com/acs/3.66/installing/install-quick-roxctl.html) but this describes installing ACS into OpenShift and not EKS. The installation steps are the same but the CLI commands are slightly different and are listed in this document. 

**Install roxctl**



1. Download the latest version of the roxctl CLI:
    1. `$ curl -O https://mirror.openshift.com/pub/rhacs/assets/3.66.0/bin/Linux/roxctl`
2. Make the roxctl binary executable:
    2. `$ chmod +x roxctl`





3. Place the roxctl binary in a directory that is on your PATH: \
To check your PATH, execute the following command:
    3. `$ echo $PATH`
4. Verify the roxctl version you have installed:
    4. `$ roxctl version`

**Install Central**

Run roxctl to generate the central settings.


```
$ roxctl central generate interactive
```


Leave everything default except the following



* Admin Password - enter any admin password here
* Orchestrator - set to k8s
* Telemetry - set to false
* Method of exposing central - set to lb
* Enter central volume type
    * For this we need to check the storage classes. In a new terminal on the same server run:

            ```
            $ kubectl get sc
            ```


* Your output should like something like the following


```
NAME      PROVISIONER       RECLAIMPOLICY  VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION  AGE
gp2 (default)  kubernetes.io/aws-ebs  Delete     WaitForFirstConsumer  false         3h38m

```



* In the output above we can see that I have a storage class called gp2 and this is the default. This means I can choose pvc for the central volume type. If you have nothing listed here we will have to do something different but most systems should have some persistent storage listed here.
* For the storage size you can leave it at 100 or for demo systems you can reduce this to 40.

Running the roxctl command will create a central-bundle directory in the current working directory. The information in this directory will be used in the following steps.

To set up the stackrox registry, run the following command.

`$ ./central-bundle/central/scripts/setup.sh` 

You will use the username and password you used when creating the stackrox account above. If the password has been forgotten, run the following command to retrieve it.


```
$ cat central-bundle/password
```


To install central, run 


```
$ kubectl apply -R -f central-bundle/central
```


The central deployment can be verified with


```
$ kubectl get pod -n stackrox
NAME           READY  STATUS  RESTARTS  AGE
central-5857f778d-wzc44  1/1   Running  0     76s
```


In the output above we can see the pod is running so we are ready for the next step.

Obtain the URL for accessing ACS by running the following:


```
$ kubectl -n stackrox get svc central-loadbalancer
NAME          TYPE      CLUSTER-IP    EXTERNAL-IP                                 PORT(S)     AGE
central-loadbalancer  LoadBalancer  10.100.173.184  a4787b21547b140c8a8e8fd3981d10a5-975395780.us-gov-west-1.elb.amazonaws.com  443:31616/TCP  3m7s
```


In the output above, we can see the external URL highlighted. You should now be able to access the ACS GUI by entering the url above into your browser. Make sure to put https:// in front of the URL in the browser address bar. In my case this would be `https://a4787b21547b140c8a8e8fd3981d10a5-975395780.us-gov-west-1.elb.amazonaws.com`

**Install scanner**

Installing scanner is pretty much the same as installing central. 


```
$ ./central-bundle/scanner/scripts/setup.sh
$ kubectl apply -R -f central-bundle/scanner
```


Verify the pods have been successfully created with


```
$ kubectl get pod -n stackrox
```


If any pods are listed as pending for more than a few minutes, run 


```
$ kubectl describe pod -n stackrox | less
```


Search for the pod name and then search for “Warning” to see why the pod is in a pending state. Usually pods end up pending when there is either not enough CPU or memory resources. If all pods are running then ACS has been successfully installed. Login to the ACS GUI using the username “admin” and the password that was entered when generating the central setup files.
