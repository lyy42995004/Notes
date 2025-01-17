# main.cc

初始化server，运行server

# web_server.h

定义WebServer

```C++
class WebServer {
public:
    WebServer(int port, int tigger_mode, int timeout_MS, int sql_port,
              const char *sql_user, const char* sql_password
              const char* database_name, int connect_pool_name
              int thread_num, bool open_log, int log_level
              int log_queue_size);
    ~WebServer();
    
  	void start();

private:
    static int SetFDNonblock();
    
    bool InitSocket();
    void InitEventMode(int trigger_mode);
    void AddClient(int fd, sockaddr_in addr);
    
    void DealListen();
    void DealWrite(HttpConnect *client);
    void DealRead(HttpConnect *client);
    
    void SendError(int fd, const char *info);
    void ExtentTime(HttpConnect *client);
    void CloseConnect(HttpConnect *client);
    
    void OnRead(HttpConnect *client);
    void OnWrite(HttpConnect *client);
    void OnProcess(HttpConnect *client);
   	    
    static const int MAX_FD = 65536;

    int port_; 					// 端口号
    bool open_linger_;			// 是否开启
    int timeout_MS_;			// 存储时间
    int listen_FD_;				// 监听套接字
    char *source_directory_;	// 服务器资源文件目录路径
    
    uint32_t listen_event_;		// 监听事件
    uint32_t connect_evernt_;	// 连接事件
    
    std::unique_ptr<HeapTimer> timer_;
    std::unique_ptr<ThreadPool> thread_pool_;
    std::unique_ptr<Epoller> epllor_;
    std::unordered_map<int, HttpConnect> users_;
}
```

# http_connect.h

读写数据，调用httprequest来解析数据，调用httpresponse来生成响应

```C++
class HttpConnect {
public:
	HttpConnect();
    ~HttpConnect();
private:
    int fd
}
```

