[giải nén()][3] là 1 hàm bổ sung cho hàm  [nén()][4] – nó chuyển dữ liệu nhị phân thành các 1 mảng liên kết dựa theo định dạng cụ thể. Nó phần nào giống với _sprintf_, chuyển dữ liệu chuỗi dựa vào định dạng cho trước. 2 hàm này cho phép chúng ta đọc và biết các bộ đêm dữ liệu nhị phân từ các định dạng chuỗi cụ thể. Nó cho phép chúng ta dễ dàng chuyển đổi dữ liệu với các chương trình được viết trong cách ngôn ngữ hay định dạng khác. Xem 1 ví dụ nhỏ sau đây.


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

Mặc dù điều này có vẻ hơi khó hiểun, phần tiếp theo cung cấp 1 vị dụ cụ thể.
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

Dòng quan trọng cần lưu lại là dòng đặc tả định dạng. Kí tự 'A6' làm hàm giari nén() lấy 6 byte đầu tiên của dữ liệu và biên dịch thành 1 chuỗi. Dữ liệu nhận được sau đó sẽ được lưu trữ trong 1 mảng liên kết với khóa là 'version'.


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

Ví dụ trên khi chạy sẽ in ra dòng bên như bên dứoi

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
