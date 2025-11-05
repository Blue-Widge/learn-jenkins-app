pipeline {
    agent any

    stages 
    {
        stage('Build') 
        {
            agent
            {
                docker
                {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps 
            {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }
        stage('CreateImages')
        {
            parallel
            {
                stage('NetlifyImage')
                {
                    agent
                    {
                        docker
                        {
                            image 'dind:latest'
                            reuseNode true
                        }

                    }
                    steps
                    {
                        sh 'docker build -t node-netlify:local -f Dockerfile.netlifyImage ./myImages/.'
                    }
                }
            }
        }
        stage("AllTests")
        {
            parallel
            {
                stage('UnitTest')
                {
                    agent
                    {
                        docker
                        {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps
                    {
                        sh '''
                            echo "Test stage"
                            grep "" ./build/index.html 1> /dev/null
                            npm test
                        '''
                    }
                    
                    post
                    {
                        always
                        {
                            junit 'JunitTest-results/junit.xml'
                        }
                    }
                }

                stage('End2EndTest')
                {
                    agent
                    {
                        docker
                        {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            //args '-u root:root' Bad idea
                        }
                    }
                    steps
                    {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build&
                            sleep 20
                            npx playwright test
                        '''
                    }
                }
            }
        }
        stage('Deploy') 
        {
            agent 
            {
                docker {
                    image 'myImages/netlifyImage:latest'
                    reuseNode true
                }
            }
            steps 
            {
                sh '''
                    node_modules/.bin/netlify --version
                '''
            }
            post 
            {
                failure 
                {
                    sh 'cat /home/node/.npm/_logs/*-debug-0.log || true'
                }
            }
        }

    }
}
