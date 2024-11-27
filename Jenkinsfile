pipeline {
    agent {
        label 'AGENT-1'
    }
       options{
        timeout(time: 10, unit: 'MINUTES')
        disableConcurrentBuilds()
        retry(1)
    }
    environment {
        DEBUG = 'true'
        appVersion = ''
        region = 'us-east-1'
        project = 'expense'
        accountid = '311141538313'
        component = 'frontend'
        environment = 'dev'
    }
    stages {
        stage('Print the version') {
            steps {
                    script{
                        def packageJson = readJSON file: 'package.json'
                        appVersion = packageJson.version
                        echo "App version: ${appVersion}"
                }
            }
        }
        stage('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }
        // here we are pushing docker image to ECR. Taking the PUSH commands from ecr repository and using here
        stage ('Docker build') {
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                sh """
                    aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${accountid}.dkr.ecr.us-east-1.amazonaws.com
                    docker build -t ${accountid}.dkr.ecr.${region}.amazonaws.com/${project}/${component}:${appVersion} .
                    docker images
                    docker push ${accountid}.dkr.ecr.${region}.amazonaws.com/${project}/${component}:${appVersion}
                    

                """
                }
            }
        }
        stage ('Deploy') {
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-creds'){
                    sh """
                        aws eks update-kubeconfig --region ${region} --name ${project}-${environment}
                        cd helm-backend
                        sed -i 's/IMAGE_VERSION/${appVersion}/g' values.yaml
                        helm upgrade --install ${component} -n ${project} -f values.yaml .
                    """
                }
            }
        }
    }
}