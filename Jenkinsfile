pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('Access_key_ID')
        AWS_SECRET_ACCESS_KEY = credentials('Secret_access_key')
        AWS_S3_BUCKET = "eman-a-a"
        ARTIFACT_NAME = "eman.jar"
        AWS_EB_APP_NAME = "eman-app"
        AWS_EB_APP_VERSION = "${BUILD_ID}"
        AWS_EB_ENVIRONMENT = "Emanapp-env"
        //SONAR_IP = "54.226.50.200"
        //SONAR_TOKEN = "sqp_a70d878110f7f7a1228416febd75ab0b830a48a1"
    }
    stages {
        stage('Validate') {
            steps {
                sh "chmod +x gradlew"
                sh "./gradlew clean"
            }
        }
        stage('Build') {
            steps {
                sh "./gradlew build"
                sh "./gradlew assemble"

            }
        }
        stage('Test') {
            steps {
                sh "./gradlew test"
            }
        }

        //stage('Quality Scan'){
        //    steps {
        //       sh '''
        //            mvn clean verify sonar:sonar \
        //            -Dsonar.projectKey=java-mshaikh \
        //            -Dsonar.host.url=http://$SONAR_IP \
        //            -Dsonar.login=$SONAR_TOKEN
        //        '''
        //    }
        //}
        stage('Package') {
            steps {
                sh "./gradlew jar"
            }
            post{
                success{
                    archiveArtifacts artifacts: 'build/libs/emanfeah-0.0.1-SNAPSHOT.jar', followSymlinks: false
                }
            }
        }
        stage('Publish artifacts to S3 Bucket') {
            steps {
                sh "aws configure set region us-east-1"
                sh "aws s3 cp ./build/libs/emanfeah-0.0.1-SNAPSHOT.jar s3://$AWS_S3_BUCKET/$ARTIFACT_NAME"
            }
         }
        stage('Deploy') {
            steps {
                sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'
                sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
            }
         }
        

    }
}