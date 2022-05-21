## jenkins pipeline

### plugins

* [Ansible](https://plugins.jenkins.io/ansible)
* [Pyenv Pipeline](https://plugins.jenkins.io/pyenv-pipeline)
* [Credentials](https://plugins.jenkins.io/credentials)
* [NodeJS](https://plugins.jenkins.io/nodejs)
* [SSH Agent](https://plugins.jenkins.io/ssh-agent)
* [SSH Credentials](https://plugins.jenkins.io/ssh-credentials)
* [HTML Publisher](https://plugins.jenkins.io/htmlpublisher)

```
pipeline {  
    agent any

    tools {
        nodejs "NodeJS 16.15.0"
    }

    parameters {
        string(name: 'test_script_path', defaultValue: './src/test-scripts/demo.ts')
        string(name: 'ec2_tag', defaultValue: 'webinar_nodejs')
    }
  
    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY= credentials('aws-secret-access-key')
    }

    stages {
        stage('install ansible') {
            steps {
                withPythonEnv('python3') {
                    sh 'pip install --upgrade pip'
                    sh 'pip install ansible jupyterlab nbconvert pandas plotly'
                }
            }
        }
  
        stage('git checkout') {
            steps {
                git branch: 'dev', credentialsId: 'git-access-key', url: 'https://github.com/dyingtime/notes.git'
            }
        }
  
        stage('ansible start instance') {
            steps {
                sshagent(credentials: ['aws-ssh-private-key']) {
                    withPythonEnv('python3') {
                        ansiblePlaybook (
                            credentialsId: 'aws-ssh-private-key',
                            playbook: 'ansible/start_instance.yml',
                            extras: '-e action=start -e tag=${ec2_tag}'
                        )
                    }

                }
            }
        }
  
        stage('install ts-node') {
            steps {
                sh 'npm install ts-node'
            }
        }
  
        stage('execute test') {
            steps {
                withPythonEnv('python3') {
                    sh 'NODE_PATH=./script/test-scripts npx ts-node --files ${test_script_path}'
                }
            }
        }
  
        stage('publish report') {
            steps {
                publishHTML(
                    target : [
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'report',
                        reportFiles: '*.html',
                        reportName: 'test-report'
                    ]
                )
            }
        }
    }
  
    post {
        cleanup {
            withPythonEnv('python3') {
                ansiblePlaybook (
                    playbook: 'ansible/stop_instance.yml',
                    inventory: 'ansible/aws_ec2.yml', 
                    extras: '-e tag=${ec2_tag}'
                )
            }
        }
    }
}
```
