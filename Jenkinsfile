pipeline{
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = 'eu-west-2'
    }
    tools{
        jdk 'jdk17'
        terraform 'terraform'
    }
    stages{
        
        stage('Initializing Terraform'){
            steps{
                script{
                    dir('terraform'){
                         sh 'terraform init'
                    }
                }
            }
        }
        stage('Validating Terraform'){
            steps{
                script{
                    dir('terraform'){
                         sh 'terraform validate'
                    }
                }
            }
        }
        stage('checkout from Git'){
            steps{
                git branch: 'test', url: 'https://github.com/kolab-web/TERRAFORM-JENKINS-CICD'
            }
        }
        stage('Terraform version'){
             steps{
                 sh 'terraform --version'
                }
        }
         
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                dir("${WORKSPACE}"){
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner 
                    -Dsonar.projectName=jenkins \
                    -Dsonar.projectKey=jenkins \
                    -Dsonar.projectBaseDir=/var/lib/jenkins/workspace/aws_instance \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://localhost:9000 \
                    -Dsonar.login=sqp_338fa99d800dbb7887756112d868a83c4413da02
                    '''
                }
            }
            }
        }    
        // stage("quality gate"){
        //    steps {
        //         script {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
        //         }
        //     } 
        // }
        // }
        // stage('OWASP Dependency Check') {
        //     steps {
        //         dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP-Check'
        //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //     }
        // }
        stage('Snyk Test') {
           steps {
               script {
                   // Run Snyk test
                   withCredentials([string(credentialsId: 'snyk', variable: 'SNYK_TOKEN')]) {
                       sh 'echo $(pwd)'
                       sh 'snyk test --all-projects'
                   }
               }
           }
       }
        stage('Terraform init'){
            steps{
                sh 'terraform init'
            }
        }
        stage('Terraform plan'){
            steps{
                sh 'terraform plan'
            }
        }
        stage('Terraform apply'){
            steps{
                sh 'terraform ${action} --auto approve'
            }
        }
    }
 }