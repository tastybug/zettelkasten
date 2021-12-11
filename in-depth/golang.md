# Golang

## Type Properties (Comparisons, Behaviour)

### Slices
* you cannot compare 2 slices directly: a == b (compiler will tell you), you have to manually compare the contents (but for byte slices, there is a helper: bytes.Equal(a,b))
* a slice can be nil, meaning it has no underlying array (e.g. "var x []string" is nil). A slice is not nil, if it has an underlying empty array, e.g. from "make ([]int, 3)[3:]"

### Arrays
* you can compare arrays directly

### Functions
* cannot be compared
* not usable as keys in maps (due to imcomparability)
* receiver functions can be called for nil instances of that type. For some types this is ok, for others this causes panics

### Maps
* Cannot be directly compared. Iterate over the entries and compare those.
* You can only use a type as key that is comparable (structs with comparable fields are ok!). Slices thus cannot be used. A trick around this can be to turn the slice into a string (fmt.Sprintf("%q", mySlice)) and use that as the key.

### Interface
* can be compared if the dynamic type is comparable (if not, you get a panic); they are equal if both "actual" dynamic types are equal and values are equal depending on the rules of that type
* when calling an interface function, this acts like syntactic sugar: the type's method will be called
* nil instances can be valid implementors of an interface; check if the concrete type survives calls on nil instances!

### Structs:
* Can be compared directly; technically it compares the fields in order of type declaration
* one can create a struct pointer quickly with new, e.g. pp := new(Point);  *pp = Point{1, 2}
* a struct can be without any fields; this still allows for receiver methods
* structs can be composed with "anonymous fields":

```
type Point struct {
    X, Y int
}

type Circle struct {
    Point
    Radius int
}

type Wheel struct {
    Circle
    Spokes int
}
```

Allowing for shorter access to fields: w.Spokes, w.Radius, w.X, w.Y can be accessed directly. Receiver methods of the inner types are available on the outer type. But the outer type is not the inner type (polymorphic). If you have a function expecting the inner type as an argument, you can't provide the outer type. Instead, provide for example wheel.Circle.

### Sets:
* there are no sets in golang, but you can use map[TYPE]bool, with the keys being a set of TYPE

## Call By Value

* call by value happens for number, string, array, structs (mind the field types!)
* map: is technically a pointer. It being copied is fine, treat it like a reference type.
* slice: is meta data (len, cap) and a pointer. It being copied is fine. Treat it like a reference type.

### Type Assertion (i.e. instanceof and type casting from interfaces)

If you have a value of an interface type (e.g. var x io.Writer), you can test for the concrete type or applicable other interfaces. This only works for interface types.

Example:

```
func (x interface{}) {
    if s, ok := x.(io.Stringer); ok {
    	fmt.Println(s) // s is a io.Stringer
    } else if i, ok := x.(int); ok {
    	fmt.Println(i) // i is an int
    }
    // and so on
}
```
Example 2, even easier:
```
var x interface{}
x = 1
switch x.(type) {
	case nil: return ``
	case uint, int: return `` // this is where we'll land
	case string: return ``
	case io.Writer: return ``
	// and so on
}
```
