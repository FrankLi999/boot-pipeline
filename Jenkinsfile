pipeline {
    agent any
    // agent { label 'java8' }
    // tools { 
        // maven 'Maven 3.6.3' 
        // jdk 'jdk8' 
    // }
    environment {
        // EMAIL_RECIPIENTS = 'jenkine.build@test.com'
        EMAIL_RECIPIENTS = 'f_li1@yahoo.com'
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                ''' 
            }
        }
        stage('Build with unit testing') {
            steps {
                // Run the maven build
                script {
                    // Get the Maven tool.
                    // ** NOTE: This 'M3' Maven tool must be configured
                    // **       in the global configuration.
                    echo 'Pulling...' + env.BRANCH_NAME
                    def mvnHome = tool 'Maven363'
                    echo "mvnHome = ${mvnHome}"
                    if (isUnix()) {
                        def targetVersion = getDevVersion()
                        print 'target build version...'
                        print targetVersion
                        sh "'${M2_HOME}/bin/mvn' -Dintegration-tests.skip=true -Dbuild.number=${targetVersion} clean package"
                        def pom = readMavenPom file: 'pom.xml'
                        print 'pom version...'
                        print pom.version
                        // get the current development version
                        developmentArtifactVersion = "${pom.version}-${targetVersion}"
                        print pom.version
                        // execute the unit testing and collect the reports
                        junit '**//*target/surefire-reports/TEST-*.xml'
                        archiveArtifacts 'target*//*.jar'
                    } else {
                        bat(/"${M2_HOME}\bin\mvn" -Dintegration-tests.skip=true clean package/)
                        def pom = readMavenPom file: 'pom.xml'
                        print pom.version
                        junit '**//*target/surefire-reports/TEST-*.xml'
                        archiveArtifacts 'target*//*.jar'
                    }
                }
            }
        }
        stage('Integration tests') {
            // Run integration test
            steps {
                script {
                    def mvnHome = tool 'Maven363'
                    if (isUnix()) {
                        // just to trigger the integration test without unit testing
                        //  sh 'mvn clean -o -gs `pwd`/configuration/settings.xml test'
                        sh "'${mvnHome}/bin/mvn'  verify -Dunit-tests.skip=true"
                    } else {
                        bat(/"${mvnHome}\bin\mvn" verify -Dunit-tests.skip=true/)
                    }

                }
                // cucumber reports collection
                cucumber buildStatus: null, fileIncludePattern: '**/cucumber.json', jsonReportDirectory: 'target', sortingMethod: 'ALPHABETICAL'
            }
        }
        // stage('Sonar scan execution') {
            // Run the sonar scan
            // steps {
                // script {
                    // def mvnHome = tool 'Maven 3.5.2'
                    // withSonarQubeEnv {

                        // sh "'${mvnHome}/bin/mvn'  verify sonar:sonar -Dintegration-tests.skip=true -Dmaven.test.failure.ignore=true"
                    // }
                // }
            // }
        // }
        // waiting for sonar results based into the configured web hook in Sonar server which push the status back to jenkins
        // stage('Sonar scan result check') {
        //    steps {
        //        timeout(time: 2, unit: 'MINUTES') {
        //            retry(3) {
        //                script {
        //                    def qg = waitForQualityGate()
        //                    if (qg.status != 'OK') {
        //                        error "Pipeline aborted due to quality gate failure: ${qg.status}"
        //                    }
        //                }
        //            }
        //        }
        //    }
        // }
        stage('Development deploy approval and deployment') {
            steps {
                script {
                    print 'current build result ... '
                    print currentBuild.result
                    if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
                        // timeout(time: 3, unit: 'MINUTES') {
                            // you can use the commented line if u have specific user group who CAN ONLY approve
                            //input message:'Approve deployment?', submitter: 'it-ops'
                            // input message: 'Approve deployment?'
                        // }
                        timeout(time: 2, unit: 'MINUTES') {
                            //
                            if (developmentArtifactVersion != null && !developmentArtifactVersion.isEmpty()) {
                                // replace it with your application name or make it easily loaded from pom.xml
                                def jarName = "application-${developmentArtifactVersion}.jar"
                                echo "the application is deploying ${jarName}"
                                // NOTE : CREATE your deployemnt JOB, where it can take parameters whoch is the jar name to fetch from jenkins workspace
                                build job: 'boot-docker-pipeline', parameters: [[$class: 'StringParameterValue', name: 'buildNumber', value: developmentArtifactVersion]]
                                echo 'the application is deployed !'
                            } else {
                                error 'the application is not  deployed as development version is null!'
                            }

                        }
                    }
                }
            }
        }
        stage('DEV sanity check') {
            steps {
                // give some time till the deployment is done, so we wait 45 seconds
                sleep(45)
                script {
                    if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
                        timeout(time: 2, unit: 'MINUTES') {
                            script {
                                def mvnHome = tool 'Maven363'
                                //NOTE : if u change the sanity test class name , change it here as well
                                sh "'${mvnHome}/bin/mvn' -Dtest=ApplicationSanityCheck_ITT surefire:test"
                                sh "docker container stop boot-pipeline-'${developmentArtifactVersion}'"
                                sh "docker container rm boot-pipeline-'${developmentArtifactVersion}'"
                                sh "docker rmi xli9999/boot-pipeline:'${developmentArtifactVersion}'"
                            }

                        }
                    }
                }
            }
        }
	}
    post {
    //     always {
    //         junit '**/reports/junit/*.xml'
    //     }
        // Always runs. And it runs before any of the other post conditions.
        always {
            // Let's wipe out the workspace before we finish!
            deleteDir()
        }
        success {
            // sendEmail("Successful");
            echo "sendEmail Successful"
        }
        unstable {
            // sendEmail("Unstable");
            echo "sendEmail Unstable"
        }
        failure {
            // sendEmail("Failed");
            echo "sendEmail Failed"
        }
    } 
}
def developmentArtifactVersion = ''
def releasedVersion = ''
// get change log to be send over the mail
@NonCPS
def getChangeString() {
    MAX_MSG_LEN = 100
    def changeString = ""

    echo "Gathering SCM changes"
    def changeLogSets = currentBuild.changeSets
    for (int i = 0; i < changeLogSets.size(); i++) {
        def entries = changeLogSets[i].items
        for (int j = 0; j < entries.length; j++) {
            def entry = entries[j]
            truncated_msg = entry.msg.take(MAX_MSG_LEN)
            changeString += " - ${truncated_msg} [${entry.author}]\n"
        }
    }

    if (!changeString) {
        changeString = " - No new changes"
    }
    return changeString
}

def sendEmail(status) {
    mail(
            to: "$EMAIL_RECIPIENTS",
            subject: "Build $BUILD_NUMBER - " + status + " (${currentBuild.fullDisplayName})",
            body: "Changes:\n " + getChangeString() + "\n\n Check console output at: $BUILD_URL/console" + "\n")
}

def getDevVersion() {
    def gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    def versionNumber;
    if (gitCommit == null) {
        versionNumber = env.BUILD_NUMBER;
    } else {
        versionNumber = gitCommit.take(8);
    }
    print 'build  versions...'
    print versionNumber
    return versionNumber
}

def getReleaseVersion() {
    def pom = readMavenPom file: 'pom.xml'
    def gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    def versionNumber;
    if (gitCommit == null) {
        versionNumber = env.BUILD_NUMBER;
    } else {
        versionNumber = gitCommit.take(8);
    }
    return pom.version.replace("-SNAPSHOT", ".${versionNumber}")
}

// if you want parallel execution , check below :
/* stage('Quality Gate(Integration Tests and Sonar Scan)') {
           // Run the maven build
           steps {
               parallel(
                       IntegrationTest: {
                           script {
                               def mvnHome = tool 'Maven 3.5.2'
                               if (isUnix()) {
                                   sh "'${mvnHome}/bin/mvn'  verify -Dunit-tests.skip=true"
                               } else {
                                   bat(/"${mvnHome}\bin\mvn" verify -Dunit-tests.skip=true/)
                               }
                           }
                       },
                       SonarCheck: {
                           script {
                               def mvnHome = tool 'Maven 3.5.2'
                               withSonarQubeEnv {
                                   // sh "'${mvnHome}/bin/mvn'  verify sonar:sonar -Dsonar.host.url=http://bicsjava.bc/sonar/ -Dmaven.test.failure.ignore=true"
                                   sh "'${mvnHome}/bin/mvn'  verify sonar:sonar -Dmaven.test.failure.ignore=true"
                               }
                           }
                       },
                       failFast: true)
           }
       }*/
