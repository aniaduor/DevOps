pipeline {
    agent any
  
    stages {
        stage('Git') {
           steps {
        script {
            git credentialsId: 'github-credentials', branch: 'master', url: 'https://github.com/aniaduor/DevOps.git'
        }
    }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install -DskipTests' 
            }
        }
        
        stage('Run Tests') {
            steps {
                // Run tests with Maven (JUnit and Mockito)
                sh 'mvn test -DskipTests'
            }
        }
        
         stage(' Build DockerImage') {
            steps {
                
                sh 'docker build -t roudainasaoudi/studentmanagment:latest -f Dockerfile .'
            }
        }
        stage('Push to Docker Hub') {
            steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                sh 'docker push roudainasaoudi/studentmanagment:latest'
            }
        }
    }
        
}
         stage(' Docker Compose') {
            steps {
                
                sh 'docker-compose up -d'
            }
        }
        stage('Jacoco Static Analysis') {
            steps {
                junit 'target/surefire-reports/**/*.xml'
                jacoco()
        }
        }
        stage ('MVN SONARQUBE' ) {
            steps {
               withCredentials([string(credentialsId: 'jenkins-sonar', variable: 'SONAR_TOKEN' )]) {
               sh 'mvn sonar:sonar -Dsonar.login=$SONAR_TOKEN'
            }
            }
        }
        stage('Deploy to Nexus') {
            steps {
                sh 'mvn deploy'
            }
        }
        stage('Prometheus') {
            steps {
               sh 'docker start prometheus '
            }
        }
        stage('Grafana') {
            steps {
               sh 'docker start grafana'
            }
        }
        stage('Terraform') {
            steps {
                sh 'terraform init'  
                sh 'terraform apply -auto-approve'
            }
        }
    }
       
    post {
    success {
        emailext(
            subject: "Build Success: ${currentBuild.fullDisplayName}",
            body: "Le pipeline a réussi. Voir les détails du build ici: ${env.BUILD_URL}",
            to: 'minar.idouas6@gmail.com'
        )
    }
    failure {
        emailext(
            subject: "Build Failed: ${currentBuild.fullDisplayName}",
            body: "Le pipeline a échoué. Voir les détails du build ici: ${env.BUILD_URL}",
            to: 'minar.idouas6@gmail.com'
        )
    }
    }
    }

    






