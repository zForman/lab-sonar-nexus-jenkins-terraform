pipeline {
    agent any
    tools {
        maven 'maven'
    }

    environment {
        ArtifactId  = readMavenPom().getArtifactId()
        Version     = readMavenPom().getVersion()
        Name        = readMavenPom().getName()
        GroupId     = readMavenPom().getGroupId()
    }

    stages {
        stage ('Build') {
            steps {
                sh 'mvn clean install package'
            }
        }

        stage ('Test') {
            steps {
                echo 'testing ...'
            }
        }

        stage ('Publish to Nexus') {
            steps {
                script { 
                def NexusRepo = Version.endsWith("SNAPSHOT") ? "EpamDevOpsHW-SNAPSHOT" : "EpamDevOpsHW-RELEASE"
                
                nexusArtifactUploader artifacts:
                [[artifactId: "${ArtifactId}",
                classifier: '',
                file: "target/${ArtifactId}-${Version}.war",
                type: 'war']],
                credentialsId: '51231d5b-eb83-4b91-8e8f-56020908d643',
                groupId: "${GroupId}",
                nexusUrl: '172.20.10.155:8081',
                nexusVersion: 'nexus3',
                protocol: 'http',
                repository: "${NexusRepo}",
                version: "${Version}"
                }
            }
        }
	
        // stage ('Print ENV Variables') {
        //     steps {
        //         echo "Artifact ID is '${ArtifactId}'"
        //         echo "Version is '${Version}'"
        //         echo "GroupID is '${GroupId}'"
        //         echo "Name is '${Name}'"
        //     }
        // }

        stage ('Deploy') {
            steps {
                // echo 'deploying ...'
                sshPublisher(publishers: 
                [sshPublisherDesc(
                    configName: 'Ansible Controller', 
                    transfers: [
                        sshTransfer(
                            cleanRemote: false,
                            execCommand: 'ansible-playbook /opt/playbooks/downloadanddeploy.yaml -i /opt/playbooks/hosts',
                            execTimeout: 120000
                        )
                    ], 
                    usePromotionTimestamp: false, 
                    useWorkspaceInPromotion: false, 
                    verbose: false)
                    ])
            }
        }
    }
}
