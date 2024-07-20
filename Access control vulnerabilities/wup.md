# Lab 1: Unprotected admin functionality

>  This lab has an unprotected admin panel.

> Solve the lab by deleting the user carlos

> Access control vulnerabilities là lỗ hổng do người dùng cấu hình có thể chưa phân quyền giữa admin và user gây ra user có thể thao tác qua 1 số chức năng để khai thác lỗ hổng đó

![346409600-83d093c2-ae95-48dc-934c-eef8dacc68c2](https://github.com/user-attachments/assets/68b02eb0-265e-45e2-aecf-1c4c720a60c9)



+ step 1: Recon

  - khi vào trang web ta thấy có chức năng về login nhưng có vẻ chưa tìm được thông tin nhiều về admin
 
  - checking url khi ta đổi path trên đó thì API display không hợp lệ (not found)  

  => còn 1 cách đó là vào path robots.txt -> chứa thông tin mà google không muốn cho bạn biết có thể đó là thông tin ẩn hoặc path ẩn

![346409628-68ccc921-5b3e-43a5-81cc-e8c1b657f70d](https://github.com/user-attachments/assets/10e108e4-672d-4c11-9ffe-8ae736bc9612)


+ step 2 : Exploit

  - vì đã có path là /administrator-panel nên ta sẽ gán vào url => ra được tài khoản admin

![346409876-dff3b778-f080-45f3-9ac8-22707f5a9925](https://github.com/user-attachments/assets/4d1f09a1-f5ee-4074-aff3-3327d473dd18)


 
=> do người dùng cấu hình không chặn path đó bên phía client để không thể tương tác qua display web 

=> solve lab 1 


# Lab 2: Unprotected admin functionality with unpredictable URL

>  This lab has an unprotected admin panel. It's located at an unpredictable location, but the location is disclosed somewhere in the application.

> Solve the lab by accessing the admin panel, and using it to delete the user carlos. 

![346414424-71c27a3e-71ee-4624-a0a2-55147562fe55](https://github.com/user-attachments/assets/4ec82c73-afc8-4582-aa55-fb8a849b9539)



+ step 1: Recon

  - trong này khi áp dụng robots.txt như bài trước thì api not found -> hệ thống đã được cấu hình chặn đường dẫn đến robots.txt
 
  - khi check thông tin trong source code ta nhận được mã javascript ẩn bên trong đó
 
  ```
      <script>
  var isAdmin = false;
  if (isAdmin) {
     var topLinksTag = document.getElementsByClassName("top-links")[0];
     var adminPanelTag = document.createElement('a');
     adminPanelTag.setAttribute('href', '/admin-g5a9bz');
     adminPanelTag.innerText = 'Admin panel';
     topLinksTag.append(adminPanelTag);
     var pTag = document.createElement('p');
     pTag.innerText = '|';
     topLinksTag.appendChild(pTag);
  }
  </script>
  ```

  => adminPanelTag tạo ra link và được setAttriute đến path /admin-g5a9bz

  => createElement('a') tạo ra thẻ a là link

+ step 2 : Exploit

  - dựa vào thông tin về link ta chèn path admin vào url thì ta được tài khoản admin -> xóa thông tin về carlos
 
![346417229-b9fba82e-8d27-4af8-abb4-6781558f06b6](https://github.com/user-attachments/assets/e2a47b76-6e19-4a71-bcc8-27f4ac55dd05)


=> solve lab 2

# Lab 3: User role controlled by request parameter

>  This lab has an admin panel at /admin, which identifies administrators using a forgeable cookie.

> Solve the lab by accessing the admin panel and using it to delete the user carlos.

> You can log in to your own account using the following credentials: wiener:peter

![346418652-9accd2a8-c9d3-484e-9f3d-8d394a310429](https://github.com/user-attachments/assets/062b94eb-a713-4197-bf6d-d322a0e16bb3)


+ step 1: Recon

  - ta đươc cấp username và password của client là wiener:peter
 
  - đăng nhập ta check wiener id thay bằng admin thì không hợp lệ -> out ra login page
 
  - check cookie ta thấy có thông tin về Admin dang để là false
 
![346419812-1f874b35-0e77-4ffe-aa96-d8d0d60f2973](https://github.com/user-attachments/assets/75a96648-bd01-4d6b-b553-e5bc31cdb2c1)


+ step 2: Exploit

  - thay thế false thành true và reload lại page ta sẽ có chức năng admin-panel tại user acc -> xóa tài khoản carlos

![346420003-c6ac0d3c-6a31-4195-bbe3-7c2870a9809d](https://github.com/user-attachments/assets/3be87f61-c3e7-4bb3-836b-5648ec5802bb)


=> solve lab 3

# Lab 4: User role can be modified in user profile

>  This lab has an admin panel at /admin. It's only accessible to logged-in users with a roleid of 2.

> Solve the lab by accessing the admin panel and using it to delete the user carlos.

> You can log in to your own account using the following credentials: wiener:peter

+ step 1: Recon:

  - login account -> check thông tin bằng brupsuite
 
  - check account ta có thêm thông tin là update email
 
![346424203-f965cd8b-f92c-4493-b027-302fb6e2e2b5](https://github.com/user-attachments/assets/4db29b6f-3afb-4ad9-abf8-f97ab5734bd3)


+ check trong brupsuite ta nhận được thông tin trước khi bị forward

![346424410-1f895a46-e831-4aa4-9019-35ff4f794bca](https://github.com/user-attachments/assets/5bdd76fc-1188-4a8b-8700-e893a8d27df9)


=> roleid có thể có thay đổi từ 1 thành 2 

sau khi thêm roleid : 2 vào change email ta được admin-panel -> xóa  tài khoản carlos 

![346424919-302f9aad-df59-4000-8f38-f0ba8617bdbe](https://github.com/user-attachments/assets/528d71c2-5142-41eb-8e06-36cb15208572)

=> solve lab 4

# Lab 5: User ID controlled by request parameter 

>  This lab has a horizontal privilege escalation vulnerability on the user account page.

> To solve the lab, obtain the API key for the user carlos and submit it as the solution.

> You can log in to your own account using the following credentials: wiener:peter

+ step 1: Recon

  - đăng nhập tài khoản wiener => ta nhận được thông tin về API account đó
 
  - ta cần chú ý thông tin trên url có thông tin về id ta vừa đăng nhập account là wiener 
 
![346822158-ae457015-4f53-4e15-a4df-d26e62432633](https://github.com/user-attachments/assets/24a25189-8c4b-4eaf-951d-66b94ef0de08)


   => sau khi test tài khoản thành admin thì nó ra page login:(( -> nên ta sẽ thử với tên  carlos kết quả :

![346822745-69043d9a-a999-435e-900a-10c6b73d6fe1](https://github.com/user-attachments/assets/641d04a7-a64e-4413-949c-55e8561a0882)


=> cho ra api của account -> dù admin acc đã phân quyền không cho client vào account của admin nhưng với các client khác thì chưa phân quyền 

=> solve lab 5

# Lab 6: User ID controlled by request parameter, with unpredictable user IDs

> This lab has a horizontal privilege escalation vulnerability on the user account page, but identifies users with GUIDs.

> To solve the lab, find the GUID for carlos, then submit his API key as the solution.

> You can log in to your own account using the following credentials: wiener:peter

> GUID là một định danh duy nhất được gán cho người dùng để phân biệt họ với những người dùng khác thường format dưới dạng hexdecimal

+ step 1: Recon

  + ta sẽ login vào account -> nó lại ra API của tài khoản nhưng lần này có vẻ là account id đã chuyển format thành dạng GUID được biểu diễn dưới dạng hexadecimal 
 
![346824281-3df573b4-9993-48e9-ac7b-303ed11ececf](https://github.com/user-attachments/assets/e353ce56-7f54-4edc-8404-41b3f444e659)




  + theo đề bài cần xác định GUID của user khác cụ thể là carlos

  + trong lúc tìm kiếm tui đã nhận ra được trong bài post có thể chứa 1 số thông tin về tài khoản khác up lên

![346825357-4fc7da3b-8550-4b00-9607-4861dfbc69f2](https://github.com/user-attachments/assets/9553b8a0-3db5-472a-95fd-76d24e7a3290)


 => view 1 post ta thấy 1 tên user là carlos 

+ step 2 : exploit

  - sau khi truy cập vào tên đăng bài viết carlos thì tui nhận được nó link tới url chứa uid của carlos
 
![346825680-15a116f7-e7d6-48c7-bcbd-f1dbf074805f](https://github.com/user-attachments/assets/defdd3c1-92ca-4645-9e7b-162353cb329e)


=> uid carlos là 2ebc7967-05e0-4ef7-b5ba-95a3e910be44

![346826098-044a5e41-a129-4365-85e6-aa43ffa4a09f](https://github.com/user-attachments/assets/09a44994-19e0-4c11-8bc0-044d1d64cba9)


=> solve lab 6

# Lab 7: User ID controlled by request parameter with data leakage in redirect

> This lab contains an access control vulnerability where sensitive information is leaked in the body of a redirect response.

> To solve the lab, obtain the API key for the user carlos and submit it as the solution.

> You can log in to your own account using the following credentials: wiener:peter

+ step 1: Recon

   - đăng nhập vào account lần này thì có vẻ web đã không cho đổi được id carlos hiển thị trên màn hình -> out page ra login vì thay đổi 

![346832379-383cfacf-477e-48ad-87ee-da9cdf31d5da](https://github.com/user-attachments/assets/db1fd3f0-5377-47f3-aa2a-a75503f30cb7)


  => sau khi check brup suite nó có vẻ đã chặn ở endpoint như khi redirect nó lại leak thông tin api mà ta có thể thay đổi được 

![346832723-826773f9-816f-472d-8c77-6db2d15c2dad](https://github.com/user-attachments/assets/75b4581b-2cfe-4a27-9bda-01e9e586c01b)


=> nó đã leak ở method 302 là redirect đến trang đích 

=> solve lab 7

# Lab 8: User ID controlled by request parameter with password disclosure

> This lab has user account page that contains the current user's existing password, prefilled in a masked input.

> To solve the lab, retrieve the administrator's password, then use it to delete the user carlos.

> You can log in to your own account using the following credentials: wiener:peter

+ solve lab
  
  - sau khi đăng nhập vào account wiener check đề bài là access vào tài khoản administrator -> id=wiener đang bị vuln có thể thay thế bằng administrator
 
![346837463-8d5cfe56-e418-4b96-a786-d8192166cd11](https://github.com/user-attachments/assets/8d04b20a-5b5e-4f46-a2da-b612c151d855)


  => ta có thể vào được admin đề bài là lấy password của admin thì có 2 cách 1 là dùng burp,  2 là dùng f12 nên ta có

![346837846-87d5b5a8-4b63-4bac-bab2-2153f61504cb](https://github.com/user-attachments/assets/af7b1583-86c3-4fce-9135-d11331c2962e)


=> value của tài khoản password : ftg4lp386hfej597pc4a 

![346838001-737d51a1-ab59-4f50-9e39-fd4374607ec3](https://github.com/user-attachments/assets/abea3cd4-38ef-483c-adbc-e66a68383eb6)


=> có pasword ta có thể chuyển sang admin account để -> xóa carlos 

=> solve lab 8

# Lab 9: Insecure direct object references
 
> This lab stores user chat logs directly on the server's file system, and retrieves them using static URLs.

> Solve the lab by finding the password for the user carlos, and logging into their account.

![346928663-5bcea6dd-1864-4c6a-ad40-97d46010a8b9](https://github.com/user-attachments/assets/155b69d4-d7ff-4b62-b65e-d31d43ce6cc1)


+ solve lab:

  - trong lab này xuất hiện chức năng live chat có vẻ ta có thể send cho còn bot chat này 1 số tin nhắn
 
  - view transcript là chức năng cho mình tải về những tin nhắn đẫ nhắn với bot -> có vẻ tin nhắn có 1 id gì đó trong từng tin nhắn
 
![346934329-1e383a32-9488-4509-b182-714546bdee19](https://github.com/user-attachments/assets/35b5b2f4-18a8-4009-8459-b1b6186562f0)


 => tin nhắn bắt đầu từ 2.txt và có vẻ nó đang tăng dần đến disconected

![346934605-d6a1d330-07c2-4bab-b2d6-e28cb6f45602](https://github.com/user-attachments/assets/6d6216de-d3d4-429b-9647-54f59966229b)


=> nhận định đây là lỗ hổng IDOR cho phép ta thay đổi id là những number khác để xem được thông tin khác 

=> ta lùi xuống id là 0.txt thì ta thấy k có gì 

![346935114-8e3a449f-cc0a-44ff-8385-a819e7db7dea](https://github.com/user-attachments/assets/d51946d9-b052-42fd-af1a-836f65997fe7)


- cho đến khi lui xuống 1.txt thì ta có 1 dòng tin nhắn ẩn lịch sử của cuộc trò chuyện đã được nhắn 

![346935356-b9342f17-1504-41a4-a67c-98d7dc4f3b37](https://github.com/user-attachments/assets/7bf5d2dd-5a50-49e8-aef2-fbacf7ce9264)


=> vì khi bắt đầu nhắn tin thì hệ thông đã setup lịch sử bắt đầu từ 2.txt 

=> tin nhắn này chứa password của carlos với Hal Pline

- ta có username:carlos và password:rrgsaprpnt8tgtcvbt57 -> login vào account carlos

=> solve lab 9

# Lab 10: URL-based access control can be circumvented

> This website has an unauthenticated admin panel at /admin, but a front-end system has been configured to block external access to that path. However, the back-end application is built on a framework that supports the X-Original-URL header.

> To solve the lab, access the admin panel and delete the user carlos
