pipeline {
  agent { label 'ubuntu-18.04' }
  options { checkoutToSubdirectory('src') }
  environment {
    CONAN_USER_HOME = "${env.WORKSPACE}"
    REMOTE = "${env.CONAN_REMOTE}"
    PACKAGE = 'conan-tools'
    USER = 'includeos'
    CHAN_LATEST = 'latest'
    CHAN_STABLE = 'stable'
    BINTRAY_CREDS = credentials('devops-includeos-user-pass-bintray')
    SRC = "${env.WORKSPACE}/src"
  }

  stages {
    stage('Setup') {
      steps {
        sh script: "ls -A | grep -v src | xargs rm -r || :", label: "Clean workspace"
        sh script: "conan config install https://github.com/includeos/conan_config.git", label: "conan config install"
      }
    }
    stage('Export') {
      steps {
        sh script: "conan export $SRC $USER/$CHAN_LATEST", label: "Export conan package"
        script { VERSION = sh(script: "conan inspect -a version $SRC | cut -d ' ' -f 2", returnStdout: true).trim() }
      }
    }
    stage('Upload to bintray') {
      parallel {
        stage('Latest release') {
          when { branch 'master' }
          steps {
            upload_package("$CHAN_LATEST")
          }
        }
        stage('Stable release') {
          when { buildingTag() }
          steps {
            sh script: "conan copy --all $PACKAGE/$VERSION@$USER/$CHAN_LATEST $USER/$CHAN_STABLE", label: "Copy to stable channel"
            upload_package("$CHAN_STABLE")
          }
        }
      }
    }
  }
}

def upload_package(String channel) {
  sh script: """
    conan user -p $BINTRAY_CREDS_PSW -r $REMOTE $BINTRAY_CREDS_USR
    conan upload --all -r $REMOTE $PACKAGE/$VERSION@$USER/$channel
  """, label: "Upload to bintray"
}
