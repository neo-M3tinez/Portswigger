# Lab 1: File path traversal, simple case

> This lab contains a path traversal vulnerability in the display of product images.

> To solve the lab, retrieve the contents of the /etc/passwd file.

> path traversal hay còn gọi là directory traversal, là một kỹ thuật tấn công an ninh mạng cho phép kẻ tấn công truy cập vào các thư mục và tệp trên máy chủ web mà thông thường họ không được phép truy cập

> Absolute Path Đường dẫn tuyệt đối chỉ rõ vị trí của một tệp hoặc thư mục từ gốc của hệ thống tệp tin (ví dụ :/var/www/html/documents/file.txt) => cụ thể nên có thể truy từ thư mục gốc

> Relative Path Đường dẫn tương đối chỉ rõ vị trí của một tệp hoặc thư mục dựa trên thư mục hiện tại (ví dụ : /var/www/html có thể trong đó có có thể /documents/file.txt) => nên phải xác định cấu trúc thư mục và tìm


![347007052-e09ef178-b23f-4571-9e10-b483331d8c9d](https://github.com/user-attachments/assets/6e046ab5-11bc-4fd1-a66a-22b1f03735cc)


+ step 1: Recon

  - để khai thác được lỗ hổng path traversal trước hết ta cần check đường dẫn nào khả thi để khai thác
 
  - dấu hiệu ta có thể tìm được ở đây là img?filename = ?.jpg
 
![347007675-2ff780a3-4632-4275-8dcf-c078ac8d2d5b](https://github.com/user-attachments/assets/4a2d5775-5680-45dd-8a50-3eb018570e43)


=> khi ta thay đổi parameter của filename thì ảnh trong web cũng thay đổi theo từc ta có thể tương tác với file image qua url img?filename 

 - nguyên nhân gây ra lỗ hổng này có thể do chưa được lọc an toàn và sử dụng hàm đầu vào chưa an toàn ví dụ chưa được xử lí kí tự đặc biệt như "../../" ... 

     + lỗ hổng này chạy trong root dir tức là "var/www/html/..file" vì không được xử lí an toàn nên có bị thực thi tại server theo request client send lên
  
     + code không an toàn ví dụ
  
              $file = $_GET['file'];
              include('/var/www/html/' . $file);


              $file = $_POST['file'];
              $content = file_get_contents('/var/www/html/' . $file);
              echo $content;

+ step 2: Exploit

  - ta sẽ test đường dẫn trên img?filename=?.jpg
   
  - payload:

    ```
        img?filename = ../../../etc/passwd
    ```

    + ../:  là lùi thư mục đến vị trí cần đến (relative path) đường dẫn tương đối
   
    + etc/passwd: thông tin về /etc/passwd gồm user của hệ thống server

![347048339-4ed46a33-7619-4fe6-a450-f4f346f9118a](https://github.com/user-attachments/assets/1c7f2b8b-e6c8-4d22-a802-845b91aab45c)


  => solve lab 1 

# Lab 2: File path traversal, traversal sequences blocked with absolute path bypass

> This lab contains a path traversal vulnerability in the display of product images.

> The application blocks traversal sequences but treats the supplied filename as being relative to a default working directory.

> To solve the lab, retrieve the contents of the /etc/passwd file

![347121887-304e3a6c-a597-486a-8ea7-ddd20a5f99df](https://github.com/user-attachments/assets/f1c6344b-6cb3-4bae-b65e-d24f8fe66b6d)



+ step 1: Recon

  - trong lab này web lỗ hổng vẫn ở parameter filename -> vì lỗ hổng này vẫn bị khai thác tại filename root directory
 
  - khi test ../../../etc/passwd không hợp lệ -> vì hệ thống đã limit process directory thì chỉ (reletive-path) không hợp lệ 
 
![347120416-eee5f382-32aa-4f1a-bbee-c59c2bffad92](https://github.com/user-attachments/assets/51b96d65-0fe3-40fa-a19c-5ff91f3f963c)


 => ta có thể sử dụng absolute path vì nó dựa vào thư mục root để truy xuất thay vì thư mục hiện tại 

 + step 2: Exploit

   - payload:

         img?filename=/etc/passwd

![347124198-2db04a9e-ef9f-41f3-840b-ae0344eecbe1](https://github.com/user-attachments/assets/ac11e349-7162-4f2e-9774-b31dda90d77b)



=> solve lab 2

# Lab 3: File path traversal, traversal sequences stripped non-recursively

>  This lab contains a path traversal vulnerability in the display of product images.

> The application strips path traversal sequences from the user-supplied filename before using it.

> To solve the lab, retrieve the contents of the /etc/passwd file

![347125518-86dc5821-fc7a-4dde-8d01-80648c19fd8e](https://github.com/user-attachments/assets/4f0cb78d-5a8f-4478-b7d3-2df3f6f83ec3)


 
+ step 1: Recon

  - lỗ hổng vẫn bị khai thác ở filename parameter
 
  - trong bài này có vẻ nó đã chặn ../ special character nên ta sẽ thử 1 số trường hợp ...//...//...// or .././..././
 
+ step 2: Exploit

  - payload:
 
        img?filename = ...//...//...//etc/passwd

        img?filename = ..././..././..././etc/passwd

![347135836-8d3ae387-29bd-44f6-b3cc-7a8db600d4cd](https://github.com/user-attachments/assets/6e4b15d8-a7ba-43ed-85da-99a5851077d6)


=> ...// : hệ thông không filter trường hợp này vì (../ và / đều bị chặn ) 

=> solve lab 3

# Lab 4: File path traversal, traversal sequences stripped with superfluous URL-decode

>  This lab contains a path traversal vulnerability in the display of product images.

> The application blocks input containing path traversal sequences. It then performs a URL-decode of the input before using it.

> To solve the lab, retrieve the contents of the /etc/passwd file

![347142037-1531d1a1-b23f-4758-ac7c-820aff43f66a](https://github.com/user-attachments/assets/306b9ee1-91f2-4b82-830c-34b911af2422)


+ step 1: Recon

  - trong bài này ta có thể khai thác lỗ hổng filename
 
  - bài này ta có thông tin đề bài là khai thác path traversal bằng cách sử dụng URL encode với payload ../../

 - theo đề bài là hệ thông sẽ đọc và giải mã kí tự đặc biệt được encode URL  rồi decode ở endpoint

+ step 2 : Exploit

  - payload:
 
        img?filename = ..%252f..%252f..%252fetc%252fpasswd

=> payload phải encode 2 lần 

![347143369-11fec63b-b0be-430c-8e74-17e95f5a462d](https://github.com/user-attachments/assets/c07666ce-fcb8-4892-bca8-192b1051afb1)


=> endpoint sẽ decode URL 2 lần để run trên web server 

=> solve lab 4


# Lab 5: File path traversal, validation of start of path

>  This lab contains a path traversal vulnerability in the display of product images.

> The application transmits the full file path via a request parameter, and validates that the supplied path starts with the expected folder.

> To solve the lab, retrieve the contents of the /etc/passwd file.

![347147714-541daed6-c227-4d71-a0d5-25d339405434](https://github.com/user-attachments/assets/aab1d07b-b428-4fe5-9f88-7a02bb171da2)


+ step 1: Recon

  - thông tin trên parameter filename có thêm 1 đường dẫn root là: "/var/www/html/?.jpg"
 
  - đây là root path có chứa file image => ta cần bắt đầu từ root path đề khai thác path traversal từ file chứa image

+ step 2 : Exploit

  - payload:

        img?filename="/var/www/html/../../../etc/passwd"

=> ta sẽ validate root path di chuyển sang thư file /etc/passwd từ nơi chứa file image 

![347148885-5677ad58-f572-4095-8cb9-a36882ba6091](https://github.com/user-attachments/assets/ab15e59e-1df1-446b-a1f5-41545afac736)


=> solve lab 5 


# Lab 6: File path traversal, validation of file extension with null byte bypass

> This lab contains a path traversal vulnerability in the display of product images.

> The application validates that the supplied filename ends with the expected file extension.

> To solve the lab, retrieve the contents of the /etc/passwd file

![347149781-063f737f-7577-402c-aba1-bad3132839a8](https://github.com/user-attachments/assets/97f79468-f3e9-4d79-80c8-80e368018585)


+ step 1: Recon

  - khai thác lỗ hổng qua img?filename=
 
  - trong bài này khi dùng ../../../ để khai thác thì không tìm thấy file có thể là do web đang checking extension của file image payload được chèn vào
 
  - test payload :
 
        img?filename=../../../etc/passwd.jpg

 ![347151260-88785933-ff15-4f1a-af43-61b715382a4c](https://github.com/user-attachments/assets/f63bc068-6058-471b-b61f-e98a84abcb65)

=> có vẻ nó thấy file.jpg là file là không hợp lệ nên ta sẽ sử dụng null byte injection để ngắt strings là tên file đằng trước extension để bypass

+ step 2: Exploit

  - payload:

        img?filename = ../../../etc/passwd%00.jpg 

![347152188-ec73d3a0-02c0-4644-bebc-0385c76aa5c6](https://github.com/user-attachments/assets/654058d3-34ed-4199-8269-327623e41ce4)


=> %00 null byte để khi hệ thống quét xong /etc/passwd thì sẽ quét đến extension là .jpg 

=> solve lab 6
