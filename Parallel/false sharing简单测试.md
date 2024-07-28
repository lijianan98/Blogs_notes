``` c++
#include <cstdlib>
#include <iostream>
#include <memory>
#include <thread>
#include <vector>
#include <chrono>

using namespace std;

int arr[16];

void foo(int idx) {
    for (int i = 0; i < 100000; i++) {
        arr[idx] += rand() % 5;
    }
}

int main()
{
    using std::chrono::high_resolution_clock;
    using std::chrono::duration_cast;
    using std::chrono::duration;
    using std::chrono::milliseconds;
    
    std::vector<std::thread> thread_pool;
    //cout<<"Hello World";
    auto t1 = high_resolution_clock::now();
    
    for (int i = 0; i < 16; i++) {
        thread_pool.push_back(std::thread(foo, i));
    }
    for (auto &th : thread_pool) {
        th.join();
    }
    
    auto t2 = high_resolution_clock::now();
    auto ms_int = duration_cast<milliseconds>(t2 - t1);
    std::cout << ms_int.count() << "ms" << std::endl;
    
    return 0;
}
```