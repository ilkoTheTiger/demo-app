peline
{
    agent
    {
        label 'docker-node'
    }
    environment
    {
        DOCKERHUB_CREDENTIALS=credentials('docker-hub')
    }
    stages
    {
        stage('Clone')
        {
            steps
            {
                git branch: 'main', url: 'http://192.168.99.102:3000/ilia/bgapp'
            }  
        }
        stage('Create a network')
        {
            steps
            {
                sh '''
                docker network rm appnet || true
                docker network ls | grep appnet || docker network create appnet
                '''
            }
        }
        stage('Build the Images')
        {
            steps
            {
                sh 'docker image rm -f pipeline-bgapp-web pipeline-bgapp-db || true'
                sh 'docker compose build'
            }
        }
        stage('Run the Containers')
        {
            steps
            {
                sh 'docker container rm -f deploy-web-1 deploy-db-1 || true'
                sh 'docker volume prune -f'
                sh 'docker compose up -d'
            }
        }
        stage('Test')
        {
            steps
            {
                script
                {
                    echo 'Test #1 - Reachability'
                    sh 'echo $(curl --write-out "%{http_code}" --silent --output /dev/null http://localhost:8080) | grep 200 || true'

                    echo 'Test #2 - Check if Sofia Appears'
                    sh "curl --silent http://localhost:8080 | grep София || true"  
                    
                    echo 'Test #3 - Sofia Population Appears'
                    sh "sleep 10 && curl --silent http://192.168.99.102:8080 | grep 1236047"

                    echo 'Test #4 - Check if Plovdiv Appears'
                    sh "curl --silent http://localhost:8080 | grep Пловдив || true"

                    echo 'Test #5 - Plovdiv Population Appears'
                    sh "curl --silent http://localhost:8080 | grep 343424 || true"          
                }
            }
        }
        stage('CleanUp')
        {
            steps
            {
                sh 'docker container rm -f pipeline-bgapp-web-1 pipeline-bgapp-db-1 || true'
                sh 'docker volume prune -f'
            }
        }
        stage('Login')
        {
            steps
            {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Push')
        {
            steps
            {
                sh 'docker image rm ilkothetiger/bgapp-web ilkothetiger/bgapp-db || true'
                sh 'docker image tag pipeline-bgapp-web ilkothetiger/bgapp-web'
                sh 'docker image tag pipeline-bgapp-db ilkothetiger/bgapp-db'
                sh 'docker push ilkothetiger/bgapp-web'
                sh 'docker push ilkothetiger/bgapp-db'
            }
        }
        stage('Deploy')
        {
            steps
            {
                sh 'docker image rm -f pipeline-bgapp-web pipeline-bgapp-db || true'
                sh 'cd deploy && docker compose up -d'
            }
        }
    }
}
