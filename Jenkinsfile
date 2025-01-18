pipeline {
    agent { label 'slave01' }

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        MAVEN_HOME = '/usr/share/maven'
        PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${env.PATH}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }

        stage('Set up Java 17') {
            steps {
                echo 'Setting up Java 17...'
                sh 'sudo apt update'
                sh 'sudo apt install -y openjdk-17-jdk'
            }
        }

        stage('Set up Maven') {
            steps {
                echo 'Setting up Maven...'
                sh 'sudo apt install -y maven'
            }
        }

        stage('Build with Maven') {
            steps {
                echo 'Building project with Maven...'
                sh 'mvn clean package'
            }
        }
        stage('Build test') {
           steps { 
             echo "testing build package"
             sh 'mvn test'
           }
              }
        stage ('SonarQube') {
          steps {
            echo 'SonarQube test'
            sh '''
            mvn clean verify sonar:sonar \
            -Dsonar.organization=${SONAR_USER}\
            -Dsonar.host.url=${SONAR_URL} \
            -Dsonar.projectKey=${PROJECT_NAME}\
            -Dsonar.login=${SONAR_TOKEN}
            '''
          }
        }
        stage('Upload Artifact') {
            steps {
                echo 'Uploading artifact...'
               curl -u "${ARTIFACT_USERNAME}:${ARTIFACT_TOKEN}" \
               -T target/my-helloworld-1.0.1.war \
               "${ARTIFACT_USERNAME}/${ARTIFACT_REPOSITORY}/my-helloworld-1.0.1.war"
            }
        }

        stage('Deploy to staging') {
          when {
             branch !main
           }
            steps {
                echo "Deploying to staging"
                 sh 'cp hello-world-war/target/*.war /opt/apache-tomcat-10.1.34/webapps/'
                }
            }
        stage('Deploy to Production') {
           when {
             branch 'main'
           }
            steps {
                echo "Deploying to staging"
                 sh 'scp hello-world-war/target/*.war root@"${PRODUCTION_IP}"/opt/apache-tomcat-10.1.34/webapps/'
                }
            } 
          }

   post{
       success{
           echo "Build is success"
           sh 'echo " Build is Success " | mail -s "Build Success"  sonyhl1918@gmail.com'
       }
       failure{
           echo "Build failed"
           sh 'echo " Build failed " | mail -s "Check logs"  sonyhl1918@gmail.com'
       }
   }
}
