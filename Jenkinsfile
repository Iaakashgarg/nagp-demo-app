pipeline {
    agent any
    
    environment {
        scannerHome = tool name: 'sonar_scanner_dotnet'
        username = 'aakashgarg'
		registry = 'iaakashgarg/nagp-demo-app'
		properties = null
		docker_port = null
    }
    
    tools {
		msbuild 'MSBuild'
	}

    
    options {
        timestamps()
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '10', numToKeepStr: '20')
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo "Start checking out code"
                git credentialsId: 'GitHub' , url: 'https://github.com/Iaakashgarg/nagp-demo-app.git'
				
            }
        }
		
		stage('Restore Packages') {
			steps {
				echo "Start restoring packages"
				bat "dotnet restore WebApplication4\\WebApplication4.csproj"
			}
		}
        
        stage('Start Sonarqube Analysis') {
            steps {
                echo "Start Sonarqube analysis"
                withSonarQubeEnv('Test_Sonar') {
                    bat "${scannerHome}\\SonarScanner.MSBuild.exe begin /k:sonar-aakashgarg /n:sonar-aakashgarg /v:1.0"
                }
            }
        }
        
        stage('Build Code') {
            steps {
				// Cleans the output of a project
                echo "Clean previous build"
                bat "dotnet clean WebApplication4\\WebApplication4.csproj"
				
				// Builds the project and its dependencies
				echo "Start Building code"
				bat 'dotnet build WebApplication4\\WebApplication4.csproj -c Release -o "WebApplication4/app/build"'
            }
        }
		
		stage('Stop Sonarqube Analysis') {
            steps {
                echo "Stop Sonarqube analysis"
                withSonarQubeEnv('Test_Sonar') {
                    bat "${scannerHome}\\SonarScanner.MSBuild.exe end "
                }
            }
        }
		
		stage('Unit Testing') {
			steps {
				echo "Execute Automated Unit Tests"
				bat "dotnet test WebApplication4-tests\\WebApplication4-tests.csproj -l:trx;LogFileName=WebApplication4TestOutput.xml"
			}
		}
		
		stage('Publish') {
			steps {
				echo "Publish Code"
				bat "dotnet publish -c Release"
			}
		}
		
		stage('Docker Image') {
			steps {
				echo "Create Docker Image"
				bat "dotnet publish -c Release"
				bat "docker build -t i-${username}-master --no-cache -f Dockerfile ."
			}
		}
		
		stage('Push Image to Docker Hub') {
			steps {
				echo "Push Image to Docker Hub"
				bat "docker tag i-${username}-master ${registry}:${BUILD_NUMBER}"
				
				withDockerRegistry([credentialsId: 'DockerHub', url: ""]) {
					bat "docker push ${registry}:${BUILD_NUMBER}"
				}
			}
		}
		
		stage('Docker Deployment') {
			steps {
				echo "Docker Deployment"
				bat "docker run --name c-${username}-master -d -p 7100:80 ${registry}:${BUILD_NUMBER}"
			}
		}
	}
		
		post {
			success {
				echo "Test Report Generation"
				xunit([MSTest(deleteOutputFiles:true, failIfNotNew: true, pattern: 'WebApplication4-tests\\TestResults\\WebApplication4TestOutput.xml', skipNoTestFiles: true)])
			}
		}
        
}