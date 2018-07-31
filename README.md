nginx-upload-module
===================

This module is based on Nginx upload module (v 2.2.0) http://www.grid.net.ru/nginx/upload.en.html. ...
Since it seems the author has not maintained that module. I changed some codes that can be installed with latest nginx.

- install
./configure --add-module={module_dir} && make && make install

- conf
```
server {
    listen       80;
    client_max_body_size 100m;

    location / {
        root  html/upload;
    }

    # Upload form should be submitted to this location
    location /upload {
        # Pass altered request body to this location
        upload_pass   /example.php;

        # Store files to this directory
        # The directory is hashed, subdirectories 0 1 2 3 4 5 6 7 8 9 should exist
        upload_store /tmp/upload 1;

        # Allow uploaded files to be read only by user
        upload_store_access user:r;

        # Set specified fields in request body
        upload_set_form_field "${upload_field_name}_name" $upload_file_name;
        upload_set_form_field "${upload_field_name}_content_type" $upload_content_type;
        upload_set_form_field "${upload_field_name}_path" $upload_tmp_path;

        # Inform backend about hash and size of a file
        upload_aggregate_form_field "${upload_field_name}_md5" $upload_file_md5;
        upload_aggregate_form_field "${upload_field_name}_size" $upload_file_size;

        upload_pass_form_field "^submit$|^description$";
    }

    location ~ \.php$ {
        root           html/upload;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```


  nginx官方推荐的第三方上传模块nginx-upload-module在nginx-1.3.9后已经不能编译通过，究其原因是因为nginx-1.3.9废弃了ngx_http_request_body_t中的to_write成员指针。

        github上有人解决了这个问题，参考https://github.com/vkholodkov/nginx-upload-module/issues/41的讨论，davromaniak给出了解决方案。但是davromaniak的解决方案有些冗余，而hongzhidao给出更加清晰的解决方案，参考https://github.com/hongzhidao/nginx-upload-module。测试该方案没有问题，但是有个遗留的问题：在上传文件时，必须手动创建/tmp/0~/tmp/9目录，用于存放上传的文件。但是，如果在配置文件中配置upload_store /tmp 1 2 3;时，手动创建目录是很麻烦的，现在我在hongzhidao的基础上解决了这个问题，上传文件时不用手动创建存放上传文件的目录，另外我还修复了代码中限速失效的bug，参考https://github.com/winshining/nginx-upload-module。
