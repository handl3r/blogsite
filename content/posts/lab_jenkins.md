*** Dựng lab chơi chơi chạy jenkins nào ***
Ai chưa biết về CI/CD thì tìm hiểu trước đi nhé.
# Mô tả lab cơ bản như sau:
* Project là 1 con web cơ bản viết bằng golang cần được test và deploy liên tục lên server thông qua jenkins.
* SCM là github. Link repo https://github.com/handl3r/multibranches
* Server deploy dùng docker hay máy ảo cũng được. Mình dùng ubuntu server chạy bằng virtualbox. Chú ý setup card mạng với set static ip cho nó nha.
* Server chạy jenkins thì mình chạy bằng docker. Jenkins cần nhận data từ github gửi về qua web-hook nên phải có public ip. Cái này thì mình dùng ngrok tạo tunnel nha.
# Cụ thể :
##  0. Mô tả luồng :
Cứ có code mới là thằng jenkins lấy về chạy test thôi. Nếu mà là code mới cho nhánh master, develop (push event kiểu như push thẳng tay hoặc merged) thì nó chạy cả deploy nữa.
Luồng phát triển thông thường nên làm thế này: 
* Các tính năng mới hay fix-bug, balabala thì push lên nhánh mới rôi tạo pull request vào develop
* Người chịu trách nhiệm project duyệt code xem ok không. Nếu ok thì merge nhánh mới zo nhánh develop. Nhánh này tương ứng với môi trường dev nha. Có code mới phát là thằng jenkins lấy code về rồi chạy test, balabala rồi deploy lên server dev.
* 1 thời gian thấy oke thì merge zo master tương ứng với môi trường product. Jenkins lại lấy code về chạy test balabala rồi deploy lên server product.
Bằng cách này thì code nào cũng được test cẩn thận, không pass phát là báo về github ngay. Hoặc có thể setup cho nó gửi mail cũng được. Hoặc lên thằng jenkins xem luôn.
## 1. Project Golang
* main.go
```golang
package main

import (
	"fmt"
	"net/http"
)

func main() {
	fmt.Println("Hello, this is multibranches app v3")
	fmt.Printf("Result of call Min(2, 3) is: %d\n", Min(2, 3))
	fmt.Println("Start web server")
	http.HandleFunc("/ping", Ping)
	_ = http.ListenAndServe(":8000", nil)
}

func Ping(w http.ResponseWriter, r *http.Request) {
	_, _ = fmt.Fprintf(w, "%s\n", "Pong v3")
}

func Min(a, b int) int {
	if a < b {
		return a
	}
	return b
}
```
1 con http server đơn giản pingpong thoiiii. func Min() để chạy test tượng trưng.
* main_test.go
```golang
package main

import "testing"

func TestMin(t *testing.T) {
	expected := 2
	result := Min(2, 3)
	if result != expected {
		t.Errorf("Expected %d, but got %d\n", expected, result)
	}
}
```
Test hàm Min thôi. Để chạy test thì:
```shell script
    go test -v
```
* go.mod, go.sum để thằng go module quản lí package thôi. Ko care
* Jenkinsfile Khai báo các stages lát thằng jenkins nó làm theo thôi. Lát mình cụ thể hơn.
---
## 2. SCM Github
Github đóng vai trò gửi các event khi mà có thay đổi trong source code của repo về thằng jenkins thông qua webhook thôi.
Nó cũng nhận data trả ngược lại để thông báo test hay deploy có thành công không nữa.
## 3. Jenkins
Vai trò jenkins là gì. Nó là 1 người công nhân cần mẫn của mình thôi. Ngày xưa không có công nhân thì mình lấy code mới về này. setup các thứ này. Chạy test này. thấy oke thì lên github merge zo này. Xong muốn deploy thì phải ssh zo server này. Down service đi này, lấy code mới về build trên server rồi chạy này. Mệt ghia.
Bây giờ mệt quá, thuê 1 anh công nhân tên jenkins. Làm hết mọi thứ lằng nhằng bên trên. Lương thì không phải trả, chỉ tốn chi phí setup thôi. Hơi giống slave dog nhỉ :v. Mà slave dog thì có khi còn tự setup hộ ý. Ông nào viết được cái tool Slave-Dog thì đỉnh vch.
Thằng jenkins không đủ thông minh để biết làm gì cụ thể thế nào đâu. Mình phải đưa cho nó bản công việc (jenkinsfile), cho nó lệnh bài để truy cập vào server (credential)
Nói chung thằng jenkin thực hiện những công việc thủ công để mình đỡ phải làm đi làm lại.
Làm việc với thằng jenkins này thế nào. Mình phải chạy nó lên rồi đăng nhập vô. Tạo 1 project cho nó rồi setup link repo, setup plugin, trigger, credentials, balalalal. Làm sao để nó biết sẽ phải làm gì khi có code mới và có đủ thẩm quyển là điều đó 
Bây giờ thì xem thằng jenkinsfile nào. Đây là bản công việc nhé:
```textmate
pipeline {
    agent any
    tools {
        go 'Golang1.15'
    }
    environment {
        GO114MODULE = 'on'
        CGO_ENABLED = 0
        GOPATH = "${JENKINS_HOME}/jobs/${JOB_NAME}/builds/${BUILD_ID}"
    }

    stages {

        stage('Pre Test') {
            steps {
                echo 'Installing dependencies'
                sh 'go version'
                sh 'go get -u golang.org/x/lint/golint'
                sh 'go get'

            }
        }

        stage('Test') {
            steps {
                sh 'go vet .'
                echo 'Running linting'
                sh 'golint .'
                echo 'Running test'
                sh 'go test -v'
            }
        }

        stage('Build And Deploy') {
            when {
                branch 'master'
            }
            steps {
                echo 'Compiling and building'
                sh 'go build -o main main.go'
                sshagent(['my-ssh-key']) {
                    sh 'ssh thai@192.168.1.10 pkill main'
                    sh 'scp ./main thai@192.168.1.10:~/app/'
                    sh '''
                       ssh thai@192.168.1.10 '~/app/main > ~/app/start.log &'
                    '''
                }
            }
        }

    }
}
```
Phân tích nào. Thằng jenkins nhận được bản công việc này ở trên github repo. Thực hiện từ trên xuống:
* Tool dùng là go 'Golang1.15' cái này lát phải define trong config nha. Mình không xem rõ cái tool này làm gì, chắc là pull docker golang về rồi chạy hoặc là cài golang lên node rồi chạy trực tiếp.
* Cài vài biến môi trường environment{}
* Các bước thực hiện stages{} gồm nhiều stage nhỏ:
    * Pre Test: xem version của go này, pull package golint về này. Cái này để lát check syntax code. Go get là để pull tất cả package cần cho việc build và chạy project golang nha.
    * Test: Chạy test nào. Đầu tiên là chạy go vet để xem có error nào trong code không. Sau thì chạy golint . kiểm tra syntax . Chạy testgo test -v
    * Test thành công thì deploy thôi: Chỉ thực hiện khi target branch là master nha.
    when { branch 'msater' }. go build -o main main.go Để build ra file binary main.
    Dùng plugin ssh agent nó cung cấp cho cái khai báo sshagent. Mục đích là để ssh vào server bằng cái credential my-ssh-key mà mình setup trong cài đặt của jenkins (lênh bài đó). Chạy ssh vào server và kill thằng process main. Sau đó dùng scp gửi file mình vừa build được lên server. Rồi chạy nó thôi. Mình phải đẩy log vào file để tránh trường hợp nó bị treo ssh làm jenkins không end task này được.
* Đó là bản công việc đó. Còn phải setup nhiều.Lát mình đi chi tiết hơn
## 4. Server deploy  
Mình chơi mỗi product thôi. Chạy 1 con ubuntu server trên virtualbox làm server. Gen key ssh ra rồi dùng nó là credential cho jenkins truy cập zo.  
***
4 Yếu tố đủ rồi. Bắt đầu đi chi tiết nào.
# Thực hành
Yêu cầu: docker, ngrok,...

## 1. Project.
Các bạn tự viết hoặc lấy trên repo của mình về test nha. Đẩy lên repo của bạn.
Chú ý là phải có thằng Jenkinsfile trong project để thằng jenkins có bản công việc nha.
## 2. Cài jenkins
Tham khảo trên trang chủ nha. Mình thì chạy bằng docker. Muốn đơn giản thì dùng compose nữa nhá.  
Đầu tiên phải tạo 1 volume cho thằng container mỗi khi run lên có dữ liệu đã
```shell script
docker volume create jenkins-data
```
Tạo cái file bash chạy cho tiện
jenkins.sh
```text
docker run -p 50000:50000 \
           -p 8080:8080 \
           -v jenkins-data:/var/jenkins_home \
           jenkins/jenkins:latest
```
Nhớ cấp quyền cho nó nha.
Chạy lên. Lần đầu chạy thì nó phải pull cả đống về nên lâu phết.
chạy ngon lành rồi thì chú ý log để  lấy access-key gì đó mà login vào jenkins nha. Xong có tạo tài khoản gì đó.
balabala... Thôi mấy cái râu ria các bạn tự search làm nha. Mình cũng chả nhớ cụ thể :v.
Kết quả là mình có 1 con jenkins chạy ngon lành trên docker
## 3. Setup jenkins
Mình nhớ vài cái plugin có dùng : Golang plugin, ssh agent, blablabla, chả nhớ. Cứ làm rồi thấy thiếu thiếu gì thì m.n tự search rồi cài nha.
## 4. Publish jenkins lên làm webhook.
Dùng tool ngrok nha. Cài xong thì
```shell script
./ngrok http 8080
```
![alt text](http://url/to/img.png) ngrok1.png

Cái subdomain mà thằng ngrok dùng làm webhook nha:
vào github repo và set webhook cho nó: http://97850f9e1993.ngrok.io/github-webhook/
Thằng github và jenkins sẽ giao tiếp qua thằng này nha.
## 5. Tạo project trên jenkins và setup.
Chọn Multibranch Pipeline nha.  
* Branch Sources dùng github, điền đẩy đủ thông tin về repo vào nhé 
    * Credential thì các bạn điền zo. Dùng token hoặc username, password đều được. Miễn sao giúp cho jenkins đủ thẩm quyền nhảy vào repo của bạn là oke.
    * Behaviours discovery all branch nha. Cho thằng jenkins biết tất luôn.
    * Discover pull requests from origin để merge pull request with target branch nha. Đại loại là nó sẽ phát hiện ra khi có pull request để merge vào 1 nhánh nào đó.
* Build Config thì để by Jenkinsfile nha. Nghĩa là nó dùng jenkinsfile trên repo của mình ý.
* Orphaned Item Strategy : Day to keep để 365, Max # để 15. Mình xem trên mạng chứ cũng chả đọc chỗ  này.
Setup ngon lành rồi thì ấn zo Scan repo now cho nó quét repo của mình và theo dõi hết branch nha. Các chức năng trên ui của nó tự tìm hiểu nhé các bạn. Thế cho kích thích. :v
Hình thù sau khi nó scan xong thì thế này đây:

Ảnh x1

Vào Multibrach pipeline events mà xem mấy cái event bắn qua lại giữa jenkins với github nha. Nhìn sướng mắt. 

Quên mất. Nhớ cài plugin Blue Ocean tí dễ nhìn nha. 

Oke rồi. Giờ này thằng jenkins truy cập được vào repo của mình rồi. Quét oke rồi. Giờ test thử.

## Test thử
Đẩy nhánh mới lên github nha. Chú ý xem log của ngrok sẽ thấy thằng git hub bắn event về cho jenkins. Bạn sẽ thấy ttl tăng nha. opn cũng nhảy 1 lúc. Data từ github bắn về  web-hook chính là cái domain mà ngrok mở cho mình. Nó sẽ forward về jenkins local của mình. Thằng jenkins bắt được thì nó sẽ chạy theo pipeline của mình. Chính là công việc định nghĩa trong Jenkinsfile đó.
Truy cập zo Ocean Blue nha. 
Ảnh x2.png
Nhìn pipeline của nó có kích thích không ạ. Ấn vào từng Stage để  xem kĩ nó thực hiện những instruction của mình trong jenkinsfile nha.  
Tại sao không deploy vậy? Vì điều kiện when của mình trong jenkinsfile đó. push action không có xảy ra trên master nên đến stage đó thì nó không chạy thôi.
Ảnh x3.
Nhìn này. jenkins cũng bắn data cho github để nó biết là các stage chạy pass ngon lành cành đào.

Jenkins nhiều chức năng lắm . Mình cũng biết có xíu à. Các bạn tự tìm hiểu nhé.
Các bạn thử  tạo pull request rồi merge linh tinh xem. Rồi quay sang jenkins web view mà xem chi tiết.
Nhớ chý ý cả ngrok là biết thằng github nó bắn data qua lại cho jenkins ý mà.
Mà quên mất lúc set webhook cho repo github nhớ chọn send me everything nha.  
Xem như là chạy ngon rồi. Có lỗi gì tự search sửa nha.

## 5. Setup auto deploy cho jenkins và server nha.
Có ubuntu server rồi. Có jenkins rồi. Bây giờ làm sao đây?  
Chỉ dẫn công việc thì có rồi. Trong stage build và deploy ý.
Nhưng mà vô ích vì có lệnh khám nhà đâu mà thằng ubuntu cho jenkins zo.
Cùng tạo credential nào: 
* Sinh ssh key ra nha. Đẩỷ public key cho thằng ubuntu server, nhớ thêm vào authorized gì đó.
* Vào manager credentials trên jenkins Add credentials thôi. Rồi điền các thức, private key zo. Có lệnh bài rồi. Quay lại jenkins file mà sửa key của mình thành tên key của bạn nha. Sửa luôn cả username và ip của con server của bạn nữa nhé. Cứ làm sao cho con jenkins truy cập vào trong ubuntu server là được. Đoạn này chắc sẽ làm lỗi nhiều ý. Cứ tự search nha :))
Có thánh chỉ rồi. Có workflow rồi thì test thử  thôi. 

Mà chưa trước tiên build cái project rồi đẩy file binary vào ubuntu server chạy lên. Chạy rồi test thử trên web browser nha. 192.168.1.10/ping sẽ thấy message là là Pong nhé.

Nhìn lại build and deploy stage nào:  
```text
stage('Build And Deploy') {
            when {
                branch 'master'
            }
            steps {
                echo 'Compiling and building'
                sh 'go build -o main main.go'
                sshagent(['my-ssh-key']) {
                    sh 'ssh thai@192.168.1.10 pkill main'
                    sh 'scp ./main thai@192.168.1.10:~/app/'
                    sh '''
                       ssh thai@192.168.1.10 '~/app/main > ~/app/start.log &'
                    '''
                }
            }
        }
```
Thằng jenkins sẽ build project ra file binary main. Nó ssh vào server với credential my-ssh-key. rồi kill process main trên server nhá. Lúc này thằng http server của mình down cái app ping đi rồi. Tiếp theo dùng scp để truyền file main vừa mới build được trên jenkins( file main này build từ version mới của code nha) lên server. Rồi sau đó là chạy nó thôi. vào web browser để trải nghiệm nhá. Nhớ là push code mới lên thì đổi thành Pong 3 hay gì gì đó để  lúc xem lại thành quả cho dễ.

## Test quy trình deploy nào.

Trên mình có nói qua rồi. 
* Các bạn tạo nhánh mới đổi code đi, ví dụ như thay message Pong chẳng hạn. 
* Push code lên nhánh mới. Chú ý ngrok sẽ thấy github bắn data về cho jenkins.
* Vào jenkins để ý Build Queue rồi zo Ocean blue nhìn cho dễ quá trình nó chạy pipeline. thấy nó test.
* Tạo merge request vào main
* Lặp lại mấy bước quan sát trên nhìn cho thích mắt.
* Merge pull request vừa rồi vào master rồi lại quan sát. Lần này sẽ thấy nó chạy stage Build and Deploy nha.
Ảnh x4.

Nhìn thú dzi đúng không. vào web browser test thử ngay con app mới vừa deploy.

Đấy. Từ nay có project nào chơi chơi. Thử làm xem. Merge code cái là boommmmmm. Tự nhiên 1 lúc sau thấy con app mới đã chạy ngon lành trên server. Slave dog hiệu quả vclLz :v.

Tạm đến đây đã. Nao có hứng chi tiết thêm tí cho ae. Chứ với từng kia hướng dẫn thì với mấy ông mới mới chắc ngồi search lỗi chán chê. Thật ra mình cũng mới tìm tòi đc có 1 tuần với lí do là không may. Cơ mà biết thêm 1 cái cũng thú dzi.  
Tạm thế.







