1. install plugins:
   pipeline utility steps
   cucumber report plugin
   build flow plugin
   https://jenkins.io/doc/pipeline/steps/pipeline-build-step/
   
   There are two ways in Jenkins to setup a job triggered by another
		"Pipeline Build Step" to trigger the new build for a given job
			 build job: 'downstream-job-name', parameters: [
		        string(name: 'upsteam_project_name', value: env.NAME)
		    ], wait: false
		"Upstream Trigger" to be automatically triggered by upstream job
			triggers {
			    cron('0 10 * * *')
			    upstream(upstreamProjects: "upstream-project-name,another-upstream-project-name", threshold: hudson.model.Result.SUCCESS)
			}
2. global tools
   
   path: Maven363
   name: /opt/maven
3. create three local branches

   git branch development
   git branch test,
   and
   git branch production
   
   list branch:
   git branch
   
   
   delete branch:
   
   // delete branch locally
   git branch -d development

   // delete branch remotely
   git push origin --delete development

4. Docker build for dev

5. Openshift build for testing
   
   Deploy to Openshift from external Jenkins server:
    to use Openshift Client Jenkins Plugin:
     1) update /etc/environment,
        added directory to "oc" to the PATH env var
     2). create a openshift credential named such as "openshift-token"
         Credentials -> System -> Global credentials -> Add Credential 
        kind: Openshift Token for Openshift client plugin
        > "oc whoami -t" to get the auth token

        or: cznQP-n4pBk-cnXTg-nkevH
        
     3) Config openshift client jenkins pipeline:
        Manage Jenkins -> Configure System -> and find the OpenShift Plugin section.
        
     
            Cluster Name: local-cluster
            API Server URL: https://api.crc.testing:6443
            Added a Jenkins user name/password credential with id: local-openshift
            Disable TLS verify: checked
            copy/paste the certifcate for  "api.crc.testing" from the browser:
                -----BEGIN CERTIFICATE-----
                MIIDRTCCAi2gAwIBAgIIaijRpQEmG78wDQYJKoZIhvcNAQELBQAwNzESMBAGA1UE
                CxMJb3BlbnNoaWZ0MSEwHwYDVQQDExhrdWJlLWFwaXNlcnZlci1sYi1zaWduZXIw
                HhcNMjAwMTI0MDMzMjM5WhcNMjAwMjIzMDMzMjQwWjAaMRgwFgYDVQQDEw9hcGku
                Y3JjLnRlc3RpbmcwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCx1lx8
                Ok3poPYnVhhV+qdRRJCpAkTcff8e7sKCVdXX9xK8+bO2ettM8e0g5GecOPMo6PZK
                kqChRiZVBZWB4AI41IW129NxdY14iwnB6/zs46UgwatmfEXkMQt2LivCoiyV9sZe
                +L4p8IfWAdjqKX5bgia0aTazqNVYcQL4gzSC4tk8qubxJcjbP4hVJBZ0+43e3wb1
                GkdfTXSh1yoVQK6TCwGdYm+RG6ZWdKZpyrLNo4MlvsQJ0YmPP235ZIHNOODRdBH2
                xZg7f/c230iLTKRZ/NLepeuHxcdzU3M6/I0vl52om2VTktQ1VceUjDN2Q4j2MEpm
                L13kyD3lqRxEcDiBAgMBAAGjcjBwMA4GA1UdDwEB/wQEAwIFoDATBgNVHSUEDDAK
                BggrBgEFBQcDATAMBgNVHRMBAf8EAjAAMB8GA1UdIwQYMBaAFN46YMxQu2ZrmsyE
                RgARUDN2hPLiMBoGA1UdEQQTMBGCD2FwaS5jcmMudGVzdGluZzANBgkqhkiG9w0B
                AQsFAAOCAQEAPMvF5LKY6OW2Vrv8UvlPROdN10cmZWtFBsnmcbg0CblLT29E2vJ2
                DsRz8tqRwk26a6F5NRrx7JbYBMUK2XnEChL01zPSGdTLJu7Ly/6BcKRF5xZkqD2p
                EJmuHRibANZDrWkLllAIqZkKRx+nynP0ZFK5DZFX4/QZPj/vYZ1+rFipA4flBaRa
                3uLXibtbivXZD3To9MRMPckxHtXvIgObNEP0iR5548Ba8Wy4FYPS36m7ssfBlGI7
                XPwLkgUUHgSjeS8f5sn4sbBf80OUEjJB6IH3ygCSB1ep27PeSNYvPL4iEnP1t0R/
                4VBLlt6cM0UUPS2pSlPlQ/H4UJ5scJHdBA==
                -----END CERTIFICATE-----

            openshift.withCluster('local-cluster', 'local-openshift') {  
            }
     
     
     option 1: config the following parameters:
        KUBERNETES_SERVICE_HOST: api.crc.testing
        KUBERNETES_SERVICE_PORT_HTTPS: 8443
        SKIP_TLS: true
        AUTH_TOKEN:  use "oc whoami -t" to get the auth token
            openshift.withCluster() {
                //TODO: this does not work
                // java.nio.file.NoSuchFileException:  /var/run/secrets/kubernetes.io/serviceaccount/token
            }
        
    option 2: 
         jenkins console -> Manage Jenkins -> Manage System
         in the "OpenShift Client Plugin", 
           add a cluster:
                	Cluster Name: local-cluster
                    API Server URL: https://api.crc.testing:6443
                    Added a Jenkins user name/password credential with id: local-openshift
                    Disable TLS verify: checked

            openshift.withCluster('local-cluster', 'local-openshift') {  
            }
    option 3: 
        openshift.withCluster('https://api.crc.testing:6443', '7VYPRf-y-GnXIY58ii8nczqh6l7xRbHYl3NzRfnDuPY') {
            //resole token via using command 
            // "oc whoami -t" to get the auth token
        }        
    There are a few ways to use an external Jenkins to deploy applications to OpenShift:

    http://v1.uncontained.io/playbooks/continuous_delivery/external-jenkins-integration.html

    https://blog.openshift.com/using-openshift-pipeline-plugin-external-jenkins/

    OpenShift Pipeline Plugin: Openshift Pipeline Jenkins Plugin
        o You can install an oc command line tool on the external Jenkins 
        o server and use oc mechanism to deploy applications to OpenShift.
        o You can use REST APIs to interact with OpenShift.
        o You can use OpenShift plugin to deploy applications to OpenShift.
     
     
     use "oc whoami -t" to get the auth token
     detect api server url: oc config view --minify -o jsonpath='{.clusters[*].cluster.server}'

   Use Jenkins deployed on Openshift cluster - Openshift Client Jenkins Plugin
   https://medium.com/t/@pavankjadda/spring-boot-application-deployment-with-jenkins-pipeline-on-open-shift-cluster-f9fafdb0d6a9

   https://github.com/pavankjadda/BookStore

   https://developers.redhat.com/blog/2019/05/02/get-started-with-jenkins-ci-cd-in-red-hat-openshift-4/

   https://ruddra.com/posts/openshift-python-gunicorn-nginx-jenkins-pipelines-part-two/

   SSH into openshift master node with AWS keypair you created while setting up the cluster
        $ ssh -i <ssh_key.pem> centos@<openshift_master_hostname>

   set up jenkins for openshift -- 
       check the booksore example
       https://github.com/pavankjadda/BookStore