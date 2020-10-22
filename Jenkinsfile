pipeline {
    // Ω∫≈◊¿Ã¡ˆ ∫∞∑Œ ¥Ÿ∏• ∞≈
    agent any

    triggers {
        pollSCM('*/3 * * * *')
    }

    environment {
      AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')
      AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
      AWS_DEFAULT_REGION = 'ap-northeast-2'
      HOME = '.' // Avoid npm root owned
    }

    stages {
        // ∑π∆˜¡ˆ≈‰∏Æ∏¶ ¥ŸøÓ∑ŒµÂ πﬁ¿Ω
        stage('Prepare') {
            agent any
            
            steps {
                // echo "Lets start Long Journey! ENV: ${ENV}"
                echo 'Clonning Repository'

                git url: 'https://github.com/junhyeokk/jenkins-test.git',
                    branch: 'master',
                    credentialsId: 'gitTestToken'
            }

            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    echo 'Successfully Cloned Repository'
                }

                always: {
                  echo "i tried..."
                }

                cleanup {
                  echo "after all other post condition"
                }
            }
        }
        
        // aws s3 ?óê ?åå?ùº?ùÑ ?ò¨Î¶?
        stage('Deploy Frontend') {
          steps {
            echo 'Deploying Frontend'
            // ?îÑÎ°†Ìä∏?óî?ìú ?îî?†â?Ü†Î¶¨Ïùò ?†ï?†Å?åå?ùº?ì§?ùÑ S3 ?óê ?ò¨Î¶?, ?ù¥ ?†Ñ?óê Î∞òÎìú?ãú EC2 instance profile ?ùÑ ?ì±Î°ùÌï¥?ïº?ï®.
            dir ('./website'){
                sh '''
                aws s3 sync ./ s3://junhyeokk-jenkins-test
                '''
            }
          }

          post {
              // If Maven was able to run the tests, even if some of the test
              // failed, record the test results and archive the jar file.
              success {
                  echo 'Successfully Cloned Repository'

                  mail  to: 'chlwnsgur205@gmail.com',
                        subject: "Deploy Frontend Success",
                        body: "Successfully deployed frontend!"
                  
              }
              failure {
                  echo 'I failed :('

                  mail  to: 'chlwnsgur205@gmail.com',
                        subject: "Failed Pipelinee",
                        body: "Something is wrong with deploy frontend"
              }
          }
        }
        
        stage('Lint Backend') {
            // Docker plugin and Docker Pipeline ?ëêÍ∞úÎ?? ÍπîÏïÑ?ïº ?Ç¨?ö©Í∞??ä•!
            agent {
              docker {
                image 'node:latest'
              }
            }
            
            steps {
              dir ('./server'){
                  sh '''
                  npm install&&
                  npm run lint
                  '''
              }
            }
        }
        
        stage('Test Backend') {
          agent {
            docker {
              image 'node:latest'
            }
          }
          steps {
            echo 'Test Backend'

            dir ('./server'){
                sh '''
                npm install
                npm run test
                '''
            }
          }
        }
        
        stage('Bulid Backend') {
          agent any
          steps {
            echo 'Build Backend'

            dir ('./server'){
                sh """
                docker build . -t server --build-arg env=${PROD}
                """
            }
          }

          post {
            failure {
              error 'This pipeline stops here...'
            }
          }
        }
        
        stage('Deploy Backend') {
          agent any

          steps {
            echo 'Build Backend'

            dir ('./server'){
              // docker rm -f $(docker ps -aq)
                sh '''
                docker run -p 80:80 -d server
                '''
            }
          }

          post {
            success {
              mail  to: 'chlwnsgur205@gmail.com',
                    subject: "Deploy Success",
                    body: "Successfully deployed!"
                  
            }
          }
        }
    }
}

// https://github.com/frontalnh/temp