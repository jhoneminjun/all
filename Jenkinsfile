pipeline {
  agent any
  stages {
    stage ('build') {
      steps {
        sh 'printenv'
      }
    }
    stage ('Publish ECR') {
      steps {
        withEnv (["AWS_ACCESS_KEY=${env.AWS_ACCESS_KEY_ID}", "AWS_SECRET_ACCESS_KEY=${env.AWS_SECRET_ACCESS_KEY}", "AWS_DEFAULT_REGION=${env.AWS_DEFAULT_REGION}"]) {
          sh 'docker login -u AWS -p $(aws ecr get-login-password --region ap-northeast-2) 548021806095.dkr.ecr.ap-northeast-2.amazonaws.com'
          sh 'docker build -t test .'
          sh 'docker tag test:latest 548021806095.dkr.ecr.ap-northeast-2.amazonaws.com/test-ecr:test1:""$BUILD_ID""'
          sh 'docker push 548021806095.dkr.ecr.ap-northeast-2.amazonaws.com/test-ecr:test1:""$BUILD_ID""'
        }
      }
    }
    stage('Push Yaml'){
      steps{
        script {
          try {
            git url: 'https://github.com/jhoneminjun/argocd', branch: "main", credentialsId: 'github'
            // sh "rm -rf /var/lib/jenkins/workspace/${env.JOB_NAME}/*"
            sh """
            #!/bin/bash
            cat>deploy.yaml<<-EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
  namespace: nginx
  labels:
    app: web
spec:
  replicas: 2
  selector:
    matchLabels:
       app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - image: 548021806095.dkr.ecr.ap-northeast-2.amazonaws.com/test-ecr:test1:${env.BUILD_NUMBER}
        name: web-nginx
EOF"""
            //sh "cat /var/lib/jenkins/workspace/${env.JOB_NAME}/yaml/deploy.yaml"
                        withCredentials([gitUsernamePassword(credentialsId: 'github')]) {
                            sh """
                            git add deploy.yaml
                            git commit -m "Deploy ${env.JOB_NAME} ${env.BUILD_NUMBER}"
                            git push https://github.com/jhoneminjun/argocd.git
                            """
                        }                     
                        env.pushYamlResult=true
                        } catch (error) {
                        print(error)
                        }
                        
                        env.pushYamlResult=false
                        currentBuild.result = 'FAILURE'
        }
      }
    }
  }
}
