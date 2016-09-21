# Ch7: Programming with the STL

## Item 43: Prefer algorithm calls to hand-written loops

* Efficiceny

STL implementers have more chance to optimize the loop (e.g. optimize traversal in deque).

* Correctness

Sometimes maybe hard to write correct code.

```c++
deque<double>::iterator insertLocation = d.begin();

for (size_t i = 0; i < numDoubles; ++ i) {
    insertLocation = d.insert(insertLocation, data[i]+41)p
    ++insertLocation;
}

// Same logic
transform(data, data+numDoubles, inserter(d, d.begin()), bind2nd(plus<int>(41)));

```

* Maintainability

Easier to read and maintain.

Always good to replace low-level words like for, while...

Sometimes you can also use for-loop if it's hard to express in function.

## Item 44: Prefer member functions to algorithms with the same names

May be much faster.

```c++
set<int> s;
set<int>::iterator i = s.find(727);  // much faster, log time.
set<int>::iterator i = find(s.begin(), s.end(), 727);  // much slower
```

Efficiency is not the only difference. STL algorithms check for equality, while
associative containers use equivalence as their "sameness" test.

This difference is especially pronounced when working with maps and multimaps.
The containers hold pair objects, yet their member functions look only at the key part of each pair.

Some functions on list behave differently from their algorithm counterparts.

## Item 45: Distinguish among count, find, binary_search, lower_bound, upper_bound, and equal_range

```c++
vector<Widget> w;
sort(vw.begin(), vw.end());
auto i = lower_bound(vw.begin(), vw.end(), w);
if (i != vw.end() && *i == w) {  // error, use equlity test!
}
```

Should use equivalence...

```c++
VWIterPair p = equal_range(vw.begin(), ve.end(), w);
if (p.first != p.second) {
}
distance(p.first, p.second);  // return the number of equal range
```

## Item 46: Consider function objects instead of functions as algorithm parameters

### faster

Function object's operator() function will have more chance to be inlined.

```c++
vector<double> v;
sort(v.begin(), v.end(), greater<double>());

inline bool doubleGreater(double d1, double d2) {
    return d1 > d2;
}
sort(v.begin(), v.end(), doubleGreater);  // doubleGreater is a function pointer bool(*)(double, double)
```

What about lambda in C++11? I think it should be well optimized since it's annonymous class object basically.

C++ sort is way more faster than C's qsort.
C++'s "overhead" is absorbed during compilation.

sort makes inline calls to its comparison function while qsort calls through pointer.

More room for the compiler to optimize.

### Have to use function objects

1. May have bugs in handling of const member functions.

2. May have some errors working with template

## Item 47: Avoid produing write-only code

Get rid of all the elements in the vector whose value is less thanx, except that elements preceding the last 
occurrence of a value at least as big as y should be retained.

```c++
vector<int> v;
int x,y;
v.erase(
    remove_if(find_if(v.rbegin(), v.rend(),
                      bind2nd(greater_equal<int>(), y)).base(),
              v.end(),
              binid2nd(less<int>(), x)),
    v.end());
```

May consider separate them.

Philosophy:

Code is read more than it is written. 

Software spends far more time in maintenance than it does in development.

Software that can not be read and understood cannot be maintained, and software that cannot be maintained is hardly
worth having.

## Item 48: Always #include the proper headers

* Almost all the containers are declared in headers of the same name.

* All but four algorithms declared in <algorithm>, except accumulate, inner_product, adjacent_difference, and partial_sum are declared in <numeric>.

* special kinds of iterators, including istream_iterators and istreambuf_iterators are declared in <iterator>.

* standard functors (e.g., less<T>) and functor adapter (e.g., not, bind2nd) are declared in <functional>.

## Item 49: Learn to decipher STL-related compiler diagnostics

May replace `basic_string<char, char_traits<char>, allocator<char>>` with `string`.

Replace by typedef.

## Item 50: Familiarize yourself with STL-related websites

The SGI STL site

The STLport site

The Boost site
