node {
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
}