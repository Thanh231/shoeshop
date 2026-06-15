pipeline {
    // 1. Chỉ định chạy Pipeline này trên Agent có tên là 'server-01' của bạn
    agent { 
        node { 
            label 'server-01' 
        } 
    }

    // Cấu hình lưu trữ lịch sử build và thời gian chạy
    options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    stages {
        // Giai đoạn 1: Kiểm tra các công cụ trên Agent xem đã sẵn sàng chưa
        stage('Check Environment') {
            steps {
                echo '===== Checking Environment Tools ====='
                sh 'java -version'
                sh 'node -v'
                sh 'npm -v'
            }
        }

        // Giai đoạn 2: Kéo code mới nhất từ GitHub về (Mặc định Jenkins tự làm, bước này để hiển thị log)
        stage('Checkout Source Code') {
            steps {
                echo '===== Source Code Checked Out Successfully ====='
                // Thư mục làm việc hiện tại của Agent
                sh 'pwd'
                sh 'ls -la'
            }
        }

        // Giai đoạn 3: Cài đặt các thư viện cần thiết cho dự án
        stage('Install Dependencies') {
            steps {
                echo '===== Installing Project Dependencies ====='
                // Sử dụng npm ci (Clean Install) giúp cài đặt nhanh và đúng phiên bản trong package-lock.json
                sh 'npm ci'
            }
        }

        // Giai đoạn 4: Tiến hành Biên dịch / Build dự án
        stage('Build Project') {
            steps {
                echo '===== Building Project ====='
                sh 'npm run build'
            }
        }

        // Giai đoạn 5: Triển khai ứng dụng (Ví dụ: Restart dự án hoặc copy sản phẩm sang thư mục chạy)
        stage('Deploy') {
            steps {
                echo '===== Deploying Application ====='
                // Bạn có thể viết lệnh khởi chạy dự án, ví dụ như PM2 hoặc restart service tại đây
                echo 'Application deployed successfully on server-01!'
            }
        }
    }

    // 2. Các kịch bản xử lý sau khi chuỗi Pipeline kết thúc
    post {
        success {
            echo 'Trạng thái: PIPELINE THÀNH CÔNG RỰC RỠ! 🎉'
        }
        failure {
            echo 'Trạng thái: PIPELINE BỊ LỖI RỒI! ❌ Hãy kiểm tra lại log của stage thất bại nhé.'
        }
        always {
            echo '===== Chuỗi CI/CD đã kết thúc xử lý ====='
        }
    }
}
