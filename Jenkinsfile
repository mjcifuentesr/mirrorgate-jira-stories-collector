/*
 * Copyright 2017 Banco Bilbao Vizcaya Argentaria, S.A.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

node('master') {

    stage('Checkout') {
        checkout(scm)
    }

    stage('Build jar file') {
        sh """
            ./gradlew clean build --refresh-dependencies
        """
    }

    stage('Publish on Jenkins') {
            sh """
                mv build/libs/*.jar build/libs/mirrorgate_jira_stories_collector-1.0-SNAPSHOT-all.jar
            """
      	    step([$class: "ArtifactArchiver", artifacts: "build/libs/mirrorgate_jira_stories_collector-1.0-SNAPSHOT-all.jar", fingerprint: true])
    }

    if(env.BRANCH_NAME == 'master' || env.BRANCH_NAME == 'develop') {
        stage('Docker - build docker image') {
            sh """
            docker build -t bbvaae/mirrorgate-jira-stories-collector .
            """
        }

        stage('Docker - push') {
            withCredentials([[$class : 'UsernamePasswordMultiBinding',
                              credentialsId   : "bot-mirrorgate-dh",
                              usernameVariable: 'DOCKER_USER',
                              passwordVariable: 'DOCKER_PASSWORD']]) {
                if (env.BRANCH_NAME == 'master') {
                    sh """
                        docker login --username $DOCKER_USER --password $DOCKER_PASSWORD
                        docker tag bbvaae/mirrorgate-jira-stories-collector bbvaae/mirrorgate-jira-stories-collector:release
                        docker push bbvaae/mirrorgate-jira-stories-collector:release
                    """
                }
                sh """
                    docker login --username $DOCKER_USER --password $DOCKER_PASSWORD
                    docker tag bbvaae/mirrorgate-jira-stories-collector bbvaae/mirrorgate-jira-stories-collector:latest
                    docker push bbvaae/mirrorgate-jira-stories-collector:latest
                """
            }
        }
    }

}
