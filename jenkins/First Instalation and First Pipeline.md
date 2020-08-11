1 - Install Jenkins
2 - Install maven plugin
3 - Install GitHub or GitLab plugin
4 - Instal SSH Agent Plugin

5 - In jenkins server you need to install a maven and configure mvn path look like the example on line 45 or configure the MAVEN_HOME in jenkins server

6 - This example, deploy the new version in the tomcat server.

7 - To configure sending the file on the server, you need to configure ssh keys on the jenkins server and copy public key to tomcat server, then, you don't need to specified the user and password to tomcat server

8 - Configure a new Pipeline project with git and specified jenkins file name on the project repository

11 - Generete new ssh keys to configure new credentials for Git in the Jenkins credentials configuration

10 - The jenkins pipeline file must be at the root of your repository. Inside this file you will create your script as in the example below
```
def PROJECT_VERSION = "UNINTIALIZED"
pipeline{
    agent any
    tools {
        maven 'maven3'
      }
    environment {
      PATH = "var/jenkins_home/maven3/bin:$PATH"
    }
    
    stages{
          stage("Git Checkout"){
            steps{
                git credentialsId: 'Jenkins_GitHub', url: 'git@github.com:MarcosVMaciel/inicial-a.git'
            }
          }
          
          stage("Maven Build"){
            steps{
              sh "mvn clean package"
              script {
                     PROJECT_VERSION = readMavenPom().getVersion()                                             
                }
                echo("Project Version=${PROJECT_VERSION}")
            }
          }      
          stage("Deploy tomcat"){
              steps{
                  sshagent(['tomcat-prod']) {                    
                    sh "scp target/*.war root@191.252.192.217:/opt/webapps_inicial/ROOT##${PROJECT_VERSION}.war"                     
                 }
              }
          }   
          stage("Discord Notification"){
            steps{
                 discordSend description: 'Uma nova versão da aplicação esta disponível no servidor de produção', footer: 'Mais informações disponíveis na tag do projeto no GitHub', image: '', link: 'https://loja.iniciala.com.br', result: 'SUCESS', thumbnail: '', title: 'Deploy', webhookURL: 'https://discordapp.com/api/webhooks/740306052939120641/r-3QgeqZinqquGeg6_wTlnG_aVFvdibUGtzyYTG7U8t8n0BYuULuGb6Jys-cILAyonKy'
            }
         }   
     }
}
```
