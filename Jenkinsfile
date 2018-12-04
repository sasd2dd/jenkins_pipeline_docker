def app

pipeline {
    agent any
    options {
      buildDiscarder(logRotator(numToKeepStr: '2'))
    }


    stages {
        stage('Checkout') {
            agent any
            steps {
                checkout([$class: 'GitSCM',
				branches: [[name: "origin/master"]],
				userRemoteConfigs: [[
                  url: 'https://github.com/sasd2dd/jenkins_pipeline_docker.git',
                  credentials:'tim-git'
                  ]],
				])
            }
        }
        stage('build'){
                agent any
            steps{
                println "build"
                script {
					app = docker.build('timgondasr/web2_web:latest')
                    RETURN_STATUS = sh "docker-compose up -d"
                }
            }
        }
        stage('test'){
                agent any
            steps{
                println "tests"
                script {
                    WEB_RETURN = sh (
                        script: 'curl http://localhost:5000',
                        returnStdout: true
                    ).trim()
                    print "Web Returned: " + WEB_RETURN
                    if ( WEB_RETURN.indexOf("carbon",0)  == -1)
                    {
                        error("Build Failed as expected result was not found")
                    }
                    else
                    {
                        print "All Web Tests Passed"
                    }
                }
            }
        }
        stage('approval to deploy'){
            //input{
                //message "Please approve the deployment"
                //ok "Ok"
                //parameters {
                //    string(name: 'Approver', defaultValue: "none", description: "Person approving the deployment")
                //    }
               // }
            
            steps{
                print "Approved"
                print "${currentBuild.fullDisplayName}"
            	//print "${env.BUILD_URL} Approved by ${Approver}"
            }
        }
        stage('push to repo'){
      	    agent any
      		steps {
		        script{
					docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
		            	app.push("${BUILD_TAG}")
		        	}
		        }
       			print "pushed to dockerhub"
      		}
      	}
        stage('cleanup'){
                agent any
            steps{
                println "build"
                script {
                    sh "docker-compose stop"
                }
            }
        }
        stage('deploy-with-ansible'){
            agent any
            steps{
                println "deploy with ansible"
                script {
                    ansiblePlaybook( inventory: 'ansiblecfg/inventory', playbook: 'ansiblecfg/site.yml')
                }
            }
        }
    }
	post {
	    success {
	      //  mail to: 'timothy.gonda@gmail.com',
	      //       subject: "Success Pipeline: ${currentBuild.fullDisplayName}",
	      //       body: "Completed ${env.BUILD_URL}"
	      
	      print "Success: ${env.BUILD_URL}"
	    }
	    failure {
	      //  mail to: 'timothy.gonda@gmail.com',
	      //       subject: "Success Pipeline: ${currentBuild.fullDisplayName}",
	      //       body: "Completed ${env.BUILD_URL}"
	      
	      print "Failed Build: ${env.BUILD_URL}"
	    }
	}
}