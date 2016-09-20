# Ch3 Associative Containers

## Item 19: Understand the diffference between equality and equivalence

Equality is based on operator== (e.g. find)

Equivalence is based on operator< (e.g. insert)

Equivalence is based on the relative ordering of obejct values in a sorted range (set, map...).

Equivalent if 

    !(w1 < w2) && !(w2 < s1)

user defined predicate:

    !c.key_comp()(x,y) && !c.key_comp()(y,x)


An example of case-insensitive set<string>:

    struct CIStringCompare: public binary_function<string, string, bool> {
        bool operator()(const string& lhs, const string& rhs) const {
            return ciStringCompare(lhs, rhs);  // ...
        }
    };

    set<string, CIStringCompare> ciss;

    ciss.insert("Persephone");
    ciss.insert("persephone");  // no new element added

    if (ciss.find("persephone") != ciss.end())...   // this test will succeed
    if (find(ciss.begin(), ciss.end(), "persephone") != ciss.end())...   // this test will fail

Prefer member functions (like set::find) to their non-member counterparts (like find).

The first one use equivalent, the latter use equality.

It will be confusing if set like STL container has two comparison functions. Then two kinds of equalilty exist.

## Item 20: Specify comparison types for associative containers of pointers

Of course... Otherwise, it only compares the pointer.

Example:
    
    struct StringPtrLess: public binary_function<const String*, const string*, bool> {
        bool operator()(const string* ps1, cosnt string* ps2) const {
            return *ps1 < *ps2;
        }
    };
    set<string*, StringPtrLess> ssp;

    // print
    for (auto i = ssp.begin(), i != ssp.end(); ++i) cout << **i << endl;
    for_each(ssp.begin(), ssp.end(), [](const string* ps){ cout << *ps << endl; });

    struct Dereferece {
        template<typename T> 
        const T& operator()(cosnt T* ptr) const {
            return *ptr;
        }
    };
    trasform(ssp.begin(), ssp.end(), ostrream_iterator<string>(cout, "\n"), Dereferece());

It's comparison type. Functions won't work.

    bool stringPtrLess(const string* ps1, const string* ps2) {
        return *ps1 < *ps2;
    }
    set<string, stringPtrLess> ssp;  // won't compile

## Item 21: Always have comparison functions return false for equal values

    set<int, less_equal<int>> s;  // don't do this

The same element will be inserted again...

    Also don't do this:

    struct StringPtrGreater: public binary_function<const String*, const string*, bool> {
        bool operator()(const string* ps1, cosnt string* ps2) const {
            return !(*ps1 < *ps2);   //  Be careful, you are also breaking the rule...
        }
    };

Even not for multiset and multimap.

## Item 22: Avoid in-place key modification in set and multiset

For map, you cannot change the key. Since it's in type `pair<const K, V >`

For set, don't change the key part.

For some implementation of iterator, the operator\* for set return a const T&. Maybe you should do casting.

    cosnt_cast<Employee&>(*i).setTitle("...");  // ok
    static_cast<Employee>(*i).setTitle("...");  // not ok. Make a copy!

Or you can erase it first and then insert it for modification.

## Item 23: Consider replacing associativec containers with sorted vectors.

1. size needed is smaller.

2. faster due to smaller size.

Emulate a set is easy, emulate a map needs more work.

```c++
typedef pair<string, int> Data;
class DataCompare {
public:
    bool operator()(cosnt Data& lhs, const Data& rhs) const {   // comparison func for sorting
        return keyLess(lhs.first, rhs.first);
    }
    bool operator()(cosnt Data& lhs, const Data::first_type& k) const {  // for lookups
        return keyLess(lhs.first, k);
    }
    bool operator()(cosnt Data::first_type& k, const Data& rhs) const {  // for lookups
        return keyLess(k, rhs.first);
    }
private:
    bool keyLess(const Data::first_type& k1, const Data::first_type& k2) const {
        return k1 < k2;
    }
};

vector<Data> vd;

sort(vd.begin(), vd.end(), DataCompare());
string s;

if (binary_search(vd.begin(), vd.end(), s, DataCompare()))...  // lookup via binary_search

auto i = lower_bound(vd.begin(), vd.end(), s, DataCompare());  // lookup via lower_bound

auto equal_range(vd.begin(), vd.end(), s, DataCompare());  // lookup via equal_range
```

## Item 24: Choose carefully between operator[] and map::insert when efficiency is important

When adding, map::insert is prefered. operator[] may degrade performance. It first default construct an object then immediately assign it a new value.

When updating, operator[] is prefered. Save the cost of creating a temporary `pair<K,V>`.

## Item 25: Familiarize yourself with the nonstandard hashed containers

C++11 has hashed containers alreadly.
