pipeline {
    agent any
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
            input{
                message "Please approve the deployment"
                ok "Ok"
                parameters {
                    string(name: 'Approver', defaultValue: "none", description: "Person approving the deployment")
                    }
                }
                steps{
                    print "Approver is : " + Approver
                }
        }
        stage('push to repo')
        {
            agent any
            steps{
                print "Pushed to repo"
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
                    ansiblePlaybook(credentialsId: 'private_key', inventory: 'ansiblecfg/hosts', playbook: 'site.yml')
                }
            }
        }
    }
}