# Ch5: Algorithms

## Item 30: Make sure destination ranges are big enough

```c++
int transmogrify(int x);
vector<int> values;
...
vector<int> results;
transform(values.begin(), values.end(), results.end(), transmogrify);  // error!
```

transform writes its results by making assignments to the elements in the destination range. So the call is wrong!
No object there.

We need back_inserter:

```c++
vector<int> results;
transform(values.begin(), values.end(), back_inserter(results), transmogrify);  // ok

// the order will be reversed
list<int> results;
transform(values.begin(), values.end(), front_inserter(results), transmogrify);  // ok

// keep the order
list<int> results;
transform(values.rbegin(), values.rend(), front_inserter(results), transmogrify);  // ok

// insert into the middle
vector<int> results;
transform(values.begin(), values.end(), inserter(results, results.begin(), results.size()/2), transmogrify);  // ok
```

Even though you use `reserve`, inserter is still needed. Since reserve only allocate the memory
```c++
vector<int> results;
results.reserve(results.size()+values.size());
transform(values.begin(), values.end(), back_inserter(results), transmogrify);  // ok
```

With resize, you can overwrite the data
```c++
vector<int> results;
if (results.size() < value.size()) {
    results.resize(values.size());
}
transform(values.begin(), values.end(), results.size(), transmogrify);  // overwrite from the first element.
```

## Item 31: Know your sorting options

Partially sort the vector!

```c++
partial_sort(widgets.begin(), widgets.begin()+20, widgets.end(), qualityCompare);  // put the best 20 elements(in order) at the front
```

Get the top nth without sorting
```c++
nth_element(widgets.begin(), widgets.begin()+20, widgets.end(), qualityCompare);  // put the best 20 elements(don't worry about the order) at the front
```

stable_sort is stable but STL doesn't conatin stable versions of partial_sort and nth_element.

nth_element can also be used to find the median value in a range or to find the value at a particular percentile.

```c++
nth_element(begin, goalPosition, end, qualityCompare);
```

partition!

```c++
// move all widgets satisfying hasAcceptableQuality to the front of widgets and return an iterator to the first widget that isn't satisfactory
vector<Widget>::iterator goodEnd = partition(Widgets.begin(), Widgets.end(), hasAcceptableQuality);
```

Speed:
1. partition
2. stable_partition
3. nth_element
4. partial_sort
5. sort
6. stable_sort

## Item 32: follow remove-like algorithms by erase if you really want to remove something

```c++
template<class ForwardIterator, class T>
ForwardIterator remove(ForwardIterator first, ForwardIterator last, const T& value);
```

The only wyt to eleiminates element is to call a member function (erase). remove cannot know the container.
So it's inpossible for remove to eliminate elements from a container.

** remove doesn't "really" remove anything, because it can't. **

remove just move all the elements that's not satisfied forward.

1 2 3 99 5 99 7 8 9 99

```c++
auto newEnd(remove(v.begin(), v.end(), 99));
```

1 2 3 5 7 8 9 8 9 99

Not actually remove anything.

erase is needed.

```c++
v.erase(remove(v.begin(), v.end(), 99), v.end());
```

Only exception is that remove member function in list actually eliminate something.

```c++
list<int> li;
li.remove(99);
```

## Item 33: Be wary of remove-like algorithms on containers of pointers

```c++
vector<Widget*> v;
v.erase(remove_if(v.begin(), v.end(), not1(mem_fun(&Widget::isCertified))), v.end());
```

Resource leak!!! Some objects don't get free.

Solutions

* partition algorithm may work.

* Delete all the pointers and set them to null before.

```c++
void delAndNullifyUncertified(Widget*& pWidget) {
    if (!pWidget->isCertified()) {
        delete pWidget;
        pWidget = 0;
    }
}
for_each(v.begin(), v.end(), delAndNullifyUncertified);
v.erase(remove(v.begin(), v.end(), static_cast<Widget*>(0)), v.end());
```

* Use smart pointer.

## Item 34: Note which algorithms expect sorted ranges

Require sorted:

binary_search

upper_bound

lower_bound

equal_range

set_union

set_difference

set_intersection

set_symmetric_difference

merge

inplace_merge

includes

Some are typically used with sorted range:

unique

unique_copy

unique: Elemintates all but the first element from every consecutive group of equal elements

```c++
sort(v.begin(), v.end(), greater<int>());
bool a5Exist = binary_search(v.begin(), v.end(), 5, greater<int>);  // using greater as the comparison function
```

## Item 35: Implement simple case-insensitive string comparisons via mismatch or lexicographical_compare

1. Can use mismatch function

2. Can use lexicographical_compare function

## Item 36: Understand the proper implementation of copy_if

No copy_if in STL.

Implement yours.

```c++
template<typename InputIterator, typename OutputIterator, typename Predicate>
OutputIterator copy_if(InputIterator begin, InputIterator end, OutputIterator destBegin, predicate p) {
    return remove_copy_if(begin, end, destBegin, not1(p));  // use remove_copy_if, not queite right
    // not1 cannot be applied directly to a function pointer.
}

// Right way
template<typename InputIterator, typename OutputIterator, typename Predicate>
OutputIterator copy_if(InputIterator begin, InputIterator end, OutputIterator destBegin, predicate p) {
    while(begin != end) {
        if (p(*begin)) *destBegin++ = *begin;
        ++begin;
    }
    return destBegin;
}
```

## Item 37: Use accumulate or for_each to summarize ranges

```c++
list<double> ld;
double sum = accumulate(ld.begin(), ld.end(), 0.0); // Be careful, shouldn't be 0
```

With a function
```c++
string::size_type stringLengthSum(string::size_type sumSoFar, const string& s) {
    return sumSoFar + s.size();
}
string::size_type lengthSum = accumulate(ss.begin(), ss.end(), 0, stringLengthSum);

float product = accumulate(vf.begin(), vf.end(), 1.0, multiplies<float>());
```

accumulate seems not to allow side-effect!

So the following may not work...

```c++
Point avg = accumulate(lp.begin(), lp.end(), Point(0,0), PointAverage());

class PointAverage: public binary_function<Point, Point, Point> {
public:
    PointAverage():xSum(0), ySum(0), numPoints(0) {}
    const Point operator(const Point& avgSoFar, const Point& p) {
        ++numPoints;
        xSum += p.x;
        ySum += p.y;
        return Point(xSum/numPoints, ySum/numPoints);
    }
private:
    size_t numPoints;
    double xSum;
    double ySum;
};
```

for_each works!

```c++
class PointAverage: public unary_function<Point, void> {
public:
    PointAverage(): xSum(0), ySum(0), numPoints(0) {}
    void operator()(const Point& p) {
        ++ numPoints;
        xSum += p.x;
        ySum += p.y;
    }
    Point result() const {
        return Point(xSum/numPoints, ySum/numPoints);
    }
private:
    size_t numPoints;
    double xSum;
    double ySum;
};

Point avg = for_each(lp.begin(), lp.end(), PointAverage()).result();
```
