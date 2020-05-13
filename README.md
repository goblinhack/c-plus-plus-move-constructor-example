Example use of the move and copy constructors with a custom class
=================================================================

Expected output:
```
 c: [10][11]
 copy ctor called
 copy of c: [10][11]
 move ctor called
 moved c: [10][11]
```

Compile as:
```
  g++ -std=c++2a -O2 -Wall -pedantic foo.cpp
```

Code:
```C++
#include <iostream>
#include <algorithm>

template<class T> class MyVector {
private:
    T *data;
    size_t maxlen;
    size_t currlen;
public:
    MyVector<T> () : data (nullptr), maxlen(0), currlen(0) { }
    MyVector<T> (int maxlen) : data (new T [maxlen]), maxlen(maxlen), currlen(0) { }

    MyVector<T> (const MyVector& o) {
        std::cout << "copy ctor called" << std::endl;
        data = new T [o.maxlen];
        maxlen = o.maxlen;
        currlen = o.currlen;
        std::copy(o.data, o.data + o.maxlen, data);
    }

    MyVector<T> (const MyVector<T>&& o) {
        std::cout << "move ctor called" << std::endl;
        data = o.data;
        maxlen = o.maxlen;
        currlen = o.currlen;
    }

    void push_back (const T& i) {
        if (currlen >= maxlen) {
            maxlen *= 2;
            auto newdata = new T [maxlen];
            std::copy(data, data + currlen, newdata);
            if (data) {
                delete[] data;
            }
            data = newdata;
        }
        data[currlen++] = i;
    }

    friend std::ostream& operator<<(std::ostream &os, const MyVector<T>& o) {
        auto s = o.data;
        auto e = o.data + o.currlen;;
        while (s < e) {
            os << "[" << *s << "]";
            s++;
        }
        return os;
    }
};

int main() {
    auto c = new MyVector<int>(1);
    c->push_back(10);
    c->push_back(11);
    std::cout << "c: " << *c << std::endl;
    auto d = *c;
    std::cout << "copy of c: " << d << std::endl;
    auto e = std::move(*c);
    delete c;
    std::cout << "moved c: " << e << std::endl;
}
```
