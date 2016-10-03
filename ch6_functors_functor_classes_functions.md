# ch6: Functors, Functor Classes, Functions, etc.

## Item 38: Design functor classes for pass-by-value

Functor classes should be passed by value. This implies two things.

First, your function objects need to be small. 

Second, your function objects must be monomorphic (i.e., not polymorphic). Slicing problem.

Solutions:

Take the data and/or the polymorphism you'd like to put in your functor class, and move it into a differnt class.
Then ive your functor class a pointer to this new class. 

```c++
template<typename T>
class BPFC : public unary_function<T, void> {   // BPFC = Big Plymorphic Functor Class
private:
  Widget w;  // this class has lots of data, so it would be inefficient to pass it by value
  int x;
  ...
public:
  virtual void operator()(const T& val) const;  // this is a virtual function, so slicing would be bad
};
```

Create a small monomorphic class that contains a pointer to an implementation class, and put all the 
data and virtual functions in the implementation class:
```c++
template<typename T>    // new implementation class for modified BPFC
class BPFCImple {
private:
  Widget w;   // all the data that used to be in BPFC are now here
  int x;
  ...
  virtual void operator()(const T& val) const;  // polymorphic classes need virtual destructor
friend class BPFC<T>;  // let BPFC access the data
};

template<typename T>  // small, monomorphic version of BPFC
class BPFC : public unary_function<T, void> {
private:
  BPFCImple<T>* pImpl;   // this is BPFC's only data
public:
  operator()(const T& val) const {  // this is now nonvirtual
    pImpl->operator()(val);
  }
};
```

May be called Bridge Patern.

## Item 39: Make predicates pure functions

A *predicate* is a function that returns bool. 

A *pure function* is a function whose return value depends only on its parameters. 

A *predicate class* is a functor class whose operator() function is a predicate.

Predicate functions must be pure functions.

```c++
class BadPredicate : public unary_function<Widget, bool> {
public:
  BadPredicate() : timesCalled(0) {}
  bool operator()(const Widget&) {
    return ++timesCalled = 3;
  }
private:
  size_t timesCalled;
};

vw.erase(remove_if(vw.begin(), vw.end(), BadRedicate()), vw.end());
```
May erase the third and the sixth elements! Due to "functor classes are passed by value".

```c++
template<typename FwdIterator, typename Predicate>
FwdIterator remove_if(FwdIterator begin, FwdIterator end, Predicate p) {
  begin = find_if(begin, end, p);  // p is passed by value
  if (begin == end) return begin;
  else {
    FwdIterator next = begin;
    return remove_copy_if(++next, end, begin, p);  // p is also passed by value, so timesCalled is still 0 here
  }
}
```

Make it const!
```c++
class BadPredicate : public unary_function<Widget, bool> {
public:
  bool operator()(const Widget&) const  {
    return ++timesCalled = 3;  // error! can't change local data in a const member function
  }
  ...
};

bool anotherBadPredicate(const Widget&, const Widget&) {
  static int timesCalled = 0;  // No!!! Bad idea.
  return ++timesCalled == 3;
}
```

Regardless of how you write your predicates, they should always be pure functions.

## Item 40: Make functor classes adaptable

```c++
list<Widget*> widgetPtrs;
bool isInteresting(const Widget* pw);
list<Widget*>::iterator i = find_if(widgetPtrs.begin(), widgetPtrs.end(), isInteresting);  // ok
list<Widget*>::iterator i = find_if(widgetPtrs.begin(), widgetPtrs.end(), not1(isInteresting));   // error! won't compile
list<Widget*>::iterator i = find_if(widgetPtrs.begin(), widgetPtrs.end(), not1(ptr_fun(isInteresting)));  // fine
```

The only thing ptr_fun does is make some typedefs available. These typedefs are required by not1.

Each of the four standard function adapters (not1, not2, bind1st, adnd bind2nd) requries the existence of certain typedefs. 

Function objects that provide the necessary typedefs are said to be adaptable. 

The typedefs are argument_type, first_argument_type, second_argument type, and result_type.

No need to know anythong about these typedefs. Inherits a base class. `std::unary_function` and `std::binary_function`

Examples:

```c++
template<typename T>
class MeetsThreshold : public std::unary_function<Widget, bool> {
private:
  const T threshold;
public:
  MeetsThreshold(const T& threshold);
  bool operator()(const Widget&) const;
};
```

Event though operator()'s arguments are of type const Widget&, the type passed to binary_function is Widget. 

The rules change when operator() takes pointer parameters.
```c++
struct PtrWidgetNameCompare: std::binary_functino<const Widget*, const Widget*, bool> {
  bool operator()(const Widget* lhs, const Widget* rhs) const;
}
```

Not good to combine the functionality of WidgetNameCompare and PtrWidgeNameCompare by creating a single struct 
with two operator() functions.

## Item 41: Understand the reasons for ptr_fun, mem_fun and mem_fun_ref

```c++
f(x);  // Syntax #1 When f is a non-member function
x.f(); // Syntax #2 When f is a member function and x is an object or a reference to an object
p->f();  // Syntax #3 When f is a member function and p is a pointer to x.

void test(Widget& w);
class Widget {
public:
  void test();
};
for_each(vw.begin(), vw.end(), test);  // compiles
for_each(vw.begin(), vw.end(), &Widget::test);  // won't compile
list<Widget*> lpw
for_each(lpw.begin(), lpw.end(), &Widget::test);  // won't compile
```

In the world we do have, we posses only one version of for_each.
```c++
template<typename InputIterator, typename Function>
Function for_each(InputIterator begin, InputIterator end, Function f) {
  while (begin != end) f(*begin++);
}
```

```c++
// declaration for mem_fun for non-const member functions taking no parameters. 
// C is the class, R is the return type of the pointed-to member function
template<typename R, typename C>
mem_fun_t<R,C> mem_fun(R(C::*pmf)());
```
mem_fun taks a pointer to a member function, pmf, and returns an object of type mem_fun_t. This is a functor class
that holds the member function pointer and offers an operator() that invokes the pointed-to member function.

```c++
list<Widget*> lpw
for_each(lpw.begin(), lpw.end(), mem_fun(&Widget::test));
```

mem_fun_ref function works for Syntax #2

Adding the typedefs can't hurt anything.

```c++
for_each(vw.begin(), vw.end(), test);
for_each(vw.begin(), vw.end(), ptr_func(test));
```

## Item 42: Make sure less<T> means operator< 

Specialize std::less

```c++
template<>
struct std::less<Widget> : public std::binary_function<Widget, Widget, bool> {
  bool operator()(const Widget& lhs, const Widget& rhs) const {
    return lhs.maxSpeed() < rhs.maxSpeed();
  }
};
```

Not a good idea to overload less<T>, since it's the default way of comparison for some containers. 

operator< is more than just the default way to implement less, it's what programmers expect less to do.

```c++
struct MaxSpeedCompare : public binary_function<Widget, Widget, bool> {
  bool operator()(const Widget& lhs, const Widget& rhs) const {
    return lhs.maxSpeed < rhs.maxSpeed;
  }
};
multiset<Widget, MaxSpeedCompare> widgets;
multiset<Widget> widgets;  // it uses less<Widget>, but virtually everybody assumes that really means operator<
```
