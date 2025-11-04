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
                docker
                {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps 
            {
                sh '''
                    npm config set strict-ssl false
                    npm install
                    npm ci
                    npm install sharp
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                '''
            }
        }
    }
}
