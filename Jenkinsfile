pipeline {
    agent none
    stages {
        stage('Test') {
            agent {
                docker { image 'obraun/vss-jenkins' }
            }
            steps {
                sh 'go test ./... -coverprofile cover.out -v'
                sh 'go test ./... -bench=.'

                // Check that code coverage was > 90 %
                sh '''
                    LAST_LINE=$(go tool cover -func cover.out | tail -1);
                    REVERSED=$(echo $LAST_LINE | rev);
                    LAST_PART=$(echo $REVERSED | cut -d ' ' -f 1);
                    COVERAGE=$(echo $LAST_PART | rev);
                    COVERAGE=${COVERAGE::-1};
                    CONDITION=$(echo "$COVERAGE >= 0.9" | bc)
                    echo $CONDITION
                    if [ $CONDITION -eq 1 ]; then
                        echo "Great! Code coverage is sufficient!"
                    else
                        echo "Code coverage insufficient, we need at least 90%!"
                    fi
                '''
            }
        }
        stage('Lint') {
            agent {
                docker { image 'obraun/vss-jenkins' }
            }   
            steps {
                sh 'golangci-lint run --enable-all --disable goimports'
            }
        }
        stage('Build Docker Image') {
            agent {
                label 'master'
            }
            steps {
                sh "docker-build-and-push -b ${BRANCH_NAME}"
            }
        }
    }
    post {
        changed {
            script {
                if (currentBuild.currentResult == 'FAILURE') { // Other values: SUCCESS, UNSTABLE
                    // Send an email only if the build status has changed from green/unstable to red
                    emailext subject: '$DEFAULT_SUBJECT',
                        body: '$DEFAULT_CONTENT',
                        recipientProviders: [
                            [$class: 'DevelopersRecipientProvider']
                        ], 
                        replyTo: '$DEFAULT_REPLYTO'
                }
            }
        }
    }
}
