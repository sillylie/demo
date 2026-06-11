pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    stages {
        stage('拉取代码') {
            steps {
                cleanWs()
                sh 'git clone git@github.com:sillylie/demo.git .'
            }
        }

        stage('测试') {
            steps {
                sh 'mvn test'
            }
        }

        stage('构建') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('启动服务') {
            steps {
                sh 'pid=$(lsof -t -i:8099 2>/dev/null || true); if [ -n "$pid" ]; then kill $pid || true; sleep 3; fi'
                sh 'JENKINS_NODE_COOKIE=dontKillMe nohup java -jar target/demo-*.jar --server.address=0.0.0.0 --server.port=8099 > app.log 2>&1 &'
            }
        }

        stage('健康检查') {
            steps {
                script {
                    for (int i = 0; i < 10; i++) {
                        try {
                            sh 'curl -sf http://localhost:8099 > /dev/null'
                            echo '>>> 服务已启动: http://localhost:8099'
                            return
                        } catch (Exception e) {
                            echo '等待启动中...'
                            sleep time: 3, unit: 'SECONDS'
                        }
                    }
                    error '服务启动超时'
                }
            }
        }
    }
}