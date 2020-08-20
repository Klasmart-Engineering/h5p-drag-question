env.APP_BRANCH = "dev"
env.H5P_LIBRARY = "h5p-drag-question"
env.H5P_TARGET_DIR = env.H5P_LIBRARY

env.GIT_REPOSITORY_URL_APP = "git@bitbucket.org:calmisland/${env.H5P_LIBRARY}.git"

// for creadential 
env.GIT_CREDENTIAL_ID = 'ssh-private-key-hoon'

podTemplate(containers: [
    containerTemplate(
        name: 'node',
        image: 'node:12.18',
        command: 'cat',
        ttyEnabled: true)
  ],
    volumes: [
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')
    ]){

    node(POD_LABEL) {
      // stage('test'){
      //     git credentialsId: env.GIT_CREDENTIAL_ID, url: env.GIT_REPOSITORY_URL_APP, branch: env.APP_BRANCH
      // }

      stage('Checkout Entire Libraries') {
        container('node') {
          sh "npm install -g h5p"
          sshagent([env.GIT_CREDENTIAL_ID]) {
            try{
              sh 'ssh -o StrictHostKeyChecking=no git@github.com'
            } catch (err) {
              echo err.getMessage()
            }

            sh "h5p get ${env.H5P_LIBRARY}"
            sh "rm -rf ${env.H5P_LIBRARY}"
          }
        }
      }

      stage('Checkout Target') {
        container('node') {
          sshagent([env.GIT_CREDENTIAL_ID]) {
            try{
              sh 'ssh -o StrictHostKeyChecking=no git@bitbucket.org'
            } catch (err) {
              echo err.getMessage()
            }

            sh "git clone ${env.GIT_REPOSITORY_URL_APP} ${env.H5P_LIBRARY}"

            dir("${env.H5P_LIBRARY}"){
              sh "git checkout ${env.APP_BRANCH}"
              env.APP_GIT_REV = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
              echo "APP_GIT_REV: ${env.APP_GIT_REV}"
            }
          }
        }
      }

      stage('Build Target') {
        container('node') {
          dir("${env.H5P_LIBRARY}"){
              sh "npm install"
              sh "npm run build"
          }
        }
      }

      stage('Packing Target') {
        container('node') {
          sh "h5p pack ${env.H5P_LIBRARY}"
          sh "mkdir -p dist"
          sh "mv libraries.h5p dist/libraries.zip"
          dir("dist"){
            sh "unzip libraries.zip"
          }
        }
      }

      stage('Extract and Upload to S3') {
        container('node') {
          withAWS(credentials:"${AWS_CREDENTIAL_ID}") {
            s3Upload(bucket:"calmisland-shared-h5p", path:"${env.APP_BRANCH}/libraries", includePathPattern:'**/*', workingDir:'dist')
          }
        }
      }






        // // stage('APP: custom iframe-resizer-react') {
        // //     dir('live/node_modules/iframe-resizer-react') {
        // //       container('node') {
        // //         sh "rm -rf *"
        // //         sh "git clone https://github.com/lgiraudel/iframe-resizer-react.git ."
        // //         sh "git checkout patch-1"
        // //         sh "npm i"
        // //         sh "npm run build"
        // //       }
        // //     }
        // // }


        // stage('APP: npm build') {
        //     dir('live') {
        //       container('node') {
        //         sh "npm run build:${env.APP_BRANCH}"
        //         sh "mv dist/record*.js dist/record.js"
        //       }
        //     }
        // }


        // stage('Upload to S3') {
        //   withAWS(credentials:"${AWS_CREDENTIAL_ID}") {
        //     s3Upload(bucket:"calmisland-kidsloop-static", path:'dev/live/', includePathPattern:'**/*', workingDir:'live/dist')
        //     cfInvalidate(distribution:"${env.CLOUDFRONT_ID}", paths:['/*'], waitForCompletion: true)
        //   }
        // }
        
        // stage('Push Docker Image') {
        //     container('docker') {
        //         docker.withRegistry("https://${env.ECR_REPO_HOST}",  "ecr:ap-northeast-2:${env.ECR_CREDENTIAL_ID}") {
        //             sh "docker tag ${ECR_REPO_NAME}:latest ${ECR_REPO_HOST}/${ECR_REPO_NAME}:${APP_BRANCH}"
        //             sh "docker tag ${ECR_REPO_NAME}:latest ${ECR_REPO_HOST}/${ECR_REPO_NAME}:${APP_GIT_REV}"
        //             sh "docker push ${ECR_REPO_HOST}/${ECR_REPO_NAME}"
        //         }
        //     }
        // }

        // stage('Checkout') {
        //     env.APP_GIT_REV = env.git_hash.substring(0, 7)
        //     println env.APP_GIT_REV
        // }

        // stage('Kubernates Deploy'){
        //     withCredentials([string(credentialsId: env.ARGOCD_AUTH_TOKEN, variable: 'ARGOCD_AUTH_TOKEN')]) { //set SECRET with the credential content
        //         sh "curl -sSL -o ./argocd https://${ARGOCD_SERVER}/download/argocd-linux-amd64"
        //         sh "chmod 755 ./argocd"
        //         sh "./argocd app set ${ARGOCD_APP_NAME} --grpc-web -p targetTag=${APP_GIT_REV}"
        //     }            
        // }
    }
}



