node {
  checkout scm
  env.PATH = "${tool 'Maven3'}/bin:${env.PATH}"
  stage('Build') {
    dir('webapp') {
      sh 'mvn clean package -DskipTests'
    }
  }

  stage('Test') {
    dir('webapp') {
      sh 'mvn -f pom.xml exec:java -DskipTests'
      sh 'mvn -f pom.xml test'
    }
  }

  stage('Sonar'){
    dir('webapp'){
      sh 'mvn verify sonar:sonar'
    }
  }

  stage('Create Docker Image') {
    dir('webapp') {
      docker.build("adriell/docker-jenkins-pipeline:${env.BUILD_NUMBER}")
      docker.build("adriell/docker-jenkins-pipeline:latest")
    }
  }

  stage ('Run Application') {
    try {
      sh "DB=`docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' db`"
      sh "docker run -e DB_URI=$DB adriell/docker-jenkins-pipeline:${env.BUILD_NUMBER}"


    } catch (error) {
    } finally {
      // Stop and remove database container here
      sh 'docker rm -f db'
      //sh 'docker-compose rm db'
    }
  }

  stage('Run Tests') {
    try {
      dir('webapp') {
        sh "mvn test"
        docker.build("arungupta/docker-jenkins-pipeline:${env.BUILD_NUMBER}").push()
      }
    } catch (error) {

    } finally {
      junit '**/target/surefire-reports/*.xml'
    }
  }
}
