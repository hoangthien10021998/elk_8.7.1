# elk_8.7.1
Install elk 8.7.1
Chuẩn bị 3 VM cài centos 7 , Tắt Selinux
3 VM có 3 IP như sau
172.16.10.52
172.16.10.53
172.16.10.54
Cài đặt Elastic Cluster: 3 node
Chú ý: Khi cài đặt cluster ta cài node 1 lên sau đó gen token.
Tiếp cài elastic lên. Mà ko đc start nếu ko sẽ báo lỗi skpiping….
Ta join các node phụ vào sau đó mới đc start elastic lên
Step 1: Install Java trên 3 node
yum -y install java-1.8.0-openjdk-devel.x86_64 
Check: 
rpm -qa | grep openjdk
java -version

Step 2: Download the Elasticsearch RPM trên 3 node
rpm -ivh https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.6.2-x86_64.rpm
Check: rpm -qa | grep elasticsearch

Step 3: Config Elastic cluster trên 3 node
File cấu hình: /etc/elasticsearch/elasticsearch.yml

Tham khảo file cấu hình trên 3 node tại:
Node1: https://github.com/hoangthien10021998/elk_8.7.1/blob/main/elasticsearch.yml
Node2: https://github.com/hoangthien10021998/elk_8.7.1/blob/main/node2_elasticsearch.yml
Node3: https://github.com/hoangthien10021998/elk_8.7.1/blob/main/node3_elasticsearch.yml
Nhớ sửa tên file thành elasticsearch.yml
 
Trên node1:
[root@localhost ~]# grep -Ev '^#|^$' /etc/elasticsearch/elasticsearch.yml
cluster.name: sds-elk
node.name: node-1
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
http.port: 9200
xpack.security.enabled: true
xpack.security.enrollment.enabled: true
xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/http.p12
xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
cluster.initial_master_nodes: ["localhost.localdomain"]
http.host: 0.0.0.0
transport.host: 0.0.0.0
[root@localhost ~]#
Sau khi cài trên node 1 xong ta join các node còn lại vào node 1
Cài xong elastic ta sẽ có được một số thông tin như sau
-------------------------- Security autoconfiguration information --------------
Authentication and authorization are enabled.
TLS for the transport and HTTP layers is enabled and configured.
The generated password for the elastic built-in superuser is : wEGbCciIWxCDxNQw0Lvp
If this node should join an existing cluster, you can reconfigure this with
'/usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token <token-here>'
after creating an enrollment token on your existing cluster.

You can complete the following actions at any time:
Reset the password of the elastic built-in superuser with
'/usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic'.
Generate an enrollment token for Kibana instances with
 '/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana'.
Generate an enrollment token for Elasticsearch nodes with
/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
-----------------------------------------------------------------------------------------------
Tạo token để join
/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
Sau khi chạy sẽ xuất hiện đoạn mã ta coppy lại và qua node 2 và 3 chạy lệnh dưới để join vào
/usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token \
eyJ2ZXIiOiI4LjYuMiIsImFkciI6WyIxNzIuMTYuMTAuNTI6OTIwMCJdLCJmZ3IiOiJlMGJlZGU5YTNjODllNzAxNGU4NDVkNWM2MTU2ZDVhM2RkNjc5MDM1YTBiMThkOGJmMTk4M2YwYjE5ZDVjNDE4Iiwia2V5Ijoid1U4RnVvWUJqajJwRDRKa3ltbnY6TFhtcTM0c2FRNUdaaVpPQWhvazliZyJ9 
 
Sau khi thêm xong ta khởi động dịch vụ
sudo systemctl daemon-reload
 sudo systemctl enable elasticsearch.service
### You can start elasticsearch service by executing
 sudo systemctl start elasticsearch.service

sudo systemctl daemon-reload
sudo systemctl restart elasticsearch.service
Làm tương tự trên các node còn lại

Trên node 2:
[root@localhost ~]# grep -Ev '^#|^$' /etc/elasticsearch/elasticsearch.yml
cluster.name: sds-elk
node.name: node-2
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
http.port: 9200
xpack.security.enabled: true
xpack.security.enrollment.enabled: true
xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/http.p12
xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
discovery.seed_hosts: ["172.16.10.52:9300"]
http.host: 0.0.0.0
transport.host: 0.0.0.0
[root@localhost ~]#
Trên node3: 
[root@localhost ~]# grep -Ev '^#|^$' /etc/elasticsearch/elasticsearch.yml
cluster.name: sds-elk
node.name: node-3
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
http.port: 9200
xpack.security.enabled: true
xpack.security.enrollment.enabled: true
xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/http.p12
xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
discovery.seed_hosts: ["172.16.10.52:9300"]
http.host: 0.0.0.0
transport.host: 0.0.0.0
[root@localhost ~]#
**** Và nhớ thêm vào cuối file trên 3 node:
# Create a new cluster with the current node only
# Additional nodes can still join the cluster later
discovery.seed_hosts: ["10.100.170.50", "10.100.170.51", "10.100.170.52"]
cluster.initial_master_nodes: ["node1", "node2", "node3"]
# Allow HTTP API connections from anywhere
# Connections are encrypted and require user authentication
http.host: 0.0.0.0

# Allow other nodes to join the cluster from anywhere
# Connections are encrypted and mutually authenticated
transport.host: 0.0.0.0
------------------------------------
Reset the password of the elastic built-in superuser with
'/usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic'.

Generate an enrollment token for Kibana instances with
 '/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana'.

Generate an enrollment token for Elasticsearch nodes with
/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node

Mở firewall cho ELK:
firewall-cmd --add-port={9200,9300-9400}/tcp --permanent
firewall-cmd --reload

Khởi chạy Elastic và chạy cùng hệ thống sau khi restart lại
systemctl start elasticsearch
systemctl enable elasticsearch





Kiểm tra Cluster: curl -k -XGET "https://172.16.10.52:9200/_cat/health?v" -u elastic
Nhập pass:
KQ: status green là ok
 
Các nút hoạt động OK
Check: curl -u elastic --cacert /etc/elasticsearch/certs/http_ca.crt https://172.16.10.54:9200/_cat/nodes?v
 
Tắt hoán đổi bằng systemd bằng cách chỉnh sửa dịch vụ Elaticsearch và thêm nội dung bên dưới;
systemctl edit elasticsearch
[Service]
LimitMEMLOCK=infinity
 
sudo systemctl daemon-reload
Tắt swap
swapoff -a
Vào file /etc/fstab và thêm # vào dòng có swap

Đặt kích thước Heap JVM
Elaticsearch đặt kích thước heap thành 1GB theo mặc định. Theo nguyên tắc thông thường, hãy đặt  Xmx không quá 50% RAM vật lý của bạn nhưng không quá 32GB.
vi /etc/elasticsearch/jvm.options
# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space

-Xms1g
-Xmx1g

Đặt Bộ mô tả tệp mở tối đa
Đặt số lượng tệp mở tối đa cho  elasticsearchngười dùng là 65.536. Điều này đã được đặt theo mặc định trong /usr/lib/systemd/system/elasticsearch.service
 
Bạn cũng nên đặt số lượng processes tối đa.
 
Cài đặt bộ nhớ ảo
Elaticsearch sử dụng một  mmapfs thư mục theo mặc định để lưu trữ các chỉ mục của nó. Để đảm bảo rằng bạn không bị hết bộ nhớ ảo, hãy chỉnh sửa /etc/sysctl.conf và cập nhật giá trị của vm.max_map_count như minh họa bên dưới.
vm.max_map_count=262144
chạy lệnh : echo "vm.max_map_count=262144" >> /etc/sysctl.conf
Reboot lại để áp dụng các thay đổi,và chạy lệnh sysctl vm.max_map_count, để xác minh cấu hình.
 
Mở các port cần thiết:
firewall-cmd --add-port={9200,9300-9400}/tcp --permanent
firewall-cmd --reload

Khởi động elastic:
systemctl daemon-reload
systemctl start elasticsearch
systemctl status elasticsearch.service

Check logs cluster nếu có lỗi:
tail /var/log/elasticsearch/<cluster-name>.log


Sau khi cụm hình thành lần đầu tiên thành công, hãy xóa  cluster.initial_master_nodes cài đặt khỏi cấu hình của từng nút. Không sử dụng cài đặt này khi khởi động lại một cụm hoặc thêm một nút mới vào một cụm hiện có.
Chạy lệnh : 
sed -i 's/^cluster.initial_master_nodes:/#&/' /etc/elasticsearch/elasticsearch.yml

Khởi động lại các node elastic.
systemctl restart elasticsearch

	Đó là tất cả về cách thiết lập cụm Elaticsearch 7.x Cluster trên CentOS 7.
 
Cert của elastic ta lấy ở 1 trong các node elastic tại:
/etc/elasticsearch/certs/http_ca.crt
CÀI ĐẶT KIBANA TRÊN CENTOS 7
Cài đặt key xác minh GPG public của kibana
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
Tạo repo kibana
vi /etc/yum.repos.d/kibana.repo
Chèn nội dung sau vào file vừa tạo
[kibana-8.x]
name=Kibana repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
Cài đặt kibana:
sudo yum install kibana

Bật cho phép kibana tự khỏi động lại khi reboot:
systemctl daemon-reload
systemctl enable kibana.service
Lệnh start stop restart kibana:
systemctl start kibana.service
systemctl stop kibana.service
File nội dung của kibana.yml
Tham khảo: https://github.com/hoangthien10021998/elk_8.7.1/blob/main/kibana.yml

[root@localhost ~]# grep -Ev '^#|^$' /etc/kibana/kibana.yml
server.port: 5601
server.host: "172.16.10.60"
xpack.screenshotting.browser.chromium.disableSandbox: true
xpack.encryptedSavedObjects.encryptionKey: 7fd43b18f726f3dec06f786658e3942b
xpack.reporting.encryptionKey: 3943d75122062711c692f8702b5e438f
xpack.security.encryptionKey: 8bb87efc1a336e25af131e76611d4844
server.publicBaseUrl: "http://172.16.10.60:5601"
elasticsearch.hosts: ["https://172.16.10.52:9200", "https://172.16.10.53:9200","https://172.16.10.54:9200",]
elasticsearch.username: "kibana_system"
elasticsearch.password: "Sds@123"
elasticsearch.ssl.certificateAuthorities: [ "/etc/kibana/certs/http_ca.crt" ]
logging:
  appenders:
    file:
      type: file
      fileName: /var/log/kibana/kibana.log
      layout:
        type: json
  root:
    appenders:
      - default
      - file
pid.file: /run/kibana/kibana.pid
[root@localhost ~]#
Tăng ram kibana tại file /etc/kibana/node.options
Chỉnh --max-old-space-size=8192
Mà ta muốn ở đây là 8GB=8192
CÀI ĐẶT Logstash TRÊN CENTOS 7

Tạo repo cho logstash.
vi /etc/yum.repos.d/logstash.repo
Chèn nội dung sau vào file và lưu lại
[logstash-8.x]
name=Elastic repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
Sau đó add key xác minh của repo vào
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
Cài đặt logstash bằng lệnh.
sudo yum install logstash

Cấu hình input: file cấu hình tại /etc/logstash/conf.d/02-beats-input.conf, phần này sẽ cấu hình để nó nhân đầu vào do Beats gửi đến cổng beats, thực hiện lệnh sau để tạo file 02-beats-input.conf
echo 'input {
  beats {
    host => "IP_logstash"
    port => 5044
  }
}' > /etc/logstash/conf.d/02-beats-input.conf

 
Cấu hình đầu ra, file cấu hình tại /etc/logstash/conf.d/30-elasticsearch-output.conf, phần này sẽ cấu hình sau khi Logstash nhận dữ liệu đầu vào từ Beats, nó xử lý rồi gửi đến Elasticsearch (localhost:9200). Thực hiện lệnh để tạo file 30-elasticsearch-output.conf
output {
  elasticsearch {
    hosts => ["https://172.16.10.52"]
    ssl => true
    ssl_certificate_verification => true
    cacert => "/etc/logstash/http_ca.crt"
    user => "elastic"
    password => "acaOlETomF*9=_4-gO2T"
    manage_template => false
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
} > /etc/logstash/conf.d/30-elasticsearch-output.conf

Ngoài ra nếu muốn lọc các log, định dạng lại các dòng log ở dạng dễ đọc, dễ hiểu hơn thì cấu hình filter tại file /etc/logstash/conf.d/10-syslog-filter.conf, ví dụ sau là cấu hình định dạng lại cấu trúc system log, lấy theo hướng dẫn tại document của Logstash
echo 'filter {
  if [fileset][module] == "system" {
    if [fileset][name] == "auth" {
      grok {
        match => { "message" => ["%{SYSLOGTIMESTAMP:[system][auth][timestamp]} %{SYSLOGHOST:[system][auth][hostname]} sshd(?:\[%{POSINT:[system][auth][pid]}\])?: %{DATA:[system][auth][ssh][event]} %{DATA:[system][auth][ssh][method]} for (invalid user )?%{DATA:[system][auth][user]} from %{IPORHOST:[system][auth][ssh][ip]} port %{NUMBER:[system][auth][ssh][port]} ssh2(: %{GREEDYDATA:[system][auth][ssh][signature]})?",
                  "%{SYSLOGTIMESTAMP:[system][auth][timestamp]} %{SYSLOGHOST:[system][auth][hostname]} sshd(?:\[%{POSINT:[system][auth][pid]}\])?: %{DATA:[system][auth][ssh][event]} user %{DATA:[system][auth][user]} from %{IPORHOST:[system][auth][ssh][ip]}",
                  "%{SYSLOGTIMESTAMP:[system][auth][timestamp]} %{SYSLOGHOST:[system][auth][hostname]} sshd(?:\[%{POSINT:[system][auth][pid]}\])?: Did not receive identification string from %{IPORHOST:[system][auth][ssh][dropped_ip]}",
                  "%{SYSLOGTIMESTAMP:[system][auth][timestamp]} %{SYSLOGHOST:[system][auth][hostname]} sudo(?:\[%{POSINT:[system][auth][pid]}\])?: \s*%{DATA:[system][auth][user]} :( %{DATA:[system][auth][sudo][error]} ;)? TTY=%{DATA:[system][auth][sudo][tty]} ; PWD=%{DATA:[system][auth][sudo][pwd]} ; USER=%{DATA:[system][auth][sudo][user]} ; COMMAND=%{GREEDYDATA:[system][auth][sudo][command]}",
                  "%{SYSLOGTIMESTAMP:[system][auth][timestamp]} %{SYSLOGHOST:[system][auth][hostname]} groupadd(?:\[%{POSINT:[system][auth][pid]}\])?: new group: name=%{DATA:system.auth.groupadd.name}, GID=%{NUMBER:system.auth.groupadd.gid}",
                  "%{SYSLOGTIMESTAMP:[system][auth][timestamp]} %{SYSLOGHOST:[system][auth][hostname]} useradd(?:\[%{POSINT:[system][auth][pid]}\])?: new user: name=%{DATA:[system][auth][user][add][name]}, UID=%{NUMBER:[system][auth][user][add][uid]}, GID=%{NUMBER:[system][auth][user][add][gid]}, home=%{DATA:[system][auth][user][add][home]}, shell=%{DATA:[system][auth][user][add][shell]}$",
                  "%{SYSLOGTIMESTAMP:[system][auth][timestamp]} %{SYSLOGHOST:[system][auth][hostname]} %{DATA:[system][auth][program]}(?:\[%{POSINT:[system][auth][pid]}\])?: %{GREEDYMULTILINE:[system][auth][message]}"] }
        pattern_definitions => {
          "GREEDYMULTILINE"=> "(.|\n)*"
        }
        remove_field => "message"
      }
      date {
        match => [ "[system][auth][timestamp]", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
      }
      geoip {
        source => "[system][auth][ssh][ip]"
        target => "[system][auth][ssh][geoip]"
      }
    }
    else if [fileset][name] == "syslog" {
      grok {
        match => { "message" => ["%{SYSLOGTIMESTAMP:[system][syslog][timestamp]} %{SYSLOGHOST:[system][syslog][hostname]} %{DATA:[system][syslog][program]}(?:\[%{POSINT:[system][syslog][pid]}\])?: %{GREEDYMULTILINE:[system][syslog][message]}"] }
        pattern_definitions => { "GREEDYMULTILINE" => "(.|\n)*" }
        remove_field => "message"
      }
      date {
        match => [ "[system][syslog][timestamp]", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
      }
    }
  }
}
' > /etc/logstash/conf.d/10-syslog-filter.conf
Thực hiện lệnh sau để xác định xem cấu hình có lỗi gì không
sudo -u logstash /usr/share/logstash/bin/logstash --path.settings /etc/logstash -t
Có thông báo Config Validation Result: OK. là được
Có thể cần mở cổng 5044 ở trên để nó nhận dữ liệu từ Server khác
firewall-cmd --permanent --add-port=5044/tcp
firewall-cmd --reload
Kích hoạt dịch vụ
systemctl enable logstash
systemctl start logstash

Cấu hình nội dung file /etc/logstash/logstash.yml
Tham khảo: https://github.com/hoangthien10021998/elk_8.7.1/blob/main/logstash.yml

[root@localhost conf.d]# grep -Ev '^#|^$' /etc/logstash/logstash.yml
path.data: /var/lib/logstash
path.logs: /var/log/logstash
xpack.monitoring.enabled: true
xpack.monitoring.elasticsearch.username: logstash_system
xpack.monitoring.elasticsearch.password: Sds@123
xpack.monitoring.elasticsearch.hosts: ["https://172.16.10.52:9200", "https://172.16.10.53:9200", "https://172.16.10.54:9200"]
xpack.monitoring.elasticsearch.ssl.certificate_authority: "/etc/logstash/http_ca.crt"
xpack.monitoring.elasticsearch.sniffing: true
[root@localhost conf.d]#
 
Phần user và pass của logstash_system ta vào Phần Stack management
 
Chọn User
 
 
Chọn vào user cần đổi pass
 
Phần change password ta đổi thành pass phù hợp và nhập lên file
 
 
Cấu hình file output ta upload file http_ca.crt lấy từ 1 trong các node của cụm cluster vào trên server logstash để xác thực
output {
  elasticsearch {
    hosts => ["https://172.16.10.52"]
    ssl => true
    ssl_certificate_verification => true
    cacert => "/etc/logstash/http_ca.crt"
    user => "elastic"
    password => "acaOlETomF*9=_4-gO2T"
    manage_template => false
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
 
Kết quả:
 
 
security-basic-setup-for-elastic
trên node1:
cd /usr/share/elasticsearch/bin/
[root@node1 bin]# ./elasticsearch-certutil ca
1 enter
2 nhập 123456123456
 








Tạo cert và key cho từng node trong cluster
1 nhập 123456123456
2 enter
3 nhập 123456123456
./elasticsearch-certutil cert --ca elastic-stack-ca.p12
 
Tại /usr/share/elasticsearch/ ta có 2 file
 
222 Tải về cả 2 và upload vào /etc/elasticsearch/certs/ và chown quyền
[root@node1 certs]# chown root:elasticsearch elastic-certificates.p12
[root@node1 certs]# chown root:elasticsearch elastic-stack-ca.p12
Và lệnh: chmod 660 elastic-certificates.p12 elastic-stack-ca.p12
Vào file vi /etc/elasticsearch/elasticsearch.yml
keystore.path: certs/elastic-certificates.p12
truststore.path: certs/elastic-certificates.p12
1.	Nếu bạn đã nhập mật khẩu khi tạo chứng chỉ nút, hãy chạy các lệnh sau để lưu mật khẩu trong kho khóa Elaticsearch:
/usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password

/usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password

ấn y va nhap pass 123456123456
Làm theo bước 222 Tải về cả 2 và upload vào /etc/elasticsearch/certs/ và chown quyền Cho các node 2 và 3
Sau khi xong ta thực hiện start stop trên 3 node
[root@node1 ~]# systemctl stop elasticsearch
[root@node1 ~]# systemctl start elasticsearch
[root@node1 ~]# systemctl status elasticsearch
 
Mã hoá giao tiếp clients cho Elastic
[root@node1 ~]# cd /usr/share/elasticsearch/bin/
[root@node1 bin]# ./elasticsearch-certutil http
Lệnh này tạo một .zip tệp chứa chứng chỉ và khóa để sử dụng với Elaticsearch và Kibana. Mỗi thư mục chứa một README.txt giải thích cách sử dụng các tệp này.

Khi được hỏi liệu bạn có muốn tạo CSR hay không, hãy nhập n.
Khi được hỏi liệu bạn có muốn sử dụng một CA hiện có hay không, hãy nhập y.
Nhập đường dẫn đến CA của bạn. Đây là đường dẫn tuyệt đối đến elastic-stack-ca.p12tệp mà bạn đã tạo cho cụm của mình.
Nhập mật khẩu cho CA của bạn.
Nhập giá trị hết hạn cho chứng chỉ của bạn. Bạn có thể nhập thời hạn hiệu lực theo năm, tháng hoặc ngày. Ví dụ: nhập 90D trong 90 ngày.
Khi được hỏi liệu bạn có muốn tạo một chứng chỉ cho mỗi nút hay không, hãy nhập y.

Mỗi chứng chỉ sẽ có khóa riêng và sẽ được cấp cho một tên máy chủ hoặc địa chỉ IP cụ thể.

Khi được nhắc, hãy nhập tên của nút đầu tiên trong cụm của bạn. Sử dụng cùng tên nút mà bạn đã sử dụng khi tạo chứng chỉ nút .
Nhập tất cả tên máy chủ được sử dụng để kết nối với nút đầu tiên của bạn. Các tên máy chủ này sẽ được thêm dưới dạng tên DNS trong trường Tên thay thế chủ đề (SAN) trong chứng chỉ của bạn.

Liệt kê mọi tên máy chủ và biến thể được sử dụng để kết nối với cụm của bạn qua HTTPS.

Nhập địa chỉ IP mà khách hàng có thể sử dụng để kết nối với nút của bạn.
Lặp lại các bước này cho mỗi nút bổ sung trong cụm của bạn.
Ví dụ:
Generate a CSR? [y/N]N
Use an existing CA? [y/N]y
CA Path: /usr/share/elasticsearch/elastic-stack-ca.p12
Password for elastic-stack-ca.p12: 123456123456
For how long should your certificate be valid? [5y] 5y
Generate a certificate per node? [y/N]y
Enter all the hostnames that you need, one per line.
When you are done, press <ENTER> once more to move on to the next step.

node1
Is this correct [Y/n]y
Enter all the IP addresses that you need, one per line.
When you are done, press <ENTER> once more to move on to the next step.

10.100.170.50
You entered the following IP addresses.

 - 10.100.170.50
Is this correct [Y/n]y
Do you wish to change any of these options? [y/N]N
Generate additional certificates? [Y/n]Y
Tiếp tục khai báo node 2 và node 3 tương tự
Đến phần node 3
Generate additional certificates? [Y/n]n
Nhập pass 123456123456
Provide a password for the "http.p12" file:  [<ENTER> for none]
Repeat password to confirm:
Kết thúc ta có ssl tại
What filename should be used for the output zip file? [/usr/share/elasticsearch/elasticsearch-ssl-http.zip]

Zip file written to /usr/share/elasticsearch/elasticsearch-ssl-http.zip
 
Ta tải về và giải nén ra sẽ được các tệp như sau
 
Upload file http.p12 trong elasticsearch\node1 vào /etc/elasticsearch/certs/
Và chạy lệnh :chown root:elasticsearch http.p12
Phân quyền: chmod 777 http.p12
Chạy tiếp:
[root@node1 certs]# cd /usr/share/elasticsearch/
[root@node1 elasticsearch]# /usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
Setting xpack.security.http.ssl.keystore.secure_password already exists. Overwrite? [y/N]y
Enter value for xpack.security.http.ssl.keystore.secure_password:
[root@node1 elasticsearch]#
Pass là 123456123456
Tương tự cho node còn lại


 
Mã hoá traffic giữa kibana và elastic ta dùng tệp elasticsearch-ca.pem
 
Ta upload file vào /etc/kibana
 
chmod 777 elasticsearch-ca.pem
vào sửa file kibana.yml
tìm đến dòng elasticsearch.ssl.certificateAuthorities: [ "/etc/kibana/http_ca.crt" ]
và sửa thành elasticsearch.ssl.certificateAuthorities: [ "/etc/kibana/ elasticsearch-ca.pem " ]
 
Mã hoá giữa Trình duyệt và kibana
Tạo csr và key
[root@node1 elasticsearch]# /usr/share/elasticsearch/bin/elasticsearch-certutil csr -name kibana-server
Please enter the desired output file [csr-bundle.zip]: ấn enter
Ta có file tại Certificate signing requests have been written to /usr/share/elasticsearch/csr-bundle.zip
Unzip file vừa tạo
[root@node1 elasticsearch]# unzip /usr/share/elasticsearch/csr-bundle.zip
 
Ký chứng chỉ bằng csr và key
[root@node1 elasticsearch]# /usr/share/elasticsearch/bin/elasticsearch-certutil cert --pem -ca /usr/share/elasticsearch/elastic-stack-ca.p12 -name kibana-server
1 Pass là 123456123456
2 là enter
3 là vị trí cert

 
Giải nén tập tin
[root@node1 elasticsearch]# unzip /usr/share/elasticsearch/certificate-bundle.zip
Archive:  /usr/share/elasticsearch/certificate-bundle.zip
  inflating: kibana-server/kibana-server.crt
replace kibana-server/kibana-server.key? [y]es, [n]o, [A]ll, [N]one, [r]ename: y
  inflating: kibana-server/kibana-server.key
[root@node1 elasticsearch]#
Và ta có cert:  
 
Upload 3 file này lên /etc/kibana/
 
Sửa file kibana.yml
Tìm đến dòng
server.ssl.enabled: false
server.ssl.certificate: /path/to/your/server.crt
server.ssl.key: /path/to/your/server.key
và sửa thành
Lưu và restart lại kibana
Và nhớ khỏi động lại elastic lần lượt 3 node
[root@node1 elasticsearch]# systemctl restart elasticsearch
[root@node1 elasticsearch]# systemctl status elasticsearch
● elasticsearch.service - Elasticsearch
   Loaded: loaded (/usr/lib/systemd/system/elasticsearch.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/elasticsearch.service.d
           └─override.conf
   Active: active (running) since Sat 2023-05-06 10:52:12 +07; 1min 19s ago
     Docs: https://www.elastic.co
 Main PID: 22874 (java)
   CGroup: /system.slice/elasticsearch.service
           ├─22874 /usr/share/elasticsearch/jdk/bin/java -Xms4m -Xmx64m -XX:+UseSerialGC -Dcli.name=server -Dcli.script=/usr/share/elasticsearch...
           ├─22937 /usr/share/elasticsearch/jdk/bin/java -Des.networkaddress.cache.ttl=60 -Des.networkaddress.cache.negative.ttl=10 -Djava.secur...
           └─22967 /usr/share/elasticsearch/modules/x-pack-ml/platform/linux-x86_64/bin/controller

May 06 10:51:19 node1 systemd[1]: Starting Elasticsearch...
May 06 10:52:12 node1 systemd[1]: Started Elasticsearch.
 
Tạo ssl dùng giao tiếp với logstash
Create the CA:
openssl genrsa -aes256 -out ca.key 4096
openssl req -key ca.key -new -x509 -days 7300 -sha256 -extensions v3_ca -out ca.crt

Create server certificate and key:
openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr
openssl x509 -req -days 3650 -in server.csr -out server.crt -CA ca.crt -CAkey ca.key -set_serial 1

Create client certificate and key:
openssl genrsa -out client.key 2048
openssl req -new -key client.key -out client.csr
openssl x509 -req -days 3650 -in client.csr -out client.crt -CA ca.crt -CAkey ca.key -set_serial 1

Java needs PKCS8 keys:
openssl pkcs8 -topk8 -in server.key -nocrypt -inform PEM -outform PEM -out server.pk8
=============================================
trên filebeats:
output.logstash:
  hosts: ["logs.mycompany.com:5044"]
  ssl.certificate_authorities: ["/etc/ca.crt"]
  ssl.certificate: "/etc/client.crt"
  ssl.key: "/etc/client.key"

trên /etc/logstash/conf.d/nginx_error.conf
input {
  beats {
    port => 5044
    ssl => true
    ssl_certificate_authorities => ["/etc/ca.crt"]
    ssl_certificate => "/etc/server.crt"
    ssl_key => "/etc/server.key"
    ssl_verify_mode => "force_peer"
  }
}
-----
 
Trước tiên ta Tạo SSL trên logstash để filebeat connect to logstash
Tham khảo: https://github.com/hoangthien10021998/elk_8.7.1/blob/main/tao%20ssl%20beat%20to%20logstash.txt 
Ta có 8 file
 


Cài đặt filebeat và đẩy syslog lên elastic
Tạo repo: vi /etc/yum.repos.d/filebeats.repo
Nhập nội dung vào:
[elastic-8.x]
name=Elastic repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
Lưu lại và thoát ra:
Cài đặt filebeat bằng lệnh: yum install -y filebeat
Tiến hành cấu hình filebeat, ta coppy file config ban đầu:
cp /etc/filebeat/filebeat.yml /etc/filebeat/filebeat.yml.old
sửa nội dung file: vi /etc/filebeat/filebeat.yml
Tham khảo: https://github.com/hoangthien10021998/elk_8.7.1/blob/main/filebeat.yml ( sửa lại các giá trị cho phù hợp)
Tiếp đến vào sửa module system:
Ta enable module: filebeat modules enable system
Sửa: vi /etc/filebeat/modules.d/system.yml
Tham khảo file config: https://github.com/hoangthien10021998/elk_8.7.1/blob/main/system.yml
Xong ta khởi động filebeat trước
systemctl enable filebeat
systemctl start filebeat
Tiếp đến ta config cho logstash
Tạo 1 file logstash
vi /etc/logstash/conf.d/syslog.conf
Tham khảo: https://github.com/hoangthien10021998/elk_8.7.1/blob/main/syslog.conf
Xong ta restart lại logstash
systemctl restart logstash
Kiểm tra xem đã nhận được log hay chưa
https://10.100.170.50:9200/_cat/indices/?v
nhập user pass của elastic vào”
ta thấy elastic đã có log
 
Vào kibana tạo
 
Chọn Data Views
 
Chọn Create
 
Sau đó save lại
 
Chon Discover
 
 
Chọn đến Index mà ta tạo
 
Kết quả:
 

