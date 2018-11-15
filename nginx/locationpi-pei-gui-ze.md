

location匹配规则分为普通匹配和正则匹配 语法为：location [=|~|~*|^~] /uri/ { … }


1.正则匹配  
    “~ ”和“~* ”前缀表示正则location ，“~ ”区分大小写，“~* ”不区分大小写;
    
2.普通匹配
    前缀（包括：“=”，“^~ ”和“@ ”）和无任何前缀的都属于普通location;
    
    
1.nginx 先匹配普通location再匹配正则location。如果有正则匹配上就使用正则匹配，如果正则没有匹配上就使用普通location
2.我们也可以让nginx匹配到了普通location不再去搜索正则location。可以在前边添加 ^~ 或者= 符号，这两个符号都可以让nginx
匹配完普通location后不再去匹配正则location。 = 和^~的区别是 = 是全值匹配^~是最左前缀
3.但其实还有一种“隐含”的方式来阻止正则location 的搜索，这种隐含的方式就是：当“最大前缀”匹配恰好就是一个“严格精确（exact match ）”匹配，
照样会停止后面的搜索。

普通location的严格精确匹配>正则匹配>普通的非严格精确匹配

测试：

         ```
         L1:
         location / {
	    deny all;
         }
	 L2:
	 location ~ \.html$ {
            root   html;
            allow all;
         }
         
         http://127.0.0.1:8080/             403 Forbidden
         http://127.0.0.1:8080/index.html   Welcome to nginx!
         http://127.0.0.1:8080/inde.html    404 Not Found
         
         
         分析： 第一个url严格精确匹配到L1 因此不会再去匹配正则 所以走L1；
               第二个url 第一步先匹配到L1并且L1没有=或者^~符合 因此还会去匹配正则L2;
               第三个url和第二个url同理;
         
         ```

         ```
             L1:
             location ^~ / {
			deny all;
             }
             
             L2:
             location ~ \.html$ {
                 root html;
                 allow all;
             }
             
             http://127.0.0.1:8080/           403 Forbidden
             http://127.0.0.1:8080/index.html 403 Forbidden
             http://127.0.0.1:8080/indx.html  403 Forbidden

             由于普通location L1 有^~符号 因此3个url都满足最左前缀匹配到L1，并且不会再去匹配正则；
             
         
         ```


            ```
             L1:
             location = / {
			deny all;
             }
             
             L2:
             location ~ \.html$ {
                 root html;
                 allow all;
             }
             
             http://127.0.0.1:8080/           403 Forbidden
             http://127.0.0.1:8080/index.html  Welcome to nginx!
             http://127.0.0.1:8080/indx.html  404 Not Found

            这次L1用=修饰  只有第一个url完全匹配L1，第二个和第三个不满足完全匹配因此还会搜索正则匹配；
                
            
            ```
        
            ```
                location ^~ / {
			deny all;
                }

                location  /hello {
		       root   html;
                      allow all;
                }
                
                location ~ \.html$ {
                    root   html;
                    allow all;
                }
                
                http://127.0.0.1:8080/            403
                http://127.0.0.1:8080/index.html  403 
                http://127.0.0.1:8080/hello/index.html  404
                
                第一个和第二个url都在普通location中找到了L1 并且L1 用了^~ 修饰 不再去匹配正则location
                
                第三个url先命中L2,L2没有任何修饰，然后再去匹配正则location
            ```
        
            ```
                    location ^~ / {
			deny all;
		    }

                    location  /hello/index.html {
			root   html;
                       allow all;
                    }
		
		    location /hello  {
			root   html;
                       deny all;
		    }
		
		http://127.0.0.1:8080/  403
		http://127.0.0.1:8080/index.html  403
		http://127.0.0.1:8080/hello/index.html  404
		http://127.0.0.1:8080/hello  403 
		http://127.0.0.1:8080/hello/inde.html  403
		
               第一个和第二个url都先匹配到L1普通location 不再去匹配正则；
               第3个url精确匹配到L2
               第4个url精确匹配到L3
               第5个url最左前缀命中到L3
            ```
          

•正则location的匹配是按照编辑的先后顺序进行匹配，匹配后不再继续遍历剩余的正则location。

               
            
































