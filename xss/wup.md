# Lab 1: Reflected XSS into HTML context with nothing encoded

>  This lab contains a simple reflected cross-site scripting vulnerability in the search functionality.To solve the lab, perform a cross-site scripting attack that calls the alert function.

>  reflected xss xảy ra khi đoạn mã độc hại được thực thi ngay lập tức và chỉ một lần thông qua các tham số hoặc các yêu cầu HTTP. Thường là link để click thực thi ngay lập tức mã độc

=> hạn chế là victim cần phải đăng kí vào 1 thông tin nền tảng gì đó thì thực thi mới có hiệu quả 

+ goal: để web hiện ra cảnh báo khi input đầu vào trên web


> step 1 : recon

  + chức năng của trang web này hiện đầu tiên là search khi ta nhập vào 1 thông tin tìm kiếm thì nó sẽ phản hồi lại input ta đã nhập

![image](https://github.com/j10nelop/m3d1r/assets/152776722/e31bd4a8-2526-4ff1-bea5-69f89eccb751)

 => ta thấy thông tin search ta nhập được reflected lại kết quả được input ta đã nhập 

 + bây giờ test thêm 1 thẻ tag ```<h1>hello friend</h1>``` nếu nó xử lí thẻ này tức là lỗ hổng xss có thể chèn được vào trong html qua user nhập vào chức năng đó

![image](https://github.com/j10nelop/m3d1r/assets/152776722/3fb87953-fff1-42a2-a238-261fdbbcf1ac)

=> nó không trả về tag kèm content mà nó đã xử lí tại html đó nên ta có thể chèn script tag vào hệ thống 


> Exploit

payload 

      <script>alert("hello friend")</script>

![image](https://github.com/j10nelop/m3d1r/assets/152776722/2e429c42-a5a8-4660-b47d-79bb9350e1e8)

> solution cho bài này

=> với dạng này cơ bản ta có html encoding ví dụ 

``` <p>&lt;h1&gt;Hello Friend&lt;/h1&gt;</p>```

=> điều đó hệ thông sẽ nhìn ra đó là 1 chuỗi và in ra cái ta nhập vào thay vì thấy nó là 1 function và thực thi nó.


# Lab 2: Stored XSS into HTML context with nothing encoded

>  This lab contains a stored cross-site scripting vulnerability in the comment functionality.To solve this lab, submit a comment that calls the alert function when the blog post is viewed. 

> store xss là 1 dạng xss dùng liên tục được hay còn gọi là Persistent XSS, xảy ra khi đoạn mã độc hại (mã JavaScript hoặc HTML) được lưu trữ và thực thi trên server hoặc trong cơ sở dữ liệu của ứng dụng web

=>  ưu điểm so với reflected là khả năng thành công cao hơn vì nó đã lưu trữ tại page đó của 1 trang web 


+ goal: thực thi alert tại trang web đó hiện thông báo

> step 1 : Recon

+ sau 1 hồi check qua chức năng ta thấy cần check tính năng post của trang web này.

  + ta thấy 1 số post-comment đã được lưu trữ trong trang web từ lâu
 
![image](https://github.com/j10nelop/m3d1r/assets/152776722/af79ce4c-bc93-40d5-b63e-9fbe9f5203db)


=> như thường lệ ta kiểm tra từng tính năng 1 trong trang web này có nhận tag script tag chưa được xử lí trên html không  

  + có 4 input đầu vào trong đó comment là reflected text của người dùng trong đó nên ta có thể check qua tag html xem hệ thống có xử lí tag không

check script tag ```<h1>hello</h1>``` => chức năng Comment 

![image](https://github.com/j10nelop/m3d1r/assets/152776722/0670e5e8-cedf-47ad-9d51-168975877f08)

=> vậy ta đã dự đoán được đoạn input bị chứa mã độc xss khi ta input html tag vào hệ thống và được xử lí raw làm cho text to hơn 

> Exploit

payload:

```
<img src=x onerror=alert("hacked")> 
```

+ <img> : là image tag hiển thị ảnh

+ src=x onerror : là src là địa chỉ nhưng vì không tồn tại => onerror xử lí lỗi

![image](https://github.com/j10nelop/m3d1r/assets/152776722/ce1b3df0-23d8-49be-85b1-a9119f3a58a6)

=> nó sẽ hiển thị thông báo mỗi khi refesh page này -> nó khai thác nhưng user access vào page này 

> solution

=> tương tự như lab 1 vì vẫn chưa encode html 


# Lab 3:  DOM XSS in document.write sink using source location.search

> This lab contains a DOM-based cross-site scripting vulnerability in the search query tracking functionality. It uses the JavaScript document.write function, which writes data out to the page. The document.write function is called with data from location.search, which you can control using the website URL.

> To solve this lab, perform a cross-site scripting attack that calls the alert function.

> Dom xss: xảy ra khi mã độc được chèn vào trang web bằng cách sử dụng các tài nguyên không được lưu trữ trên máy chủ, mà được tải từ máy chủ và xử lý trên trình duyệt của người dùng. (Script độc hại tồn tại trong client-side code) 


 + step 1 : Recon

   - trong dom có đoạn mã js đề cập đến

   ```
       <script>
                        function trackSearch(query) {
                            document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');
                        }
                        var query = (new URLSearchParams(window.location.search)).get('search');
                        if(query) {
                            trackSearch(query);
                        }
                    </script>
   ```

   + sink : document.write trong tracksearch function
  
  => query không được thoát đúng cách 

> solution cơ bản

        // Escape dữ liệu trước khi sử dụng
        var safeQuery = encodeURIComponent(query);
        document.write('<img src="/resources/images/tracker.gif?searchTerms=' + safeQuery + '">');

> Exploit:

payload: 

    "><script>alert("hello")</script>

    or

    "><svg onload=alert(1)>

    or

    bla" onload="alert()

final payload:

      "><img src=x onerror=alert("hello")>

sau khi chèn payload thì cấu trúc sẽ là thế này ```document.write('<img src="/resources/images/tracker.gif?searchTerms=' + query + '">');```

=> svg: à một định dạng hình ảnh vector dựa trên XML được sử dụng để hiển thị đồ họa hai chiều trên web. Không giống như các định dạng hình ảnh raster (như JPEG hoặc PNG) dựa trên pixel, đồ họa vector được tạo ra bằng các hình học như điểm, đường, và hình khối, cho phép chúng mở rộng mà không bị mất chất lượng

![image](https://github.com/j10nelop/Pr1vate/assets/152776722/b6eddd84-1c74-4664-9ca5-d0eb16711a18)

=> solve lab 3

# Lab 4: DOM XSS in innerHTML sink using source location.search

>  This lab contains a DOM-based cross-site scripting vulnerability in the search blog functionality. It uses an innerHTML assignment, which changes the HTML contents of a div element, using data from location.search.

> To solve this lab, perform a cross-site scripting attack that calls the alert function.

+ step 1: Recon

  - check script tag trên chức năng search thì có vẻ script tag không được thực thi vì đây là innerHTML sink có cơ chế bảo vệ khỏi script tag
 
  - document write sink thì không
 
  => có cách nào để chèn tag mà có thể thực thi được không ? có

  => sink trong này là

  ```
     <script>
                            function doSearchQuery(query) {
                                document.getElementById('searchMessage').innerHTML = query;
                            }
  ```
=> gây ra lỗ hổng dom xss

+ step 2: Exploit

  -payload:

  ```
   <img src=x onerror = alert()>
  ```

![image](https://github.com/j10nelop/Pr1vate/assets/152776722/1ab360fb-de03-407e-ab5e-2628979d1847)

=> solve lab 4 

# Lab 5: DOM XSS in jQuery anchor href attribute sink using location.search source

>  This lab contains a DOM-based cross-site scripting vulnerability in the submit feedback page. It uses the jQuery library's $ selector function to find an anchor element, and changes its href attribute using data from location.search.

> To solve this lab, make the "back" link alert document.cookie.

+ step 1 : Recon

```
$(function() {
                                $('#backLink').attr("href", (new URLSearchParams(window.location.search)).get('returnPath'));
                            });
```
  
  - check dòng code jquery hàm function đang gọi # là id backlink herf là đường link trỏ về => ta có thể lợi dụng để chèn và returnpath herf

- jQuery là một thư viện JavaScript phổ biến, được thiết kế để đơn giản hóa việc viết mã JavaScript và tương tác với Document Object Model (DOM) của HTML. jQuery cung cấp nhiều tính năng giúp bạn dễ dàng xử lý các tác vụ phổ biến như thao tác DOM, xử lý sự kiện, hiệu ứng và Ajax.

+ step 2: Exploit

  - payload

    ```
     https://domain?returnPath=javascript:alert(document.cookie)
    ```
  
javascript:alert() là một cách sử dụng JavaScript để hiển thị một thông báo cảnh báo (alert) trên trình duyệt web

![image](https://github.com/j10nelop/Pr1vate/assets/152776722/eacddf90-9bd3-479c-a1fb-bcc52eb81890)

=> chức năng back đã bị chèn xss alert

=> solve lab 5

# Lab 6: DOM XSS in jQuery selector sink using a hashchange event

>  This lab contains a DOM-based cross-site scripting vulnerability on the home page. It uses jQuery's $() selector function to auto-scroll to a given post, whose title is passed via the location.hash property.

> To solve the lab, deliver an exploit to the victim that calls the print() function in their browser. 

+ step 1: Recon

  
