
pipeline {
    agent any 
    stages {

        stage('SCM') {
            steps {
                script {
                    properties([pipelineTriggers([pollSCM('* * * * *')])])
                }
                git branch: 'main', url: 'https://github.com/morg1207/ros1_ci.git'
            }
        }

        stage('Will check if we need to clone or just pull') {
            steps {
                sh 'cd ~/catkin_ws/src'
                sh '''
                    #!/bin/bash
                    if [ ! -d "ros1_ci" ]; then
                        git clone https://github.com/morg1207/ros1_ci.git
                        echo 'no existe el repositorio'
                    else
                        cd ros1_ci
                        git pull origin main
                        echo 'existe el repositorio'
                    fi
                    '''
            }
        }
        stage(' install and build docker image') {
            steps {
                sh '''
                sudo apt-get update
                sudo apt-get install docker.io docker-compose -y
                sudo service docker start
                sudo usermod -aG docker $USER
                newgrp docker
                cd ~/catkin_ws/src/ros1_ci
                sudo docker build -t tortoisebot_test .
                '''
            }
        }
        stage('Create container') {
            steps {
                
                sh '''
                sudo usermod -aG docker $USER
                newgrp docker
                sudo docker run --rm --name tortoisebot_container -e DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix tortoisebot_test:latest &
                sleep 10
                sudo usermod -aG docker $USER
                newgrp docker
                sudo docker exec tortoisebot_container /bin/bash -c ". /opt/ros/noetic/setup.bash && . catkin_ws/devel/setup.bash && rostest tortoisebot_waypoints waypoints_test.test --reuse-master"

                '''

            }
        }
        stage('Done') {
            steps {
                sleep 2
                sh 'sudo docker rm -f tortoisebot_container'
                echo 'Pipeline completed'
            }
        }
    }
}