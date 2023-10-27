## cpp-httplib

https://github.com/yhirose/cpp-httplib

```cpp
namespace httplib
{ 
    struct MultipartFormData 
    { 
        std::string name; 
        std::string content; 
        std::string filename; 
        std::string content_type; 
    }; 
    using MultipartFormDataItems = std::vector<MultipartFormData>; 
    struct Request 
    { 
        std::string method;//存放请求方法
        std::string path;//存放请求资源路径
        Headers headers;//存放头部字段的键值对map 
        std::string body;//存放请求正文
        // for server 
        std::string version;//存放协议版本
        Params params;//存放url中查询字符串 key=val&key=val的 键值对map 
        MultipartFormDataMap files;//存放文件上传时，正文中的文件信息
        Ranges ranges; 
        bool has_header(const char *key) const;//判断是否有某个头部字段
        std::string get_header_value(const char *key, size_t id = 0) const;//获取头部字段值
        void set_header(const char *key, const char *val);//设置头部字段
        bool has_file(const char *key) const;//文件上传中判断是否有某个文件的信息
        MultipartFormData get_file_value(const char *key) const;//获取指定的文件信息
    }; 

    struct Response 
    { 
        std::string version;//存放协议版本
        int status = -1;//存放响应状态码
        std::string reason; 
        Headers headers;//存放响应头部字段键值对的map 
        std::string body;//存放响应正文
        std::string location; // Redirect location重定向位置
        void set_header(const char *key, const char *val);//添加头部字段到headers中
        void set_content(const std::string &s, const char *content_type);//添加正文到body中
        void set_redirect(const std::string &url, int status = 302);//设置全套的重定向信息
    }; 
    
    class Server 
    { 
        using Handler = std::function<void(const Request &, Response &)>;//函数指针类型
        using Handlers = std::vector<std::pair<std::regex, Handler>>;//存放请求-处理函数映射
        std::function<TaskQueue *(void)> new_task_queue;//线程池
        Server &Get(const std::string &pattern, Handler handler);//添加指定GET方法的处理映射
        Server &Post(const std::string &pattern, Handler handler); 
        Server &Put(const std::string &pattern, Handler handler); 
        Server &Patch(const std::string &pattern, Handler handler); 
        Server &Delete(const std::string &pattern, Handler handler); 
        Server &Options(const std::string &pattern, Handler handler); 
        bool listen(const char *host, int port, int socket_flags = 0);//开始服务器监听
        bool set_mount_point(const std::string &mount_point, const std::string &dir, 
                             Headers headers = Headers());//设置http服务器静态资源根目录
    }; 
}
```

