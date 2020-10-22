pipeline {
    // �������� ���� �ٸ� ��
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
        // �������丮�� �ٿ�ε� ����
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
        
        // aws s3 ?�� ?��?��?�� ?���?
        stage('Deploy Frontend') {
          steps {
            echo 'Deploying Frontend'
            // ?��론트?��?�� ?��?��?��리의 ?��?��?��?��?��?�� S3 ?�� ?���?, ?�� ?��?�� 반드?�� EC2 instance profile ?�� ?��록해?��?��.
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
            // Docker plugin and Docker Pipeline ?��개�?? 깔아?�� ?��?���??��!
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