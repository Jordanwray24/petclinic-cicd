pipeline {
  agent any

  environment {
    JAVA_HOME_11 = "/usr/lib/jvm/java-11-openjdk-amd64"
    TOMCAT_WEBAPPS = "/var/lib/tomcat9/webapps"
    APP_WAR = "petclinic.war"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build (Java 11)') {
      steps {
        sh '''
          set -eux
          export JAVA_HOME=${JAVA_HOME_11}
          export PATH=$JAVA_HOME/bin:$PATH
          java -version
          mvn -v
          mvn clean test package
        '''
      }
    }

    stage('Deploy to Tomcat') {
      steps {
        sh '''
          set -eux
          WAR_PATH=$(ls -1 target/*.war | head -n 1)
          echo "Built WAR: $WAR_PATH"
          sudo cp "$WAR_PATH" "${TOMCAT_WEBAPPS}/${APP_WAR}"
          sudo systemctl restart tomcat9
        '''
      }
    }

    stage('Smoke Test') {
      steps {
        sh '''
          set -eux
          sleep 8
          curl -I --max-time 10 http://localhost:8081/${APP_WAR%.*}/ || true
        '''
      }
    }
  }
}
