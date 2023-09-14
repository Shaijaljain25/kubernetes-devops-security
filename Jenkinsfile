pipeline {
  agent any
  stages {
    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }
    stage('Unit Tests - JUnit and Jacoco') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }
    stage('SonarQube Analysis') {
      // def mvn = tool 'MavenTool';
      // withSonarQubeEnv() {
      steps {
        sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=numeric -Dsonar.projectName='numeric'"
      }
    }
    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([ credentialsId: "DockerHub", url: "" ]) {
          sh "printenv"
          sh 'docker build -t shaijal/demos:""$GIT_COMMIT"" .'
          sh 'docker push shaijal/demos:""$GIT_COMMIT""'
        }
      }
    }
    stage('k8s-ns') {
      steps {
        withKubeConfig([ credentialsId: "kubeconfig" ]) {
          sh "sed -i 's#replace#shaijal/demos:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml" //comment
        }
      }
    }
  }
}