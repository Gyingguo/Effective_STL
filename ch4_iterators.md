# Ch4 Iterators 

iterator, const_iterator, reverse_iterator, const_reverse_iterator

All the stl implementation: http://www.sgi.com/tech/stl/download.html

## Item 26: Prefer iterator to const_interator, reverse_iterator, and const_reverse_iterator.

insert, erase only accept exactly iterator type.

    iterator insert(iterator posistion, const T& x);
    iterator erase(iterator position);
    iterator erase(iterator rangeBegin, iterator rangeEnd);

Conversions:

iterator -> const_iterator

iterator -> reverse_iterator

reverse_iterator -> const_reverse_iterator

reverse_iterator -> iterator (base())

const_reverse_iterator -> const_iterator (base())

Why Prefer iterators?

* Some versions of insert and erase require iterators. If you want to call those functions, 
you're going to have to produce iterators. const and reverse iterator won't do.

* It's not possible to implicitly convert a const iterator to an iterator and the technique in Item 27 for
generating an iterator from const_iterator is neither universally applicable nor guaranteed to be efficient.

* Conversion from reverse_iterator to an iterator may require iterator adjustment after the conversion. Item 28.

Compare an iterator and a const_iterator

    if (i == ci)
        ...

However, some implementations declare operator== for const_iterators as a member functoin instead of
a non-member functions.

    if (ci == i)  // workaround...

    if (i - ci >= 3)
    // workaround
    if (ci + 3 <= i)

## Item 27: Use distance and advance to convert a container const_iteratros to iterators

How to convert const_iterator to iterator?

1. casting

    Iter i(ci);  // error

    Iter i(const_cast<Iter>(ci));  // Still error

    Cannot compile. Except for vector and string.

    In vector or string, the iterator is typedef for `T*`.

2. Use advance

    Iter i(d.begin());
    advance(i, distance(i, ci));  // Must be tweaked before it will compile

Look at the declaration of advance:
    
    template<typename InputIterator>
    typename iterator_traits<InputIterator>::difference_type
    distance(InputIterator first, InputIterator last);

InputIterator type is confusing, iterator or const_iterator?

So:
    
    advance(i, distance<ConstIter>(i, ci));

The complexities:

1. Constant-time for random access iterators (vector, string, deque).

2. Linear-time for others.

## Item 28: Understand how to use a reverse_iterator's base iterator.

### Insert

To emulate insertion at a position specified by a reverse_iterator ri, insert at the position
ri.base() instead.

    vector<int>::reverese_iterator it = find(v.rbegin(), r.rend(), 3);
    vector<int>::iterator i(ri.base());

### Erase

To emulate erasure at a position specified by a reverse_iterator it, erase at the postition
preceding ri.base() instead. For purposes of erasure, ri and ri.bases() are not equivalent,
and ri.base() is not the iterator corresponding to ri.

    vector<int>::reverese_iterator it = find(v.rbegin(), r.rend(), 3);
    v.erase(--ri.base());  // will typically not compile

This code will workwith every standard containter except vector and string.

Some are implemented using built-in pointers. So the ri.base() is a pointer.

Both C and C++ dicatate that pointers returned from functions shall not be modified.

Portable way:

   v.erase((++ri).base());

For insertion purposes, iterator's base member function returns the *corresponding* iterator

For erasure purposes, it does not.

## Item 29: Consider istreambuf_iterators for character-by-character input.

istream_iterators use operator<< functions to do the actual reading. By default, it skips whitespace.

    ifstream inputFile("filename.txt");
    inputFile.unsetf(ios::skipws);
    string fileData((istream_iterator<char>(inputFile)), istream_iterator<char>());

Slow... create and destroy sentry objects, check stream flags, perform comprehensive checking for read errors...

    string fileData((istreambuf_iterator<char>(inputFile)), istreambuf_iterator<char>());

Much faster. Go straight to the stream's buffer and read the next character directly.
