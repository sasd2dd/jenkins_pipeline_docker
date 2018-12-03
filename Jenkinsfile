pipeline {
    agent any
    options {
      buildDiscarder(logRotator(numToKeepStr: '2'))
    }

    stages {
        /* stage('Checkout') {
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
        } */
        stage('build'){
                agent any
            steps{
                println "build"
                script {
                    RETURN_STATUS = sh "docker-compose up -d --build"
                }
            }
        }
        stage('test'){
                agent any
            steps{
                println "tests"
                script {
                    //sh "curl http://localhost:5000"
                    
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
            //    message "Please approve the deployment"
            //    ok "Ok"
            //    parameters {
            //        string(name: 'Approver', defaultValue: "none", description: "Person approving the deployment")
            //        }
            //    }
            steps{
                print "Approved"
            }
        }
        stage('push to repo')
        {
            agent any
      		steps {
      		    script {
        			webimg = docker.image("web2_web:latest")
        			docker.withRegistry("https://store.docker.com/sasd2dd", 'dockerhub'){
                     	newImage.push("latest")
                    }
        			print "pushed to dockerhub"
        		}
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
}