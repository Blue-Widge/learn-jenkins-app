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
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps 
            {
                sh '''
                    # Update Alpine and install required packages for sharp + SSL
                    apk add --no-cache ca-certificates build-base vips-dev python3 make g++

                    # (Optional) update certs just in case
                    update-ca-certificates

                    # Configure npm
                    npm config set strict-ssl false

                    # Install Netlify CLI
                    npm install netlify-cli@20.1.1

                    # Check installed version
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
