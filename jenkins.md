   docker run -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
   
   
   ```
      def notifySlack(String buildStatus = 'STARTED') {
          // Build status of null means success.
          buildStatus = buildStatus ?: 'SUCCESS'

          def color

          if (buildStatus == 'STARTED') {
              color = '#CACFD2'
          } else if (buildStatus == 'SUCCESS') {
              color = '#196F3D'
          } else if (buildStatus == 'FAILURE') {
              color = '#922B21'
          } else {
              color = '#F4D03F'
          }

          def msg = "${buildStatus}: `${env.JOB_NAME}` #${env.BUILD_NUMBER}:\n${env.BUILD_URL}"

          slackSend(color: color, message: msg)
      }

      node {
          try {
              notifySlack("STARTED")

              stage('SCM Checkout') {            
                  git branch: 'master', credentialsId: 'dtx-token', url: 'https://github.com/dtx-company/tv-animation.git'
              }        
              stage('Copy files and restart services') {
                 sshPublisher(publishers: [sshPublisherDesc(configName: '10.5.255.163', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''
                  cd flowcode/tv-animation-stag/
                  sudo docker-compose -f docker-compose.stage.yml down || true
                  sudo docker-compose -f docker-compose.stage.yml  up --build -d
                  ''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/flowcode/tv-animation-stag', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '**/*')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
              }
          } catch (e) {
              currentBuild.result = 'FAILURE'
              throw e
          } finally {
              notifySlack(currentBuild.result)
          }
      }
   ```
   
   https://faun.pub/configure-jenkins-ci-cd-with-github-and-deploy-to-server-via-ssh-ade1fdc996fc
