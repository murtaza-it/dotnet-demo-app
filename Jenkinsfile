node {

    stage('Checkout') {
        checkout scm
    }

    stage('Test') {
        sh 'docker build --network host -f Dockerfile.test -t todoapp-tests .'
    }

    stage('Build') {
        sh 'docker build -t todoapp-jenkins ./TodoApp'
    }

    stage('Deploy') {
        catchError(buildResult: 'SUCCESS') {
            sh 'docker stop todoapp'
            sh 'docker rm todoapp'
        }

        sh '''
            docker run -d --name todoapp \
              -p 8081:8080 \
              --add-host=host.docker.internal:host-gateway \
              -e ConnectionStrings__TodoDb="Server=host.docker.internal;Port=3306;Database=todo_db;User=todo_usr;Password=letmeinplz;" \
              -e ASPNETCORE_ENVIRONMENT=Development \
              todoapp-jenkins
        '''
    }

    stage('Deploy Production') {
        sshagent(credentials: ['dmz010-ssh']) {
            withCredentials([
                string(credentialsId: 'todo-db-password', variable: 'TODO_DB_PASSWORD')
            ]) {
                sh '''
                    docker save todoapp-jenkins | \
                    ssh -o StrictHostKeyChecking=no murtaza@192.0.2.10 "docker load"

                    ssh -o StrictHostKeyChecking=no murtaza@192.0.2.10 "
                        docker stop todoapp-prod 2>/dev/null || true
                        docker rm todoapp-prod 2>/dev/null || true

                        docker run -d \
                          --name todoapp-prod \
                          -p 8081:8080 \
                          -e ConnectionStrings__TodoDb=\\"Server=192.0.2.20;Port=3306;Database=todoappdb;User=todo_user;Password=${TODO_DB_PASSWORD};\\" \
                          -e ASPNETCORE_ENVIRONMENT=Production \
                          todoapp-jenkins
                    "
                '''
            }
        }
    }
}
