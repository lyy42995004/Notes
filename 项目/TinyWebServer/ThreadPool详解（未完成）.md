```C++
class ThreadPool {
public:
    ThreadPool() = default;
    ThreadPool(ThreadPool&&) = default;
    explicit ThreadPool(int thread_count = 8) : pool_(std::make_shared<Pool>()) {
        assert(thread_count > 0);
        for (int i = 0; i < thread_count; ++i) {
            std::thread([this]() {
                std::unique_lock<std::mutex> locker(pool_->mtx_);
                while (true) {
                    if (!pool_->tasks_.empty()) {   // 有任务，取任务执行
                        auto task = std::move(pool_->tasks_.front());
                        pool_->tasks_.pop();
                        locker.unlock();
                        task();
                        locker.lock();
                    } else if (!pool_->is_closed) { // 未关闭，等待
                        pool_->cond_.wait(locker);
                    } else {                        // 已关闭，退出 
                        break;
                    }
                }
            }).detach();
        }
    }

    ~ThreadPool() {
        if (pool_) {
            std::lock_guard<std::mutex> locker(pool_->mtx_);
            pool_->is_closed = true;
        }
        pool_->cond_.notify_all();
    }

    template<class T>
    void AddTask(T&& task) {
        std::lock_guard<std::mutex> locker(pool_->mtx_);
        pool_->tasks_.push(std::forward<T>(task));
        pool_->cond_.notify_one();
    }

private:
    struct Pool {
        bool is_closed;
        std::mutex mtx_;
        std::condition_variable cond_;
        std::queue<std::function<void()>> tasks_; // 任务队列
    };
    std::shared_ptr<Pool> pool_; 
};
```

