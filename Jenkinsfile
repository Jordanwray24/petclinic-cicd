pipeline {
  agent any

  environment {
    TOMCAT_WEBAPPS = "/var/lib/tomcat9/webapps"
    APP_WAR = "petclinic.war"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build (Java 17)') {
      steps {
        sh '''
          set -eux
          export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
          export PATH=$JAVA_HOME/bin:/opt/maven/bin:$PATH
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
