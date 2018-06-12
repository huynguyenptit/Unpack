# Giải nén dữ liệu nhị phân trong PHP

Hiếm khi chúng ta được yêu cầu làm việc với các file nhị phân trong PHP.Tuy nhiên khi cần thiết thì hàm 'pack' và 'unpack' của PHP lại giúp ích rất nhiều. Chúng ta sẽ bắt đầu với 1 vấn đề lập trình, điều này sẽ giữ cho bài viết được nhất quán liền mạch về ngữ cảnh của bài viết. Vấn đề ở đây là : chúng ta muốn viết 1 hàm nhận biến là file ảnh và cho chúng ta biết đó có phải là file ảnh GIF hay không; bất kể là file ảnh có đuôi file thế nào. Chúng ta sẽ không sử dụng bất cứ hàm thư viện GD nào.

#### Header của 1 file GIF

Với yêu cầu không được sử dụng bất kỳ hàm trong thư viện đồ họa nào, để giải quyết vấn đề này chúng ta cần phải lấy những dữ liệu liên quan từ chính file GIF. Không giống như HTML hay XML hay các file định dạng văn bản khác, 1 file GIF và hầu hết các định dạng ảnh khác được lưu ở định dạng nhị phân.Hầu hết các file nhị phân có header ở trên đầu file chứa thông tin  meta  về file cụ thể. Chúng ta có thể sử dụng thông tin này để biết được loại file và những thứ khác, như là chiều cao và chiều rộng trong trường hợp file GIF. 1 header GIF thô điển hình sẽ như bên dưới, sử dụng cái trình soạn thảo hex như là [WinHex][1]. 

![][2]

Mô tả chi tiết về header ở bên dưới.

| ----- |
| 
    
    
    Offset   Length   Contents
      0      3 bytes  "GIF"
      3      3 bytes  "87a" or "89a"
      6      2 bytes  
      8      2 bytes  
     10      1 byte   bit 0:    Global Color Table Flag (GCTF)
                      bit 1..3: Color Resolution
                      bit 4:    Sort Flag to Global Color Table
                      bit 5..7: Size of Global Color Table: 2^(1+n)
     11      1 byte   
     12      1 byte   
     13      ? bytes  
             ? bytes  
             1 bytes   (0x3b)

 | 

Vì vậy để kiểm tra file ảnh có phải file GIF hợp lệ không, chúng ta cần kiểm tra 3 byte khởi đầu của header, nơi có dấu hiệu 'GIF', và 3 byte tiếp theo, là số phiên bản; là '87a' hoặc là '89a'. Những lúc này thì hàm unpack() là thực sự cần thiết. Trước khi chúng ta tìm giải pháp, nhìn qua xem hàm unpack() hoạt động thế nào.

#### Sử dụng hàm unpack() 

[unpack()][3] là 1 hàm bổ sung cho hàm  [pack()][4] – nó chuyển dữ liệu nhị phân thành các 1 mảng liên kết dựa theo định dạng cụ thể. Nó là một thứ gì đó xuyên suốt ở các dòng _sprintf_, chuyển dữ liệu chuỗi dựa vào định dạng cho trước. 2 hàm này cho phép chúng ta đọc và biết các bộ đêm dữ liệu nhị phân từ các định dạng chuỗi cụ thể. Nó cho phép chúng ta dễ dàng chuyển đổi dữ liệu với các chương trình được viết trong cách ngôn ngữ hay định dạng khác. Xem 1 ví dụ nhỏ sau đây.


| ----- |
| 
    
    
    $data = unpack('C*', 'codediesel');
    var_dump($data);

 | 

Nó sẽ in ra dòng sau đây, mã thập phân của 'codediesel' :


| ----- |
| 
    
    
    array
      1 => int 99
      2 => int 111
      3 => int 100
      4 => int 101
      5 => int 100
      6 => int 105
      7 => int 101
      8 => int 115
      9 => int 101
      10 => int 108

 | 

Trong ví dụ bên trên, tham số đầu tiên là định dạng chuỗi và cái thứ 2 là dữ liệu thực tế. Định dạng chuỗi đặc tả cách mà dữ liệu nên được chuyển đổi thành. Trong ví dụ này phần đầu tiên của định dạng 'C', cho biết chúng ta cần coi kí tự đầu tiên của dữ liệu như 1 byte không dấu. Phần tiếp theo '*', làm cho hàm áp dụng các mã định dạng đăc tả cho tất cả các kí tự còn lại.

Mặc dù điều này có vẻ hơi khó hiểu, phần tiếp theo cung cấp các ví dụ cụ thể.
#### Thu thập dữ liệu header

Dười đây là cách giải quyết cho bài toán GIF sử dụng hàm giải nén(). Hàm _is_gif()_ sẽ trả về true nếu file đưa vào ở định dạng GIF.

| ----- |
| 
    
    
    function is_gif($image_file)
    {
     
        /* Mở file ảnh ở chế độ nhị phân */
        if(!$fp = fopen ($image_file, 'rb')) return 0;
     
        /* Đọc 20 byte đầu file */
        if(!$data = fread ($fp, 20)) return 0;
     
        /* Tạo định dạng đặc tả */
        $header_format = 'A6version';  # Get the first 6 bytes
    
        /* Giải nén dữ liệu header */
        $header = unpack ($header_format, $data);
     
        $ver = $header['version'];
     
        return ($ver == 'GIF87a' || $ver == 'GIF89a')? true : false;
     
    }
     
    /* Run our example */
    echo is_gif("aboutus.gif");

 | 

Dòng quan trọng cần lưu lại là dòng đặc tả định dạng. Kí tự 'A6' làm hàm giải nén() lấy 6 byte đầu tiên của dữ liệu và biên dịch thành 1 chuỗi. Dữ liệu nhận được sau đó sẽ được lưu trữ trong 1 mảng liên kết với khóa là 'version'.


1 ví dụ khác được cho dưới đây. Nó có thêm 1 vài dữ liệu header của file GIF, bao gồm chiều rộng và chiều cao ảnh.

| ----- |
| 
    
    
    function get_gif_header($image_file)
    {
     
        /* Mở file ảnh dưới định dạng nhị phân */
        if(!$fp = fopen ($image_file, 'rb')) return 0;
     
        /* Đọc 20 byte đầu file */
        if(!$data = fread ($fp, 20)) return 0;
     
        /* Tạo định dạng đặc t */
        $header_format = 
                'A6Version/' . # Lấy 6 bytes đầu tiên
                'C2Width/' .   # Lấy 2 bytes tiếp theo
                'C2Height/' .  # Lấy 2 bytes tiếp theo
                'C1Flag/' .    # Lấy 1 byte tiếp theo
                '@11/' .       # Nhảy đến byte thứ 12
                'C1Aspect';    # Lấy 1 byte tiếp theo
    
        /*  Giải nén dữ liệu header */
        $header = unpack ($header_format, $data);
     
        $ver = $header['Version'];
     
        if($ver == 'GIF87a' || $ver == 'GIF89a') {
            return $header;
        } else {
            return 0;
        }
    }
     
    /* Chạy ví dụ */
    print_r(get_gif_header("aboutus.gif"));

 | 

Ví dụ trên sẽ in ra như dưới đây khi chạy

| ----- |
| 
    
    
    Array
    (
        [Version] => GIF89a
        [Width1] => 97
        [Width2] => 0
        [Height1] => 33
        [Height2] => 0
        [Flag] => 247
        [Aspect] => 0
    )

 | 

Tiếp theo chúng ta sẽ đi đến chi tiết cách mà đặc tả định dạng hoạt động. Tôi sẽ chia các định dạng, và cung cấp thông tin cho mỗi loại.

| ----- |
| 
    
    
    $header_format = 'A6Version/C2Width/C2Height/C1Flag/@11/C1Aspect';

 | 

| ----- |
| 
    
    
    A - Đọc 1 byte và biên dịch thành 1 chuỗi. 
       Số byte đọc được sẽ ở kế tiếp
    6 - Đọc tất cả 6 byte, từ vị trí 0
    Version - Tên của khóa trong mảng liên kết lưu trữ dữ liệu thu được từ 'A^'
   
     
    / - Bắt đầu dịnh dạng code mới
    C - Biên dịch dữ liệu tiếp theo thành byte không dấu
    2 - Đọc tất cả 2 bytes
    Width - Khóa trong mảng liên kết
     
    / - Bắt đầu dịnh dạng code mới
    C - Biên dịch dữ liệu tiếp theo thành byte không dấu
    2 - Đọc tất cả 2 bytes
    Height- Khóa trong mảng liên kết
     
    / - Bắt đầu dịnh dạng code mới
    C - Biên dịch dữ liệu tiếp theo thành byte không dấu
    1 - Đọc tất cả 2 bytes
    Flag - Khóa trong mảng liên kết
     
    / - Bắt đầu dịnh dạng code mới
    @ - Chuyển byte offset đặc tả bằng giá trị số theo sau 
        Nhớ rằng vị trí đầu tiên của chuỗi nhị phân là 0.
    11 - Chuyển tới vị trí 11
     
    / - Bắt đầu dịnh dạng code mới
    C - Biên dịch dữ liệu tiếp theo thành byte không dấu
    1 - Đọc tổng cộng 1 byte
    Aspect - Khóa trong mảng liên kết

 | 

Các lựa chọn định dạng thêm có thể tìm ở [đây][4]. Mậc dù tôi mới chỉ đưa 1 ví dụ nhỏ, nén/giải nén có thể xử lý những công việc phức tạp hơn được trình bày ở đây

Lưu ý: Từ PHP phiên bản 7.2.0  loại float và double hỗ trợ cả BIig Endian và Little Endian.

[1]: http://www.x-ways.net/winhex/index-m.html
[2]: http://www.codediesel.com/wp-content/uploads/2010/09/winhex.gif "winhex"
[3]: http://php.net/manual/en/function.unpack.php
[4]: http://www.php.net/manual/en/function.pack.php
