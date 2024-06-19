<img src="https://github.com/Harsh971/JPetStore-CICD/blob/master/architecture.gif"></img>

# JPetStore CICD

JPetStore 6 is a full web application built on top of MyBatis 3, Spring 5 and Stripes.

Essential Jenkins Plugins
----------

- `jdk` : Eclipse Temurin installer
- `sonar` : SonarQube Scanner
- `owasp` : OWASP Dependency-Check
- `docker` : Docker, Docker Pipeline

## Configuration of Plugins

### Configure Plugins : 
- Manage Jenkins > Global Tool Configuration

### JDK Installation
- Name : jdk17
- Install Automatically > Install from adoptium.net > Version 17

### SonarQube Scanner installations
- Name : sonar-scanner
- Install Automatically > Version (Default)

### Maven installations
- Name : maven3
- Install Automatically > Version 3.6.0

### Dependency-Check installations
- Name : DP
- Install Automatically > Install from github.com > dependency-check 6.5.1

### Docker installations
- Name : docker
- Install Automatically > Version (latest)


## Initiating SonarQube
- for the stage named : "SonarQube Analysis" we need to have sonarqube running so we will do it using Docker
Command : <br> `docker run -d --name sonar -p 9000:9000 sonarqube:lts-community`

## Jenkins File :

```
pipeline {
    agent any
	
	tools{
		jdk 'jdk17'
		maven 'maven3'
	}
	
	environment {
		SCANNER_HOME= tool 'sonar-scanner'
	}

    stages {
        stage('Git Checkout') {
            steps {
                git changelog: false, poll: false, url: 'https://github.com/Harsh971/JPetStore-CICD.git'
            }
        }
		stage('Maven Compile') {
            steps {
                sh "mvn clean compile"
            }
        }
		stage('SonarQube Analysis') {
            steps {
                sh ''' $SCANNER_HOME/bin/sonar-scanner \
				-Dsonar.host.url=http://<IP>:9000 \
				-Dsonar.login=<SonarQube TOKEN> \
				-Dsonar.projectName=petstore \
				-Dsonar.java.binaries=. \
				-Dsonar.projectKey=petstore '''
            }
        }
		stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
				dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Maven Build') {
            steps {
                sh "mvn clean install"
            }
        }
		stage('Build and Push Docker Image') {
            steps {
                script{
					withDockerRegistry(credentialsId: 'DockerHub', toolName: 'docker') {
						sh "docker build -t petstore ."
						sh "docker tag petstore harsh0369/petstore:latest"
						sh "docker push harsh0369/petstore:latest"
					}
				}				
            }
        }
		stage('Deploy Docker Image') {
            steps {
                script{
					withDockerRegistry(credentialsId: 'DockerHub', toolName: 'docker') {
						sh "docker run -d -p 8081:8080 harsh0369/petstore:latest"
					}
				}				
            }
        }
    }
}

```

## Output Snapshots : 
<img src="https://github.com/Harsh971/JPetStore-CICD/blob/master/output1.png"></img>          
<br>
<img src="https://github.com/Harsh971/JPetStore-CICD/blob/master/output2.png"></img>

## Source Credits :
Original Source Code : [Click Here](https://github.com/mybatis/jpetstore-6)
Tutorial Reference : [Click Here](https://www.youtube.com/watch?v=jwGe59RnxtA&list=PPSV)

## Feedback

Your feedback is valuable! If you have suggestions for improving existing content or ideas for new additions, please open an issue or reach out to the repository maintainers. If you have any other feedbacks, you can reach out to us at harsh.thakkar0369@gmail.com


## Connect with Me
<p>

 <a href="https://twitter.com/HarshThakkar971" target="blank"><img align="center" src="https://img.freepik.com/premium-vector/vector-new-twitter-x-white-logo-black-background_744381-866.jpg" alt="HarshThakkar971" height="40" width="50" /></a>
  &nbsp;&nbsp;
  	<a href="https://linkedin.com/in/harsh-thakkar-7764bb1a4" target="blank"><img align="center" src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/ca/LinkedIn_logo_initials.png/800px-LinkedIn_logo_initials.png" alt="harsh-thakkar-7764bb1a4" height="40" width="40" /></a>
  &nbsp;&nbsp;
 <a href="https://instagram.com/harsh_thakkar09" target="blank"><img align="center" src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e7/Instagram_logo_2016.svg/768px-Instagram_logo_2016.svg.png" alt="harsh_thakkar09" height="40" width="40" /></a>
</p>


