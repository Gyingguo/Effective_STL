# Ch2: vector and string

## Item 13: Prefer vector and string to dynamically allocated arrays

Of course, we should prefer vector and string.

No need to handle memory allocation/deallocation.

Use dynamic array, you need to delete, delete in the right way and delete exactly once.

Reference count in string may degrade the performance in multithreaded environment.

## Item 14: Use reserve to avoid unnecessary reallocations

Use reserve to minimize the number of reallocations that must be performed.

## Item 15: Be aware of variations in string implementations

Introduce four kinds of string implementation.

Information may be held: size, capacity, value, allocator, reference count.

Small String optimization.

* string values may or may not be reference counted.

* string objects may range in size from one to at least seven times the size of char\* pointer.

* Creation of a new string value may require zeor, one, or two dynamically allocations.

* string objects may or may not share information on the string's size and capacity.

* string may or may not support per-object allocators.

* Different implementations have different policies regarding minimum allocations for character buffers.

## Item 16: Know how to pass vector and string data to legacy APIs

```c++
&v[0]

s.c_str()

void doSomething(const int* pints, size_t numInts);
void doSomething(const char* pints);
```

&v[0] is better than v.begin(). v.begin() is an iterator. Or you can do this: &\*v.begin().

You'd better not to modify the content...

Or don't change the size...

```c++
size_t fillArray(double* pArray, size_t arraySize);
vector<double> vd(maxNumDoubles);

vd.resize(fillArray(&vd, vd.size()));  // do a resize...
```

Other stl types? Pass them to vector first.

```c++
list<double> l(vd.begin(), vd.end())
deque<double> l(vd.begin(), vd.end())

set<int> inSet;
vector<int> v(inSet.begin(), inSet.end());
doSomething(&v[0], v.size());
```

## Item 17: Use the swap trick to trim excess capacity

shrink to fit...

```c++
vector<Contestant>(contestants).swap(contestants);
string(s).swap(s);
```

This method doesn't take the advantage of std::move. All the objects are copied to the new vector.

For the shrink to fit with std::move, refer to 
http://stackoverflow.com/questions/23502291/is-shrink-to-fit-the-proper-way-of-reducing-the-capacity-a-stdvector-to-its

Can also clear and minimize its capacity:

    vector<Contestant>().swap(v);
    string().swap(s);

## Item 18: Avoid using `vector<bool>`

Cannot get reference to `vector<bool>::operator=()`.

Proxy object
