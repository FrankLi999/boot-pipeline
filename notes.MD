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

   https://ruddra.com/posts/openshift-python-gunicorn-nginx-jenkins-pipelines-part-two/

   set up jenkins for openshift -- 
       check the booksore example
       https://github.com/pavankjadda/BookStore