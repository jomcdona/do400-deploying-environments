pipeline {
    agent {
        node {
            label 'maven'
        }
    }
    environment {
        RHT_OCP4_DEV_USER = 'jomcdona'
        DEPLOYMENT_CONFIG_STAGE = 'shopping-cart-stage'
        DEPLOYMENT_CONFIG_PRODUCTION = 'shopping-cart-production'
    }
    stages {
        stage('Tests') {
            steps {
                sh './mvnw clean test'
            }
        }
        stage('Package') {
            steps {
                sh '''
                    ./mvnw package -DskipTests \
                    -Dquarkus.package.type=uber-jar
                '''
                archiveArtifacts 'target/*.jar'
            }
        }
        
        stage('Build Image') {
            environment { QUAY = credentials('QUAY_USER') }

            steps {
               sh '''
                   ./mvnw quarkus:add-extension -Dextensions="kubernetes,container-image-jib"
               '''

               sh '''
                   ./mvnw package -DskipTests -Dquarkus.container-image.build=true -Dquarkus.container-image.registry=quay.io -Dquarkus.container-image.group=$QUAY_USR -Dquarkus.container-image.name=do400-deploying-environments -Dquarkus.container-image.username=$QUAY_USR -Dquarkus.container-image.password=$QUAY_PSW -Dquarkus.container-image.push=true
               '''
            }
        }

        stage('Deployment - Stage') {
           environment {
               APP_NAMESPACE = "${RHT_OCP4_DEV_USER}-shopping-cart-stage"
           }
           steps {
             sh "oc rollout latest dc/${DEPLOYMENT_CONFIG_STAGE} -n ${APP_NAMESPACE}"
           }
        }
        
        stage('Deployment - Production') {
           environment {
               APP_NAMESPACE = "${RHT_OCP4_DEV_USER}-shopping-cart-production"
           }
           steps {
             sh "oc rollout latest dc/${DEPLOYMENT_CONFIG_PRODUCTION} -n ${APP_NAMESPACE}"
           }
        }
    }
}
