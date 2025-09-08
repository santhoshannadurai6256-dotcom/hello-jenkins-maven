pipeline {
  agent any

  tools {
    jdk 'JDK17'
    maven 'Maven3'
  }

  environment {
    WILDFLY_HOST = 'your-wildfly-server'
    WILDFLY_PORT = '9990'
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
      when { branch 'main' }
      steps {
        withCredentials([usernamePassword(credentialsId: 'WILDFLY_MGMT',
                                          usernameVariable: 'WF_USER',
                                          passwordVariable: 'WF_PASS')]) {
          sh '''
            mvn -B wildfly:deploy               -Dwildfly.hostname="${WILDFLY_HOST}"               -Dwildfly.port="${WILDFLY_PORT}"               -Dwildfly.username="${WF_USER}"               -Dwildfly.password="${WF_PASS}"               -Ddeploy.force=true
          '''
        }
      }
    }
  }

  post {
    success { echo '✅ Deployed successfully!' }
    failure { echo '❌ Something went wrong. Check the console output.' }
  }
}
