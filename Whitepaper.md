# An XML exchange format for variable programming tasks

**Version 1.0**

Robert Garmann

## 1 Introduction

This document specifies syntax and semantics for a standardized "Variable ProFormA-XML Format" - an exchange format for variable programming exercises/tasks. It formalises the specification of variable programming exercises. The format contains two main parts: variability and materialization which are each described below.

### 1.1 Example

Let's consider an example of a variable programming task. First, we show two task variants in the following table. The variants differ in their relative difficulty. While it is not obvious, what a difficulty of 1 means, it seems clear, that a task variant of difficulty 1.4 is 75% harder than a task variant of difficulty 0.8. The variants also differ in their automatically executed *tests*. A submission is graded by a compilation test and a JUnit test run. Variant B additionally runs the style checker test PMD. The test results are weighted and accumulated into an overall grading result:

<table>
<tr><th></th><th>description</th><th>difficulty</th><th>grading result</th></tr>
<tr><td>variant A</td><td>
<p>Write a static method <code>greet</code> as part of the Java class <code>Program</code> that returns the message <code>&quot;Hello world.&quot;</code> as a string.</p>
<p>You should base your work on the prepared code template <code>Program.java</code>.</p>
</td><td>0.8</td>
<td>compile (25%), junit (75%)</td></tr>
<tr><td>variant B</td><td>
<p>Write a static method <code>greet</code> as part of the Java class <code>Welcome</code> that returns the message <code>&quot;Nice to meet you, &lt;name&gt;.&quot;</code> as a string, where the value of <code>&lt;name&gt;</code> is passed to <code>greet</code> as a method parameter.</p>
<p>Include javadoc comments with your method.</p>
<p>You should base your work on the prepared code template <code>Welcome.java</code>.</p>
<td>1.4</td>
<td>compile (20%), junit (60%), pmd (20%)</td>
</tr>
</table>

Both of the above task variants are specializations of the following task *template*. The template is built from several so-called *variation points*, that are shown in the table below. Variation points are resolved to values by using the [mustache syntax](https://mustache.github.io/mustache.5.html):

<table>
<tr><th></th><th>description</th><th>difficulty</th><th>grading result</th></tr>
<tr><td>template</td><td>
<p>Write a static method <code>greet</code> as part of the Java class <code>{{class}}</code> that returns the message <code>&quot;{{msg}}{{#param}}&lt;name&gt;{{/param}}.&quot;</code> as a string{{#param}}, where the value of <code>&lt;name&gt;</code> is passed to <code>greet</code> as a method parameter{{/param}}.</p>
<p>{{#comment}}Include javadoc comments with your method.{{/comment}}</p>
<p>You should base your work on the prepared code template <code>{{class}}.java</code>.</p>
<td>{{difficulty}}</td>
<td>compile (20%), junit (60%), pmd (20%). If the variation point <em>comment</em> has <em>false</em> value, then exclude the pmd test and raise the weights of the compile and junit tests accordingly.</td>
</tr>
</table>

The following table shows the variation points, their meanings, and their value in both of the above variants.

| variation point | meaning                                                | data type | variant A     | variant B            |
|-----------------|--------------------------------------------------------|-----------|---------------|----------------------|
| class           | class name                                             | string    | "Program"     | "Welcome"            |
| param           | should method have a parameter?                        | boolean   | false         | true                 |
| msg             | Output message                                         | string    | "Hello world" | "Nice to meet you, " |
| comment         | should method get a javadoc comment                    | boolean   | false         | true                 |
| weight          | factor to be applied to compile and junit test weights | double    | 1.25          | 1.0                  |
| difficulty      | how difficult is the variant?                          | double    | 0.8           | 1.4                  |

One of the problems in specifying variable programming tasks is, that the variation point values depend on each other. Later in this white paper we will deal with this and propose a compact specification of all valid variants of all variation points of a task template.

### 1.2 Task template

The above example defines a task template
- by defining variation points,
- by including placeholders into artifacts,
- and by defining operations to be applied to the artifacts. Examples of such operations are:
  * the multiplication of a weight factor 
  * or the execution of the mustache template engine.
	
A variable programming task is a task template that is defined by a zip archive holding the following files:
- the `task.xml` file of a regular ProFormA task plus any additional artifacts referenced by `task.xml`,
- and a file `tpl.xml`.

The file `tpl.xml` (*tpl* is an abbreviation for *template*) contains a *templateSpec* element as the root element. This element includes the following child elements (all mandatory):
- a *varSpec* element specifies the variation points and the set of all valid combinations of variation point values ([section 2.3](#23-specification-of-all-valid-variants-of-all-variation-points-of-a-task-template)), 
- a *defaultValue* element specifies a valid combination of variation point values ​​as default ([section 2.2](#22-variant---variation-point-values)), 
- a *matSpec* element specifies the materializations, i. e. how to generate a particular task variant from a valid combination of variation point values ([section 3](#3-materialization-specification)). 



## 2 Variability specification

### 2.1 Variation point

The *vp* element defines a variation point. Variation points have a name (attribute *key*) and a type (attribute *type*). Basic variation point types are *integer*, *boolean*, *string*, etc. The variation point type *table* denotes a table type whose columns are recursively defined as variation points. The variation point type *double* also defines an *accuracy* attribute as the smallest value that can be distinguished from 0.

A *varSpec* element (XSD type *varSpecRoot*) usually defines several variation points, which are summarized in a *cvp* element (*composite variation point*).

###### Code

```xml
<v:cvp>
  <v:vp key="class" type="string"></v:vp>
  <v:vp key="param" type="boolean"></v:vp>
  <v:vp key="msg" type="string"></v:vp>
  <v:vp key="comment" type="boolean"></v:vp>
  <v:vp key="weight" accuracy="0.0" type="double"></v:vp>
  <v:vp key="difficulty" accuracy="0.0" type="double"></v:vp>
</v:cvp>
```

### 2.2 Variant - variation point values

Variation points can accept values (so-called variants). A primitive variant is represented by one of the following XSD types:

- *vi*: represents an integer variant
- *vd*: represents a double variant
- *vs*: represents a string variant
- *vb*: represents a boolean variant
- *vc*: represents a character variant

A bundle of all variants belonging to a set of variation points is stored in a *cv* element (*composite variant*). The *defaultValue* element specifies the default values. 

A special variant is that of a table variation point:

- *vt*: represents a table variant. The rows of a table variant are members of a list, where the list is expressed as a *cvList* element, which is a sequence of *cv* elements.

###### Code

```xml
<v:defaultValue>
  <v:string>Program</v:string>
  <v:boolean>false</v:boolean>
  <v:string>Hello world</v:string>
  <v:boolean>false</v:boolean>
  <v:double>1.25</v:double>
  <v:double>0.8</v:double>
</v:defaultValue>
```

### 2.3 Specification of all valid variants of all variation points of a task template

The set of all valid variants of all the variation points of a task template can become so large due to combination effects that it does not seem reasonable to list all combinations separately. Instead we specify the set of all valid variants (*varSpecRoot*) as a tree of the basic operations cartesian product, union set, and derivative function:

- The leaves of the tree usually specify values of individual variation points:
  * the *val* element specifies a single value, e. g. 
    ```xml
	<val>
	  <boolean>true</boolean>
	</val>
	```
  * the *collect* element specifies a set of values, e. g. 
    ```xml
	<collect>
	  <string>Message</string>
	  <string>Program</string>
	  <string>Welcome</string>
	</collect>
	```
  * the *range* element specifies a range of values, e. g. 
    ```xml
	<range>
	  <firstInteger>2</firstInteger>
	  <lastInteger>10</lastInteger>
	  <count>5</count>
	</range>
	```
	specifies the set { 2, 4, 6, 8 ,10 }
- A common case is that of having several individual variation points, that are independent of each other. These variation points are combined into an overall specification of all the variation points by a *combineGroup* element as a cartesian product. In this case, the *varSpec* element has a *combineGroup* element as it's only tree child:
  ```xml
  <v:varSpec>
    <v:combineGroup>
      <v:collect> <!-- This lists all values of "class" vp -->
        <v:string>Message</v:string>
        <v:string>Program</v:string>
        <v:string>Welcome</v:string>
      </v:collect>
      <v:collect> <!-- This lists all values of "comment" vp -->
        <v:boolean>true</v:boolean>
        <v:boolean>false</v:boolean>
      </v:collect>
    </v:combineGroup>
    <v:cvp>
      <v:vp key="class" type="string"></v:vp>
      <v:vp key="comment" type="boolean"></v:vp>
    </v:cvp>
  </v:varSpec>
  ```
- Deviating from the standard case of independent variation points, the valid variants can conversely be specified as the union set (*collectGroup*) at the root. In this case, tuples also occur as values ​​on the leaves (*combine*):
  ```xml
  <v:varSpec>
    <v:collectGroup>
      <v:combine>
        <v:boolean>false</v:boolean> <!-- a value of "param" vp -->
        <v:string>Hello world</v:string> <!-- a corresponding "msg" value -->
      </v:combine>
      <v:combine>
        <v:boolean>true</v:boolean>  <!-- another value of "param" vp -->
        <v:string>Nice to meet you, </v:string> <!-- a corresponding "msg" value -->
      </v:combine>
    </v:collectGroup>
    <v:cvp>
      <v:vp key="param" type="boolean"></v:vp>
      <v:vp key="msg" type="string"></v:vp>
    </v:cvp>
  </v:varSpec>
  ```
 
- By nesting *combineGroup* and *collectGroup* elements hierarchically, we can specify any combinations of parts of the space of all valid variants, whereby in the *key* elements (defined in XSD type *varSpecReorderingNode*) the names of the variation points spanning the respective subspace are named:
  ```xml
  <v:varSpec>
    <v:combineGroup>
      <v:collect> <!-- This lists all values of "class" vp -->
        <v:string>Message</v:string>
        <v:string>Program</v:string>
        <v:string>Welcome</v:string>
      </v:collect>
      <v:collectGroup> <!-- This lists all valid combinations of "param" and "msg" -->
        <v:combine>
          <v:boolean>false</v:boolean>
          <v:string>Hello world</v:string>
        </v:combine>
        <v:combineGroup> <!-- The "param" value true corresponds with three possible "msg" values -->
          <v:val>
            <v:boolean>true</v:boolean>
          </v:val>
          <v:collect>
            <v:string>Hello </v:string>
            <v:string>Welcome </v:string>
            <v:string>Nice to meet you, </v:string>
          </v:collect>
          <v:key>param</v:key> <!-- specifying key elements is optional but might enhance readability -->
          <v:key>msg</v:key>
        </v:combineGroup>
        <v:key>param</v:key>
        <v:key>msg</v:key>
      </v:collectGroup>
      <v:collect> <!-- This lists all values of "comment" vp -->
        <v:boolean>true</v:boolean>
        <v:boolean>false</v:boolean>
      </v:collect>
      <v:key>class</v:key>
      <v:key>param</v:key>
      <v:key>msg</v:key>
      <v:key>comment</v:key>
    </v:combineGroup>
    <v:cvp>
      <v:vp key="class" type="string"></v:vp>
      <v:vp key="param" type="boolean"></v:vp>
      <v:vp key="msg" type="string"></v:vp>
      <v:vp key="comment" type="boolean"></v:vp>
    </v:cvp>
  </v:varSpec>
  ```

- At the leaves specific values ​​as *vi*, *vd*, *vs*, etc. elements are specified. In the case of variation points of simple data types these are primitive values. In the case of a table variation point a *vt* element recursively contains a tree specification with a *spec* element (XSD type *varSpecNodeTable*). 
- The data model allows specifying the valid variants of a variation point as a derivation function of other variation points implemented in javascript. The *derive* element contains a *jsSource* child element specifying a javascript function `apply` with a single object parameter. The object parameter provides other variation point values as input values. The return type of that function (attribute *aggregateType*) is one of the following:
  * either a single *value*, e. g. `function apply(obj) { return obj.foo + 42; }`,
  * a javascript array (*collection*), e. g. `function apply(obj) { return [obj.foo, obj.bar, obj.baz]; }`,
  * or a javascript object defining a *range* of values, e. g. `function apply(obj) { return {first:2, last:10, count:obj.foo}; }`. 
  
  The following example shows two *derive* elements:
  ```xml
  <v:varSpec>
    <v:combineGroup>
      <v:collect> <!-- This lists all values of "param" vp -->
        <v:boolean>false</v:boolean>
        <v:boolean>true</v:boolean>
      </v:collect>
      <v:collect> <!-- This lists all values of "comment" vp -->
        <v:boolean>false</v:boolean>
        <v:boolean>true</v:boolean>
      </v:collect>
      <v:derive aggregateType="value"> <!-- This derives an appropriate "weight" value -->
        <v:jsSource>function apply(obj) { return obj.comment ? 1.0 : 1.25; }</v:jsSource>
      </v:derive>
      <v:derive aggregateType="value"> <!-- This derives an appropriate "difficulty" value -->
        <v:jsSource>function apply(obj) { return (obj.comment ? 1.0 : 0.8) * (obj.param ? 1.4 : 1.0); }</v:jsSource>
      </v:derive>
    </v:combineGroup>
    <v:cvp>
      <v:vp key="param" type="boolean"></v:vp>
      <v:vp key="comment" type="boolean"></v:vp>
      <v:vp key="weight" accuracy="0.0" type="double"></v:vp>
      <v:vp key="difficulty" accuracy="0.0" type="double"></v:vp>
    </v:cvp>
  </v:varSpec>
- Finally, the data model allows reusing subtrees (*define*, *ref*). A *define* element specifies any subset of the set of all valid vp values. The defined subset gets an identifier that can be included anywhere in the rest of the specification by a *ref* element:
  ```xml
  <v:varSpec>
    <v:define id="id1">
      <v:collectGroup>
        <v:combine>
          <v:string>Hello world</v:string> <!-- a value of "msg" vp -->
          <v:boolean>false</v:boolean> <!-- the corresponding "param" value -->
        </v:combine>
        <v:combine>
          <v:string>Nice to meet you, </v:string> <!-- another value of "msg" vp -->
          <v:boolean>true</v:boolean>  <!-- the corresponding "param" value -->
        </v:combine>
      </v:collectGroup>
      <v:key>msg</v:key>
      <v:key>param</v:key>
    </v:define>
    <v:collectGroup>
      <v:combineGroup>
        <v:combine>
          <v:boolean>true</v:boolean> <!-- a value of "comment" vp -->
          <v:double>1.0</v:double> <!-- the corresponding value of "weight" vp -->
        </v:combine>
        <v:ref id="id1"></v:ref> <!-- include the whole subset "id1" from above -->
      </v:combineGroup>
      <v:combineGroup>
        <v:combine>
          <v:boolean>false</v:boolean> <!-- another value of "comment" vp -->
          <v:double>1.25</v:double> <!-- the corresponding value of "weight" vp -->
        </v:combine>
        <v:ref id="id1"></v:ref> <!-- include the whole subset "id1" from above -->
      </v:combineGroup>
      <v:key>comment</v:key>
      <v:key>weight</v:key>
      <v:key>msg</v:key>
      <v:key>param</v:key>
    </v:collectGroup>
    <v:cvp>
      <v:vp key="param" type="boolean"></v:vp>
      <v:vp key="msg" type="string"></v:vp>
      <v:vp key="comment" type="boolean"></v:vp>
      <v:vp key="weight" accuracy="0.0" type="double"></v:vp>
    </v:cvp>
  </v:varSpec>
  ```
  As can be seen in this example, *collectGroup*, *combineGroup* and *define* nodes can have *key* child elements defining a node-local reordering of the keys.

## 3 Materialization specification

TODO



