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
    SSH_KEY_FILE = "${env.HOME}/.ssh/id_worker"
    IMAGE_DIR_BASE = "${env.WORKSPACE}/image"
    EXPORT_DIR_BASE = "${env.WORKSPACE}/export"
  }

  parameters {
    booleanParam(name: 'useCache', defaultValue: false, description: 'Don\'t force building a fresh image')
    booleanParam(name: 'noTest', defaultValue: false, description: 'Don\'t test the image')
    booleanParam(name: 'needAdminApproval', defaultValue: false, description: 'Wait for admin approval after testing')
    booleanParam(name: 'noRelease', defaultValue: false, description: 'Don\'t release the image')
    string(name: 'logLevel', defaultValue: '3', description: 'Log level')
  }

  stages {
    stage('Pull image tools and image source') {
      steps {
        sh "find . -mindepth 1 -delete"
        dir('image') {
          checkout scm
        }
        dir('tools') {
          checkout([
            $class: 'GitSCM',
            poll: false,
            branches: [[name: 'jrmtb/rewrite']],
            userRemoteConfigs: [[url: "https://github.com/scaleway/image-tools.git"]]
          ])
        }
      }
    }
    stage('Set environment') {
      steps {
        script {
          subdir = sh(
            script: "grep '${env.JOB_NAME}' ${env.IMAGE_DIR_BASE}/images.list | cut -d'|' -f2",
            returnStdout: true
          ).trim()
          env.IMAGE_DIR = "${env.IMAGE_DIR_BASE}/${subdir}"
          if (params.useCache) {
            env.BUILD_OPTS = ""
          } else {
            env.BUILD_OPTS = "--no-cache"
          }
          env.LOG_LEVEL = params.logLevel
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
          sh "make ARCH=x86_64 IMAGE_DIR=${env.IMAGE_DIR} EXPORT_DIR=${env.EXPORT_DIR_BASE}/x86_64 BUILD_OPTS='${env.BUILD_OPTS}' scaleway_image"
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
          sh "make ARCH=arm64 IMAGE_DIR=${env.IMAGE_DIR} EXPORT_DIR=${env.EXPORT_DIR_BASE}/arm64 BUILD_OPTS='${env.BUILD_OPTS}' scaleway_image"
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
          sh "make ARCH=arm IMAGE_DIR=${env.IMAGE_DIR} EXPORT_DIR=${env.EXPORT_DIR_BASE}/arm BUILD_OPTS='${env.BUILD_OPTS}' scaleway_image"
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
            for (Map image : images) {
              sh "make tests IMAGE_DIR=${env.IMAGE_DIR} EXPORT_DIR=${env.EXPORT_DIR_BASE}/${image['arch']} ARCH=${image['arch']} IMAGE_ID=${image['id']} TESTS_DIR=${env.IMAGE_DIR}/tests NO_CLEANUP=${params.needAdminApproval}"
            }
            if (env.needsAdminApproval) {
              input "Confirm that the images are stable ?"
            }
          }
        }
      }
      post {
        always {
          script {
            if (env.needsAdminApproval) {
              dir('tools') {
                for (Map image : images) {
                  sh "scripts/test_images.sh stop ${env.EXPORT_DIR_BASE}/${image['arch']}/${image['id']}.servers"
                }
              }
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

