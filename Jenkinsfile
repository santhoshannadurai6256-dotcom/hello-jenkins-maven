pipeline {
  agent any

  tools {
    jdk 'JDK17'
    maven 'Maven3'
  }

  environment {
    WILDFLY_HOST = '54.221.57.130'   // e.g. 192.168.1.100
    WILDFLY_PORT = '9991'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build & Test') {
      steps {
        sh 'mvn -B -U clean verify'
      }
      post {
        always {
          junit '**/target/surefire-reports/*.xml'
          archiveArtifacts artifacts: '**/target/*.war, **/target/*.jar', fingerprint: true
        }
      }
    }

    stage('Deploy to WildFly') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'WILDFLY_MGMT',
                                          usernameVariable: 'WF_USER',
                                          passwordVariable: 'WF_PASS')]) {
          // Debug to confirm env vars
          sh '''
            echo "== Deployment debug info =="
            echo "WildFly Host: ${WILDFLY_HOST}"
            echo "WildFly Port: ${WILDFLY_PORT}"
            echo "WildFly User: ${WF_USER}"
          '''

          // Deploy to WildFly
          sh '''
            mvn -B wildfly:deploy \
              -Dwildfly.hostname="${WILDFLY_HOST}" \
              -Dwildfly.port="${WILDFLY_PORT}" \
              -Dwildfly.username="${WF_USER}" \
              -Dwildfly.password="${WF_PASS}" \
              -Ddeploy.force=true
          '''
        }
      }
    }
  }

  post {
    success { echo '✅ Build & deploy complete.' }
    failure { echo '❌ Build or deploy failed. Check logs.' }
  }
}
