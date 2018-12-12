#!/usr/bin/env groovy

pipeline {
    agent any
        
        stages {
        stage('Awaiting for user input') {
            when {
                  branch "master"
           }
            steps {
                script {
                    def response
                    timeout(time:120, unit:'SECONDS') {
                    response = input message: 'User',
                    parameters: [string(defaultValue: 'user1',
                    description: 'Enter your userID:', name: 'userid')]
                }
                echo "Username = " + response 
            } 
            }
        }
        stage('Read data from AWS Batch') {
          steps {
              script {
                  def job_name = sh returnStdout: true, script: 'aws batch describe-job-queues --job-queues --region us-east-1'
                  env.job_name = job_name
                  def job_def = sh returnStdout: true, script: 'aws batch describe-job-definitions | jq -r \'.jobDefinitions[] | select(.status=="ACTIVE") | .jobDefinitionArn\''
                  env.job_def = job_def
                }
            }
        } 
       stage('Submit new job to AWS Batch') {
          steps {
              script {
                def job_name =  readJSON text: env.job_name
                job_name.jobQueues.each { jobQueue ->
                def job_id = sh returnStdout: true, script: "aws batch submit-job --job-name example --job-queue ${jobQueue.jobQueueName}  --job-definition ${env.job_def}"
                env.job_id = job_id 
                echo "${env.job_id}"
                def job_id_num =  readJSON text: env.job_id
                job_id_num.each { jobId ->
                echo "${job_id_num.jobId}"
                env.test = "${job_id_num.jobId}"
                echo "${env.test}"
                }
                }
                }
            }
        }
        stage('Job status checking') {
          steps {
              script {
                sh   '''#!/bin/bash
                        i=0
                        out="aws batch list-jobs --job-queue first-run-job-queue --job-status SUCCEEDED | jq -r '.jobSummaryList[] | select(.status=="SUCCEEDED") | .jobId' | grep $env.test"
                        for (( i=1; i <= 100; i++ ))
                        do
                         if [ -n "$out" ]; then
                            echo "The job was successfully submited"
                            break
                         fi 
                         sleep 2
                        done'''
              }
            }    
        }
    }
}
