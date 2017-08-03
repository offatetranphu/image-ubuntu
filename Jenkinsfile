def images = []

pipeline {
  agent {
    label 'worker&&docker'
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '1'))
  }

  triggers {
    pollSCM("H */2 * * *")
    cron('H H * * H')
  }

  environment {
    SSH_KEY_FILE = "${env.HOME}/.ssh/id_worker.pub"
    IMAGE_DIR = "${env.WORKSPACE}/image/16.04"
    EXPORT_DIR_BASE = "${env.WORKSPACE}/export"
  }

  parameters {
    booleanParam(name: 'noTest', defaultValue: false, description: 'Don\'t test the image')
    booleanParam(name: 'needAdminApproval', defaultValue: false, description: 'Wait for admin approval after testing')
    booleanParam(name: 'noRelease', defaultValue: false, description: 'Don\'t release the image')
  }

  stages {
    stage('Pull image tools and image source') {
      steps {
        dir('tools') {
          checkout([
            $class: 'GitSCM',
            poll: false,
            branches: [[name: 'jrmtb/rewrite']],
            userRemoteConfigs: [[url: "https://github.com/scaleway/image-tools.git"]]
          ])
        }
        dir('image') {
          checkout scm
        }
      }
    }
    stage('Log into Scaleway') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'scw-test-orga-token', usernameVariable: 'SCW_ORGANIZATION', passwordVariable: 'SCW_TOKEN')]) {
          sh "scw login -o '${SCW_ORGANIZATION}' -t '${SCW_TOKEN}' -s >/dev/null 2>&1"
        }
      }
    }
    stage('Create image on Scaleway: x86_64') {
      steps {
        dir('tools') {
          sh "make ARCH=x86_64 IMAGE_DIR=${IMAGE_DIR} EXPORT_DIR=${env.EXPORT_DIR_BASE}/x86_64 BUILD_OPTS='--no-cache' scaleway_image"
          script {
            imageId = readFile("${env.EXPORT_DIR_BASE}/x86_64/image.id").trim()
            images.add([arch: "x86_64", id: imageId])
          }
        }
      }
    }
    stage('Create image on Scaleway: arm64') {
      steps {
        dir('tools') {
          sh "make ARCH=arm64 IMAGE_DIR=${IMAGE_DIR} EXPORT_DIR=${env.EXPORT_DIR_BASE}/arm64 BUILD_OPTS='--no-cache' scaleway_image"
          script {
            imageId = readFile("${env.EXPORT_DIR_BASE}/arm64/image.id").trim()
            images.add([arch: "arm64", id: imageId])
          }
        }
      }
    }
    stage('Create image on Scaleway: arm') {
      steps {
        dir('tools') {
          sh "make ARCH=arm IMAGE_DIR=${IMAGE_DIR} EXPORT_DIR=${env.EXPORT_DIR_BASE}/arm BUILD_OPTS='--no-cache' scaleway_image"
          script {
            imageId = readFile("${env.EXPORT_DIR_BASE}/arm/image.id").trim()
            images.add([arch: "arm", id: imageId])
          }
        }
      }
    }
    stage('Test the images') {
      when {
        expression { params.noTest == false }
      }
      steps {
        dir('tools') {
          script {
            if (params.createImage == false) {
              env.imageId = params.imageId
            }
          }
          withCredentials([usernamePassword(credentialsId: 'scw-test-orga-token', usernameVariable: 'SCW_ORGANIZATION', passwordVariable: 'SCW_TOKEN')]) {
            sh "./test_image.sh start ${params.arch} ${env.imageId} servers_list"
          }
          script {
            serverIds = readFile('servers_list').trim()
          }
          echo "The following servers have been booted and passed basic checks:\nTYPE NAME ID\n${serverIds}"
          script {
            if (params.needAdminApproval) {
              emailext(
                to: "jtamba@online.net",
                subject: "Image test from image-release #${env.BUILD_NUMBER} needs admin approval",
                body: """<p>A new version of an image is being tested. The following servers have been booted and passed basic checks:\nTYPE NAME ID\n${serverIds}. You can ssh to it and do some manual checks.</p>
                <p>If the image is fit for release, you can <a href="${env.JENKINS_URL}/blue/organizations/jenkins/image-release/detail/image-release/${env.BUILD_NUMBER}">go to the pipeline</a> to confirm the build or otherwise abort it.</p>
                """
              )
              input message: "You can run some manual checks on the booted server(s). Confirm that the image is stable ?", ok: 'Confirm'
            }
          }
        }
      }
      post {
        always {
          dir('test-and-release') {
            withCredentials([usernamePassword(credentialsId: 'scw-test-orga-token', usernameVariable: 'SCW_ORGANIZATION', passwordVariable: 'SCW_TOKEN')]) {
              sh "./test_image.sh stop servers_list"
            }
          }
        }
      }
    }
    stage('Release the image') {
      when {
        expression { params.noRelease == false }
      }
      steps {
        script {
          marketplace_id = sh(
            script: "curl https://api-marketplace.scaleway.com/images | jq -r '.images[] | select(.name | contains(\"Ubuntu Xenial\")) | .id'",
            returnStdout: true
          ).trim()
          message = [
            type: image,
            data: [
              marketplace_id: marketplace_id,
              versions: images
            ]
          ]
          writeJSON(file: 'release.json', json: message)
          json_string = readFile(file: 'release.json')
          versionId = input(
            message: "${json_string}",
            parameters: [string(name: 'version_id', description: 'ID of the new image version')]
          )
        }
        echo "Created new marketplace version of image: ${versionId}"
      }
    }
  }

  post {
    always {
      deleteDir()
    }
  }
}

