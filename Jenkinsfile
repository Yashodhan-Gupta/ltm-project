pipeline {
    agent { label 'controller' }
    environment {
        IMAGE_NAME = "mindtreerepo"
        IMAGE_TAG  = "${BUILD_NUMBER}"
	    IMAGE_REPO="public.ecr.aws/y5h2u1j4/mindtree"
 
    }
 
    stages {
 
        stage('Checkout Code') {
            steps {
                echo "Checking out source code"
                git 'https://github.com/Yashodhan-Gupta/ltm-project'
                sh 'sudo chown jenkins:jenkins /var/lib/jenkins/efs'
                sh 'sudo cp -r * /var/lib/jenkins/efs'
                sh 'sudo chown jenkins:jenkins /var/lib/jenkins/efs/*'
            }
        }
 
        stage('Build Application (Java + Maven)') {
            agent { label 'build' }
 
            steps {
 
                sh '''
                    sudo chown jenkins:jenkins /home/jenkins/build
                    sudo chmod 777 /home/jenkins/build
                    cd /home/jenkins/build
                    sudo chown jenkins:jenkins /home/jenkins/build/*
                    sudo mvn clean package
                    cd target
                    cp ROOT.war /home/jenkins/build && cd ..
                    ls -al
                '''
            }
        }
 
        stage('Build & Push Docker Image') {
            agent { label 'docker' }
            steps {
                sh '''
                    sudo chown jenkins:jenkins /home/jenkins/docker
                    sudo chmod 777 /home/jenkins/docker
                    cd /home/jenkins/docker
                    ls -al
                    sudo chown jenkins:jenkins /home/jenkins/docker/*
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/m1r4i2g4
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_REPO}:${IMAGE_TAG}
                    docker push ${IMAGE_REPO}:${IMAGE_TAG}
                '''
            }
        }
        stage('Deploy to Kubernetes') {
            agent { label 'docker' }
            steps {
                sh '''
				 cd /home/jenkins/docker
                 sudo kubectl delete deployment tomcat --ignore-not-found=true
                 sudo kubectl delete pods --all --force --grace-period=0
                 sudo kubectl apply -f deployment.yml
                 sudo kubectl set image deployment/tomcat mindtreerepo="${IMAGE_REPO}:${IMAGE_TAG}" --record
                 sudo kubectl apply -f svc.yml
              '''
            }
        }
 
        
    }
 
}
