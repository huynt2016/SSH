#SSH
##I.Tổng quan
###1.SSH là gì?
- SSH (Secure Shell) là một giao thức mạng dùng để thiết lập kết nối dữ liệu một cách bảo mật giữa máy chủ và máy trạm.

###2.Đặc điểm của SSH?
- Bí mật
- Toàn vẹn
- Xác thực

###3.SSH làm gì?
- Đảm bảo thực hiện các lệnh từ xa an toàn
- Đảm bảo truyền file an toàn
- Tạo các tunnel truyền dữ liệu cho các ứng dụng dựa trên TCP/IP
  
##II.Cơ chế
[Tham khảo](https://www.youtube.com/watch?v=zlv9dI-9g1U&index=3&list=PLnXMmXQOsIRmr5V80BjCS7xjIV7YmzgTg)
###1.Kiến trúc bộ giao thức SSH?
- Phía Server: Giao thức SSH TLP (Transport Layer Protocol) - Đảm bảo tính xác thực, bí mật và toàn vẹn.
- Phía Client: Giao thức xác thực người dùng UAP (User Authentication Protocol)
- Giao thức kết nối: SSH CP (Connection Protocol)
  <ul>
  <li>Thiết lập các phiên kết nối</li>
  <li>Chạy trên cả hai giao thức dành cho server và client</li>
  </ul>
  
###2.Giao thức SSH Transport Layer
####a.Thiết lập kết nối
- Client khởi tạo kết nối đến cổng 22 của Server
- Khi kết nối được thiết lập, cả Client và Server sẽ gửi cho nhau thông tin phiên bản, đó là chuỗi ID dạng 

`SSH-protoversion-softwareversion comments`

####b.Trao đổi khóa
- Cả Client và Server đều có danh sách các thuật toán mã hóa. Mỗi phía thực hiện chọn một thuật toán và gửi khóa khởi tạo tương ứng cho phía bên kia.
  <ul>
  <li>Nếu trùng nhau thì thuật toán đó được sử dụng</li>
  <li>Nếu khác nhau thì thủ tục này được lặp lại ở phía Client</li>
  </ul>
  
###3.Giao thức và phương pháp xác thực người dùng
####a.Giao thức xác thực người dùng
- Được sử dụng để chạy trên giao thức tầng vận tải SSH
- Dịch vụ cho giao thức này là `ssh-userauth`. Khi bắt đầu thực hiện, nó nhận định phiên làm việc, nó được sử dụng cho việc ký chứng nhận người sở hữu khóa bí mật.
  <ul>
  <li>Client gửi yêu cầu xác thực `SSH_MSG_USERAUTH_REQUEST` mà không có phương pháp xác thực cụ thể. Yêu cầu này là bước lấy danh sách các phương pháp xác thực cụ thể.</li>
  <li>Server sẽ trả lời với thông điệp và kèm theo danh sách các phương pháp xác thực mà server hỗ trợ. Điều này cho phép server chủ động trong quá trính xác thực.</li>
  <li>Client có thể chọn phương pháp xác thực từ danh sách trên.</li>
  </ul>
- Thời gian chờ cho việc xác thực khoảng 10'. Có hạn định số lần đăng nhập không thành công trong một phiên, nếu vượt quá ngưỡng thì server sẽ ngắt kết nối.

####b.Phương pháp xác thực
- Phương pháp sử dụng khóa công khai
  <ul>
  <li>Được thực hiện bằng cách gửi chữ kỹ được tạo ra với khóa bí mật của người dùng</li>
  <li>Người dùng gửi yêu cầu lấy thuật toán khóa công khai sử dụng. Server sẽ từ chối yêu cầu nếu nó không hỗ trợ thuật toán đó</li>
  <li>Server gửi thông điệp `SSH_MSG_USERAUTH_PK_OK` nếu nó hỗ trợ</li>
  <li>Sau khi quyết định thuật toán sử dụng, người dùng sẽ gửi thông điệp đã được ký</li>
  <li>Server kiểm tra xem khóa và chữ ký có hợp lệ không. Nếu cả hai hợp lệ thì người dùng được xác thực.</li>
  </ul>
  
- Phương pháp sử dụng mật khẩu
  <ul>
  <li>Người dùng gửi gói như sau `SSH_MSG_USERAUTH_REQUEST`</li>
  <li>Mật khẩu được truyền đi trong gói ở dạng rõ nhưng toàn bộ gói được mã hóa bởi tầng vận tải</li>
  </ul>
  
- Phương pháp xác thực dựa trên định danh máy (host)
  <ul>
  <li>Được thực hiện qua việc client gửi chữ ký được tạo ra với khóa bí mật của client, và server kiểm tra bằng cách sử dụng khóa công khai của host đó.</li>
  <li>Một khi định danh máy client được thiết lập, sự xác thực sẽ được dựa trên tên người sử dụng.</li>
  </ul>
  
###4.Giao thức kết nối
- Cung cấp các phiên đăng nhập, thực hiện các lệnh từ xa, chuyển hướng các kết nối TCP/IP.
- Tất cả các kênh được ghép vào một tunnel được mã hóa.
- Được thiết kế để chạy trên tầng vận tải SSH và giao thức xác thực người dùng.

####a.Cơ cấu kênh truyền
- Tất cả các phiên làm việc dạng terminal, chuyển hướng kết nối là các kênh (channels)
- Những kênh này được định danh bởi các số
- Các yêu cầu mở một kênh chứa số kênh của người gửi

####b.Mở một kênh
- Khi một bên muốn mở kênh mới, nó gán một số cục bộ cho kênh, sau đó gửi thông điệp `SSH_MSG_CHANNEL_OPEN` đến bên kia
- Phía bên nhận quyết định mở kênh hoặc không và trả lời theo một trong hai cách sau

`SSH_MSG_CHANNEL_OPEN_CONFIRMATION`

`SSH_MSG_CHANNEL_OPEN_FAILURE`
  
####c.Truyền dữ liệu
- Truyền dữ liệu được thực hiện với định dạng thông điệp sau
`SSH_MSG_CHANNEL_DATA`

####d.Đóng kênh
- Khi không bên nào muốn gửi dữ liệu nữa nó sẽ gửi thông điệp
`SSH_MSG_CHANNEL_EOF`
- Khi một bên muốn đóng kênh nó sẽ gửi thông điệp
`SSH_MSG_CHANNEL_CLOSE`

####e.Phiên làm việc
- Một phiên làm việc là một sự thực thi một chương trình từ xa (một shell, một ứng dụng, một lệnh hệ thống ...)
- Nhiều phiên có thể thực hiện đồng thời
- Phiên được khởi tạo bằng thông điệp `SSH_MSG_CHANNEL_OPEN`
- Khi một phiên được thiết lập, một chương trình có thể được khởi tạo.
