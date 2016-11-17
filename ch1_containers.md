# Ch1: Containers

## Item 1: Choose your containers with care

Standard STL sequence containers, standard STL associate containter, nonstandard sequence containers, nonstandard associative containers...

Contiguous-memory containers, Node-based containers.

## Item 2: Beware the illusion of container-independent code

It's unrealistic to write containter independent codes. Each container has its own expertise as well as APIs.

If you really want to do, use encapsulation.

## Item 3: Make copying cheap and correct for objects in containers

When you store object in container, make sure the object can be copied correctly.

In new standard, may be also moved!

Afraid of the overhead? Use container of pointers.

Still some headaches? Use contianer of smart pointers.

## Item 4: Call empty istead of checking size() against zero

Prefer to use empty instead of size.

For list, size() function may be O(n) time.

The reason is that if size is constant time, then some operations for list, such as splice need to be linear time.

## Item 5: Prefer range member functions to their single-element counterparts

```c++
v1.assign(v2.begin(), v2.size()/2, v2.end());
```

Reasons to prefer range member functions:

* It's generally less work to write the code using the range member functions.

* Range member functions tend to lead to code that is clearer and more straightforward.

Why not use copy

```c++
copy(data, data+numValues, inserter(v, v.begin()));
```

1. unnecessary function calls.

2. cost of inefficiently moving the existing elements in v to their final post-insertion positions.

3. unnecessary memory allocation.

What range member functions we have?

* Range construction

* Range insertion

* Range erasure

* Rnage assignment

## Item 6: Be alert for C++'s most vexing parse

```c++
ifstream dataFile("ints.dat");
list<int> data(istream_iterator<int>(dataFile), istream_iterator<int>());  // define a function

int f(double d);
int f(double (d));  // same as above, the parens are ignored
int f(double);  // the same
```

How to solve it?

```c++
// Option 1
list<int> data((istream_iterator<int>(dataFile)), istream_iterator<int>());

// Option 2
istream_iterator<int> dataBegin(dataFile);
istream_iterator<int> dataEnd;
list<int> data(dataBegin, dataEnd);
```

## Item 7: When using containers of newed pointers, remember to delete the pointers before the container is destroyed

Remember to delete the pointers before container is destroyed.

What if exception happens?

Use shared_ptr. RAII

## Item 8: Never create containers of auto_ptrs

Containers of auto_ptr (COAPs) are prohibited.

Automatically change ownership.

```c++
auto_ptr<Widget> pw1(new Widget);
auto_ptr<Widget> pw2(pw1); // ownership transfers to pw2
pw1 = pw2  // transfer to pw1
```

Consider sorting.

```c++
vector<auto_ptr<Widget>> widgets;
sort(widgets.begin(), widgets.end(), widgetAPCompare);
```

Quick sort for example, may create pivotValue. The ownership will transfer!

```c++
ElementType pivotValue(*i);
```

## Item 9: Choose carefully among erasing options

### Eliminate all objects that have a particular value

For vector, string or deuque, use the erase-remove idiom.

```c++
c.erase(remove(c.begin(), c.end(), 1963), c.end());
```

For list, use list::remove.

```c++
c.remove(1963);
```

For standard associative container, use its erase member function.

```c++
c.erase(1963);
```

### Eliminate all objects that satisfy a particular predicate

For vector, string or deque, use erase-remove_if idiom.

```c++
c.erase(remove_if(c.begin(), c.end(), badValue), c.end());
```

For list, remove_if.

```c++
c.remove_if(badValue);
```

For associative container, use remove_copy_if and swap or write a for loop.

```c++
// option 1
AssocContainer<int> goodValues;
remove_copy_if(c.begin(), c.end(), inserter(goodValues, goodValues.end()), badValue);
c.swap(goodValues);

// option 2
for (AssocContainer<int>::iterator i = c.begin(); i != c.end();) {
    if (badValue(*i))
        c.erase(i++);  // erase will invalidate the pointer, so use i++
    else
        ++i;
}
```
postincrement-the-iterator-you-pass-to-erase technique.

### To do something inside the loop.

For standard sequence container, write a loop, update your iterator with erase return values.

```c++
for (SeqContainer<int>::iterator i = c.begin(); i != c.end();) {
    if (badValue(*i)) {
        logFile << "Erasing" << *i << '\n';
        i = erase(i);
    }
    else ++i;
}
```
The return value of erase is exactly what we need.

## Item 10: Be aware of allocator conventions and restrictions

Allocators are weird.

All allocator objects of the same type are equivalent and always compare equal.

```c++
template<typename>
class SpecialAllocator {...};

typedef SpecialAllocator<Widget> SAW;

list<Widget, SAW> L1;
list<Widget, SAW> L2;.
L1.splice(L1.begin(), L2);
```

Memory can be deallocated by another allocator of the same type.

So, allocators may not have state.

allocators and operator new:

```c++
void* operator new(size_t bytes);
pointer allocator<T>::allocate(size_type numObjects);
```

Most of the standard containers never make a single call to the allocators with which thery are instantiated.

Reason behind? Container may need to allocate memory of other type.

Consider list:

```c++
template<typename T, typename Allocator = allocator<T>>
class List {
private:
    Allocator alloc;
    struct ListNode {
        T data;
        ListNode* prev;
        ListNode* next;
    };
}
```

So, its easy to see that every time when we allocate memory, `T` is not enough, we need to allocate `ListNode` memory.

rebind to another type (T -> ListNode)

```c++

template<typename T>
class allocator {
public:
    template<typename U>
    struct rebind {
        typedef allocator<U> other;
    };
    ...
};

Allocator::rebind<ListNode>::other   // represent ListNode
```

### Things to Remember

* Make your allocator a template, with the template parameter T representing the type of objects for which you are 
allocating memory.

* Provide the typedefs pointer and reference, but always have pointers be T\* and reference be T&.

* Never give your allocators per-object state. In general, allocators should have no nonstatic data members.

* Remeber that an allocator's allocate member functions are passed the number of objects for which memory required, not
the number of bytes needed. Also remember that these functions return T\* pointers (via the pointer typedef), even though
no T objects have yet been constructed.

* Be sure to provide the nested rebind template on which standard containers depend.

More to read:

1. http://www.josuttis.com/cppcode/allocator.html

    A very allocator example. Use allocate() to allocate memory and construct() to construct object upon it. Use operator new and placement new respectively.

2. http://www.drdobbs.com/the-standard-librarian-what-are-allocato/184403759

## Item 11: Understand the legitimate uses of custom allocators

The default STL memory manager (i.e., allocator<T>) is too slow, wastes memory or suffers excessive fragmentation.
Or you discover that allocator<T> takes precautions to be thread-safe, but you're intereseted only in single-threaded execution.
Or you know that objects in certain containers are typically used together.  
Or you'd like to set up a unique heap that corresponds to shared memory.

You neeed custom allocator.

```c++
// special routines you have to allcoate memory
void* mallocShared(size_t bytesNeeded);
void freeShared(void* ptr);

template<typename T>
class SharedMemoryAllocator {
public:
    ...
    pointer allocate(size_type numObjects, const void* localityHint = 0) {
        return static_cast<pointer>(mallocShared(numObjects*sizeof(T)));
    }
    void deallocate(pointer ptrToMemory, size_type numObjects) {
        freeShared(ptrToMemory);
    }
};

vector<double, SharedMemoryAllocator<double>> v;
```

To put both v's contents and v itself into shared_memory, you have to allocate/construct/destroy/destroy/deallocate...

Second example. Two heaps.

```c++
class Heap1 {
public:
    static void* alloc(size_t numBytes, const void* memoryBlockToBeNear);
    static void dealloc(void* ptr);
};
class Heap2 {...};

template<typename T, typename Heap>
SpecificHeapAllocator {
public:
    pointer allocate(size_type numObjects, const void* localityHint = 0) {
        return static_cast<pointer>(Heap::alloc(numObjects*sizeof(T)), localityHint);
    }
    void deallocate(pointer ptrToMemory, size_type numObjects) {
        Heap::alloc(ptrToMemory);
    }
};

vector<int, SpecificHeapAllocator<int, Heap1>> v;
set<int, SpecificHeapAllocator<int, Heap1>> s;
```

## Item 12: Have realistic expectations about the thread safety of STL containers.

* Mutliple readers are safe.

* Mutliple write to different containers are safe.

STL does't guarantee thread safety. So do it yourself...

Idiom: Resource acquisition is initialization.

No need to release manually.

Robust to exceptions.

```c++
// one way
vector<int> v;

getMutexFor(v);
vector<int>::iterator first5(find(v.begin(), v.end(), 5));
if (first5 != v.end()) {
    *first5 = 0;
}
releaseMutexFor(v);

// another way, RAII, similar to lock_guard
template<typename Container>
class Lock {
public:
    Lock(const Container& container): c(container) {
        getMutexFor(c);
    }
    ~Lock() {
        releaseMutexFor();
    }
private:
const Containter& c;
};

{
    Lock<vector<int>> lock(v);
    vector<int>::iterator first5(find(v.begin(), v.end(), 5));
    if (first5 != v.end()) {
        *first5 = 0;
    }
}  // close block, automatically releasing the mutex

```
