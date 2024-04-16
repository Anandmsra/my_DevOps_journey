pipeline{
    agent any
    environment{
        SONAR_HOME = tool "Sonar"
    }
    stages{
        stage("code"){
            steps{
                git url: "https://github.com/Anandmsra/node-todo-cicd.git", branch: "master"
                echo "code clone successfully"
            }
        }
        
        stage("SonarQube Analysis"){
            steps{
               withSonarQubeEnv("Sonar"){
                   sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=nodetodo -Dsonar.projectKey=nodetodo"
               }
            }
        }
        
        stage("SonarQube Quality Gates"){
            steps{
               timeout(time: 30, unit: "MINUTES"){
                   waitForQualityGate abortPipeline: true
               }
            }
        }
        
        stage("OWASP"){
            steps{
               dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWASP'
               dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
               }
            }
            
            stage("build and test"){
            steps{
                sh "docker build . -t node-app-batch6:latest"
                echo "code build successfully"
            }
            }
        
        stage("Trivy"){
            steps{
               sh "trivy image node-app-batch6"
               }
            }
        
        stage("image push to dockerhub"){
            steps{
                withCredentials([usernamePassword(credentialsId: "dockerHubCreds", usernameVariable: "dockeruser", passwordVariable: "dockerpass")])
                {
                sh "docker login -u ${env.dockeruser} -p ${env.dockerpass}"
                sh "docker tag node-app-batch6:latest ${env.dockeruser}/node-app-batch6:latest"
                sh "docker push ${env.dockeruser}/node-app-batch6:latest"
                }
            }
        }
        
        stage("deploy"){
            steps{
                sh "docker-compose down && docker-compose up -d"
            }
        }
    }
}
