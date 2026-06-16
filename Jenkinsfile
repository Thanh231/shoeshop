pipeline {
    agent { 
        node { 
            label 'server-01' // Chạy trên máy ảo Agent của bạn
        } 
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '5'))
        timestamps()
    }

    // Định nghĩa các biến môi trường cho ứng dụng Spring Boot
    environment {
        // Nạp tự động Username và Password từ Credentials Jenkins đã tạo ở Bước 1
        DB_CREDS = credentials('mysql-shoeshop-creds') 
        
        // Cấu hình các biến Spring Boot sẽ đọc lúc chạy Test hoặc Build
        SPRING_DATASOURCE_URL = 'jdbc:mysql://localhost:3306/shoeshop?useSSL=false&serverTimezone=UTC'
        DB_USERNAME = "${env.DB_CREDS_USR}"
        DB_PASSWORD = "${env.DB_CREDS_PSW}"
    }

    stages {
        // Giai đoạn 1: Kiểm tra môi trường Java & Maven
        stage('Check Environment') {
            steps {
                echo '===== Checking Java & Maven Tools ====='
                sh 'java -version'
                sh 'mvn -v' // Nếu dự án dùng Gradle, thay bằng: sh './gradlew -v'
            }
        }

        // Giai đoạn 2: Dọn dẹp và Chạy Unit Test (Kiểm tra kết nối DB nội bộ nếu có)
        stage('Unit Test') {
            steps {
                echo '===== Running Database & Application Tests ====='
                // Khởi chạy test với các biến môi trường Database đã nạp ở trên
                sh 'mvn clean test' 
            }
        }

        // Giai đoạn 3: Biên dịch dự án ra file .jar
        stage('Build Artifact') {
            steps {
                echo '===== Packaging Spring Boot Application ====='
                // Bỏ qua bước test lại để build cho nhanh
                sh 'mvn package -DskipTests' 
            }
        }

        // Giai đoạn 4: Deploy (Triển khai chạy ngầm dự án Spring Boot trên Server)
        stage('Deploy Application') {
            steps {
                echo '===== Deploying Spring Boot Jar ====='
                script {
                    // Tắt ứng dụng cũ đang chiếm port 8080 (nếu có)
                    sh """
                        PID=\$(ps -ef | grep 'shoe-ShoppingCart' | grep -v grep | awk '{print \$2}')
                            if [ ! -z "\$PID" ]; then
                            echo "Stopping application with PID: \$PID"
                        sudo kill -9 \$PID
                            else
                            echo "Application is not running."
                            fi
                        """
                    
                    withCredentials([usernamePassword(credentialsId: 'mysql-shoeshop-creds',usernameVariable: 'MY_USER',passwordVariable: 'MY_PASS')]) {
                    sh """
                        JENKINS_NODE_COOKIE=dontKillMe nohup java -jar target/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar \
                        --spring.datasource.url="${SPRING_DATASOURCE_URL}" \
                        --spring.datasource.username="${MY_USER}" \
                        --spring.datasource.password="${MY_PASS}" > springboot.log 2>&1 &
                    """
                    }
                    echo 'Spring Boot Application is running in background. Check log at springboot.log'
                }
            }
        }
    }

    post {
        success {
            echo 'Trạng thái: PIPELINE SPRING BOOT THÀNH CÔNG 🎉'
        }
        failure {
            echo 'Trạng thái: PIPELINE BỊ LỖI LÚC BUILD HOẶC TEST ❌'
        }
    }
}
