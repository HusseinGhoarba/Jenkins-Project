pipeline {
    agent { label 'jenkins-slave' }
    stages {
        stage('Build') {
            steps {
                script {
                    if (env.BRANCH_NAME == "developers" ) {
                       withCredentials([usernamePassword(credentialsId: 'hu-dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                           sh """
                                docker login -u $USERNAME -p $PASSWORD
                                docker build -t husseinghoraba/bakehouse:${BUILD_NUMBER} .
                                docker push husseinghoraba/bakehouse:${BUILD_NUMBER}
                                echo ${BUILD_NUMBER} > ../bakehouse-build-number.txt
                           """
                       }
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == "main" || env.BRANCH_NAME == "test" || env.BRANCH_NAME == "prod") {
                        withCredentials([file(credentialsId: 'k8s-conf', variable: 'KUBECONFIG')]) {
                            sh """
                                  export BUILD_NUMBER=\$(cat ../bakehouse-build-number.txt)
                                  mv Deployment/deploy.yaml Deployment/deploy.yaml.tmp
                                  cat Deployment/deploy.yaml.tmp | envsubst > Deployment/deploy.yaml
                                  rm -f Deployment/deploy.yaml.tmp
                                  kubectl apply -f Deployment --kubeconfig=${KUBECONFIG}
                            """
                        }   
                    }       
                }
            }
        }
    }
}

