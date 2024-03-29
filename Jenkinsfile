pipeline {
  agent any
    environment {
      // The following variable is required for a Semgrep Cloud Platform-connected scan:
      SEMGREP_APP_TOKEN = credentials('SEMGREP_APP_TOKEN')
      SEMGREP_BASELINE_REF = "origin/main"
      SEMGREP_BRANCH = "${BRANCH_NAME}"
      SEMGREP_COMMIT = "${GIT_COMMIT}"
      SEMGREP_REPO_URL = env.GIT_URL.replaceFirst(/^(.*).git$/,'$1')
      SEMGREP_PR_ID = "${env.CHANGE_ID}"
    }
    stages {

      stage ('Generate-LockFile') {
        steps {
          withGradle {
            sh './gradlew dependencies --write-locks'
          }
        }
      }
      
      stage('Semgrep-Scan') {
        steps {
                script {
                  echo "Hello from ${env.GIT_BRANCH} branch"
                  echo "Main branch: SEMGREP_BASELINE_REF"
                    if (env.GIT_BRANCH == "origin/main") {
                          sh '''docker pull returntocorp/semgrep && \
                            docker run \
                            -e SEMGREP_APP_TOKEN=$SEMGREP_APP_TOKEN \
                            -v "$(pwd):$(pwd)" --workdir $(pwd) \
                            returntocorp/semgrep semgrep ci '''
                    }  else {
                        sh "git fetch origin +ref/heads/*:refs/remotes/origin/*" 
                        sh '''docker pull returntocorp/semgrep && \
                                  docker run \
                                  -e SEMGREP_APP_TOKEN=$SEMGREP_APP_TOKEN \
                                  -e SEMGREP_BASELINE_REF=$SEMGREP_BASELINE_REF \
                                  -e SEMGREP_REPO_URL=$SEMGREP_REPO_URL \
                                  -e SEMGREP_BRANCH=$SEMGREP_BRANCH \
                                  -e SEMGREP_JOB_URL=$SEMGREP_JOB_URL \
                                  -e SEMGREP_COMMIT=$SEMGREP_COMMIT \
                                  -e SEMGREP_PR_ID=$SEMGREP_PR_ID \
                                  -v "$(pwd):$(pwd)" --workdir $(pwd) \
                                  returntocorp/semgrep semgrep ci'''
                    }
                }
            }
        }
    }
}
