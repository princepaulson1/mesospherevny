def gitCommit() {
    sh "git rev-parse HEAD > GIT_COMMIT"
    def gitCommit = readFile('GIT_COMMIT').trim()
    sh "rm -f GIT_COMMIT"
    return gitCommit
}

node {
    // Checkout source code from Git
    stage 'Checkout'
    checkout scm

    // Build Docker image
    stage 'Build'
    sh "docker build -t pp023549/mesospherevny:${gitCommit()} ."

    // Log in and push image to GitLab
    stage 'Publish'
    withCredentials(
        [[
            $class: 'UsernamePasswordMultiBinding',
            credentialsId: 'dockerhub-mesosphere',
            passwordVariable: 'DOCKERHUB_PASSWORD',
            usernameVariable: 'DOCKERHUB_USERNAME'
        ]]
    ) {
        sh "docker login -u '${env.DOCKERHUB_USERNAME}' -p '${env.DOCKERHUB_PASSWORD}' -e princepaulson1@gmail.com"
        sh "docker push pp023549/mesospherevny:${gitCommit()}"
    }

// Deploy
    stage 'Deploy'

    marathon(
        url: 'http://marathon.mesos:8080',
        forceUpdate: false,
        credentialsId: 'dcos-token',
        filename: 'marathon.json',
        id: 'nginx-mesosphere',
        docker: "pp023549/mesospherevny:${gitCommit()}".toString()
    )
}