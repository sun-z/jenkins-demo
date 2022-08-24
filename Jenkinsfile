node('bypay-jnlp') {
    stage('Clone') {
        echo "1.Prepare Stage"
        checkout scm
        script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        }
    }
    stage('Test') {
      echo "2.Test Stage"
    }
    stage('Build') {
        echo "3.Build Docker Image Stage"
        sh "docker build -t 192.168.3.123:30002/ukubernetes/jenkins-demo:${build_tag} ."
    }
    stage('Push') {
        echo "4.Push Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: 'DockerHub', passwordVariable: 'DockerHubPassword', usernameVariable: 'DockerHubUser')]) {
            sh "docker login http://192.168.3.123:30002 -u ${DockerHubUser} -p ${DockerHubPassword}"
            sh "docker push 192.168.3.123:30002/ukubernetes/jenkins-demo:${build_tag}"
        }
    }
    stage('Deploy') {
        echo "5. Deploy Stage"
        def userInput = input(
            id: 'userInput',
            message: 'Choose a deploy environment',
            parameters: [
                [
                    $class: 'ChoiceParameterDefinition',
                    choices: "Dev\nQA\nProd",
                    name: 'Env'
                ]
            ]
        )
        echo "This is a deploy step to ${userInput}"
        sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
        //sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s.yaml"
        if (userInput == "Dev") {
            // deploy dev stuff
            sh "kubectl -n dev apply -f k8s.yaml"
        } else if (userInput == "QA"){
            // deploy qa stuff
            sh "kubectl -n test apply -f k8s.yaml"
        } else {
            // deploy prod stuff
            sh "kubectl -n kube-ops apply -f k8s.yaml"
        }
    }
}
