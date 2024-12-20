pipeline {
    agent any

    environment {
        // Replace 'SonarQubeServer' with the name of your SonarQube server configured in Jenkins
        SONAR_HOME = tool 'sonar-scanner' 
        MAVEN_HOME = tool 'maven'
        LIZARD_PATH = '/home/ubuntu/lizard-env/bin/lizard'
        containerName = 'java_app'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Checkout the code from your Git repository
                git url: 'https://github.com/MahimaRajput/scoreme-cicd', branch: 'main'
            }
        }
        stage('Build and Test') {
            steps {
                // Run Maven build and tests
                sh "'${MAVEN_HOME}/bin/mvn' clean package"
                sh 'mkdir -p pkg'
                sh 'mv target/demo.war pkg/demo.war'    
                sh "'${MAVEN_HOME}/bin/mvn' clean test"

            }
        }
        stage('JaCoCo coverage report') {
            steps {
                // Publish JaCoCo coverage report
                jacoco execPattern: '**/target/*.exec', classPattern: '**/target/classes', sourcePattern: '**/src/main/java'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                // Run SonarQube analysis
                withCredentials([string(credentialsId: 'sonar', variable: 'sonar')]){
                    withSonarQubeEnv("sonar-server") {
                        sh '''
                            $SONAR_HOME/bin/sonar-scanner \
                            -Dsonar.projectKey=testcicd \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=${SONAR_HOST_URL}  \
                            -Dsonar.login=${SONAR_AUTH_TOKEN}
                        '''
                    }
                }
            }
        }
        stage("Sonar Quality Gate Scan"){
            steps{
                timeout(time: 2, unit: "MINUTES"){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Run Lizard') {
            steps {
                script {
                    // Run Lizard to analyze code complexity
                    sh "${LIZARD_PATH} . --output report.txt"
                }
            }
        }
        stage('Owasp Dependency'){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'owasp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'          
           }
        }
        stage('Docker Build & Push') 
		{
			steps 
			{
			    script
			    {
                  withDockerRegistry([url: 'https://index.docker.io/v1/', credentialsId: 'Docker_id']){
                        sh "docker build -t mahimarajput26/scorerepo:${BUILD_NUMBER} ."
				        sh "docker build -t mahimarajput26/scorerepo:latest ."
					    sh "docker push mahimarajput26/scorerepo:${BUILD_NUMBER}"
					    sh "docker push mahimarajput26/scorerepo:latest"	
					}
			    }
			}
		}
		stage('Deploy to container'){
            steps{
                sh "docker ps -q -f name=${containerName} | grep -q . && docker stop ${containerName} && docker rm ${containerName} || true"
                sh 'docker run -d --name java_app -p 8000:8000  mahimarajput26/scorerepo:latest '
            }
        }
    }
    
    post {
        success {
            // Send success email if build succeeds
            mail to: 'fourthmahima@gmail.com',
                 subject: "Build Successful: ${env.JOB_NAME}",
                 body: "The build was successful: ${currentBuild.currentResult}- ${BUILD_URL}"
                 
        }
        failure {
            // Send failure email if build fails
            mail to: 'fourthmahima@gmail.com',
                 subject: "Build Failed: ${env.JOB_NAME}",
                 body: "The build failed: ${currentBuild.currentResult}. Please check the Jenkins logs for details."
                 
        }
    }
}
