pipeline {
    // 스테이지 별로 다른 거
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
        // 레포지토리를 다운로드 받음
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
            }
        }
        
        // aws s3 ?뿉 ?뙆?씪?쓣 ?삱由?
        stage('Deploy Frontend') {
          steps {
            echo 'Deploying Frontend'
            // ?봽濡좏듃?뿏?뱶 ?뵒?젆?넗由ъ쓽 ?젙?쟻?뙆?씪?뱾?쓣 S3 ?뿉 ?삱由?, ?씠 ?쟾?뿉 諛섎뱶?떆 EC2 instance profile ?쓣 ?벑濡앺빐?빞?븿.
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
            // Docker plugin and Docker Pipeline ?몢媛쒕?? 源붿븘?빞 ?궗?슜媛??뒫!
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