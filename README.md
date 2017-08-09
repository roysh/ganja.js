# ganja.js - Geometric Algebra for javascript.

<CENTER><IMG SRC="ganja_thumb.jpg"></CENTER>

Ganja.js is a Geometric Algebra code generator for javascript. It supports
algebra's of any signature and implements advanced operator overloading and
algebraic constants.

* Supports any metric (p,q,r) (spacelike/timelike/lightlike)
* Operator overloading
* Algebraic constants
* smallish (138 lines)
* inverses up to 5D.
* geometric, inner, outer product
* conjugate, reverse, involute, dual, negative
* 4 API's (inline, asciimath, object oriented, functional)

To use it, first include the ganja.js script.
```html
<SCRIPT SRC="https://raw.githubusercontent.com/enkimute/ganja.js/master/ganja.js"></SCRIPT>
```
### Create your algebra class.

To create an Algebra, all you need to know is its metric. The Algebra
function will generate an ES6 class that implements the algebra with the
specified metric (p,q,r). (p = # of dimensions that square to 1, q = # that
square to -1, r = # that square to 0).

```javascript
var Complex = Algebra(0,1);     // Complex numbers.
var E2 = Algebra(2);            // Euclidean 2D space.
var E3 = Algebra(3);            // Euclidean 3D space.
var timeSpace = Algebra(3,1);   // timespace
var P3 = Algebra(3,0,1);        // Projective 3D space
var C3 = Algebra(4,1);          // Conformal 3D space
```

The result of this call will be an ES6 class object (deriving from
Float64Array). These classes can be used to generate algebra elements with
all expected operators available. Apart from the (default) OO syntax, they
also provide a functional syntax (through static member functions), and two
inline syntaxes (javascript and AsciiMath).

The Inline and AsciiMath syntaxes provide in full operator overloading and 
literal algebraic objects - targetting the more mathematically inclined. The 
Object Oriented and functional syntax on the other hand will probably feel 
more familiar to engineers and programmers.  

After creating your algebra you can view its properties (metric, Cayley
table, basis names) in the console with the describe function. 

```javascript
Complex.describe();
```
To use the inline syntax call the _inline_ function passing in your function
object, for the AsciiMath syntax, you pass in a string. In both cases the
_inline_ function will return a new function object. (where e-notation and
operators are resolved) 

```javascript
// Inline javascript syntax : 
Complex.inline(function(){
  return 2 + 3e1; // 2+3i
});

// Inline AsciiMath syntax : 
Complex.inline("2+3e_1")
```
### ganja.js Syntax Overview.

Here's a list of the supported operators in all syntax flavors : 

|Inline JS | AsciiMath | Object Oriented | Functional
|----------|-----------|-----------------|------------
| ~x       |  hat(x)   | x.Conjugate     | A.Conjugate(x)
| x.Involute|  tilde(x) | x.Involute      | A.Involute(x)
| x.Reverse|  ddot(x)  | x.Reverse       | A.Reverse(x)
| !x       |  hat(x)   | x.Dual          | A.Dual(x)
| x**-1    |  x^-1     | x.Inverse       | A.Inverse(x)
| x**y     |  x^y      | x.Pow(y)        | A.Pow(x,y)
| x*y      |  x**y     | x.Mul(y)        | A.Mul(x,y)
| x/y      |  x/y      | x.Div(y)        | A.Div(x,y)
| x^y      |  x^^y     | x.Wedge(y)      | A.Wedge(x,y)
| x<<y     |  x*y      | x.Dot(y)        | A.Dot(x,y)
| x-y      |  x-y      | x.Sub(y)        | A.Sub(x,y)
| x+y      |  x+y      | x.Add(y)        | A.Add(x,y)
| 1e1      |  1e_1     | new A([0,1])    | A.Vector(1)
| 2e2      |  2e_2     | new A([0,0,2,0])| A.Vector(0,2)
| 2e12     |  2e_12    | new A([0,0,0,2])| A.Bivector(2)

### Javascript inline syntax.

Ganja.js' inline syntax allows you to write GA statements seamlessly inside your
javascript functions. Simply wrap your functions to get full operator
overloading and algebraic constants.

To enable you to directly write algebraic constants, we overload the
scientific notation. Allowing you to write basis blade names 
**e<sub>1</sub>**, **e<sub>2</sub>**, **e<sub>12</sub>**,
**e<sub>235</sub>**, etc as : 

```javascript
// Direct algebraic objects specifying basisblades with e notation.
  var xy_bivector = 1e12;    // 1e12 = 1 time the basis xy-bivector
  var z_vector = 1e3;        // 1e3 = the basis z-vector

// Algebraic objects and operator overloading (^ = wedge) 
  var xy_bivector = 1e1^1e2; // wedge x-vector and y-vector to get the xy-bivector
```

Here are some more examples showing you the power and readability of this
API in C and R3. 

```javascript
Complex.inline(function(){
  console.log( (3+2e1)*(1+4e1) );      // complex multiplication. (outputs -5+14i)
  console.log( (1+4e1)**-1*(1+4e1) );  // complex inverse (outputs 1)
})();

E3.inline(function(){
  console.log( 1e1 );                  // x-vector (outputs [0, 1, 0, 0, 0, 0, 0, 0])
  console.log( 1e1^1e2 );              // x wedge y (outputs [0, 0, 0, 0, 1, 0, 0, 0])
  console.log( 1e12<<1e2 );            // xy dot y (outputs [0, 1, 0, 0, 0, 0, 0, 0])
  console.log( 1e1*1e2 );              // x * y (outputs [0, 0, 0, 0, 1, 0, 0, 0])
  console.log( 1e1.Dual );             // x.Dual (outputs [0, 0, 0, 0, 0, 0, -1, 0])
})();
```

### Example : Mandlebrot

In this example we use the smallest Geometric Algebra that cannonicaly
embeds the Complex numbers. We graph a two dimensional function where
we evaluate the number of iterations the complex expression (z*z+c) needs to
converge to infinity (well .. to more than two ;) ).

This famous equation is known as the Mandelbrot set. In the following piece
of javascript, z and c are complex numbers. The imaginary unit _i_ is
written **_e<sub>1</sub>_** as the first basis vector of R(0,1) squares to -1.  

```javascript
var Complex = Algebra(0,1);

var canvas = Complex.graph(function(x,y){
  var n=110, z=0e1, c=x*1.75-1+y*1e1;
  while (z < 2 && n--) z=z**2+c;
  return (n/100);
});

document.body.appendChild(canvas);
```

<CENTER><IMG SRC="ganja_mandelbrot.png"></CENTER>

### Example Hue Rotor

This example uses three dimensional euclidean space which is the smallest GA
that cannonicaly embeds the quaternions. Because of our order choice of
basis blades (i.e. **e<sub>13</sub>** instead of **e<sub>31</sub>**), the
**e<sub>13</sub>** basis bivector is negated
compared to the default quaternion implementations.

In the code below, **c** is a vector and **rot** is a quaternion (scalar +
bivector). We graph the color vector obtained by rotating red (1e1) in the
(1,1,1) bivector plane. (this is changing the hue).

```javascript
var E3 = Algebra(3);

var canvas = E3.graph(function(x,y){
  var c=1e1, rot=Math.cos(x*2) + Math.sin(x*2)*(1e12+1e23-1e13);
  return (rot*c*rot**-1).Vector;
});

document.body.appendChild(canvas);
```
<CENTER><IMG SRC="ganja_hue.png"></CENTER>


### Example Projective 2D

This example uses projective 2D. (degenerate signature R*2,0,1). (Dual ..
i.e. points are higher grade than lines). (Ganja.js can handle degenerate
metrics). Points are constructed by adding in E0, and the join is the dual of the
wedge of the duals. (meet is just wedge). We construct two lines from points
and find their intersection point, angles and distances.

We've also used a custom ordering of the basis blades, and used
**e<sub>20</sub>** instead of the default **e<sub>e02</sub>**. This matches
the convention of Charles Gunn (refer to his thesis for more information
on projective geometric algebra). 


```javascript
Algebra({metric:[0,1,1],basis:['1','e0','e1','e2','e12','e20','e01','e012']}).inline(function(){ 
  // basis points in Dual Projective 2D. (e3*e3=0)
  var E0 = 1e12, E1 = 1e20, E2 = 1e01;

  // some projective elements/operations (p,p1,p2=point;l,l1,l2=line;X,Y=eucledian coordinates;x,y=multivectors)
  var point           = (X,Y)=>E0+X*E1+Y*E2,
      to_point        = (p)=>p.e12?[p.e01/p.e12,p.e20/p.e12]:[p.e01*Infinity||0,p.e20*Infinity||0],
      join            = (x,y)=>!(!x^!y),                               // union
      meet            = (x,y)=>x^y,                                    // intersect 
      dist_points     = (p1,p2)=>join(p1/p1.e12,p2/p2.e12).Length,     // distance between points
      dist_point_line = (p,l)=>((p.Normalized)^(l.Normalized)).e012,   // oriented distance point to line.
      angle_lines     = (l1,l2)=>(l1.Normalized)<<(l2.Normalized).s;   // angle between lines

  // define 4 points
  var a=point(0,0), b=point(0,1), c=point(1,0), d=point(1,1);

  // join them in 6 lines.
  var ab=join(a,b), ac=join(a,c), ad=join(a,d), bc=join(b,c), bd=join(b,d), cd=join(c,d);

  // output points
  console.log('points : ',[a,b,c,d].map(x=>to_point(x)).join(' and '));   

  // output distances.
  console.log('a to d',dist_points(a,d));
  console.log('ad to c',dist_point_line(ad,c));
  console.log('ab to c',dist_point_line(ab,c));
  console.log('ad meet bc to c',dist_points(meet(ad,bc),c));

  // output intersections
  console.log('ad intersect bc', to_point(meet(ad,bc)));
  console.log('ab intersect cd', to_point(meet(ab,cd)));
  console.log('ac intersect bd', to_point(meet(ac,bd)));

  // output angles.
  console.log('angle ab ad',angle_lines(ab,ad));
  console.log('angle ab cd',angle_lines(ab,cd));
  console.log('angle ad bc',angle_lines(ad,bc));
})();
```

This example outputs :

```
points :  0,0 and 1,0 and 0,1 and 1,1

a to d 1.4142135623730951
ad to c -0.7071067811865475
ab to c -1
ad meet bc to c 0.7071067811865476

ad intersect bc (2) [0.5, 0.5]
ab intersect cd (2) [Infinity, 0]
ac intersect bd (2) [0, -Infinity]

angle ab ad 0.7071067811865475
angle ab cd 1
angle ad bc 0
```
