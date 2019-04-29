# An XML exchange format for variable programming tasks

**Version 1.0 - SNAPSHOT**

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

| variation point | meaning                                                | data type | variant A     | variant B                 |
|-----------------|--------------------------------------------------------|-----------|---------------|---------------------------|
| class           | class name                                             | string    | "Program"     | "Welcome"                 |
| param           | should method have a parameter?                        | boolean   | false         | true                      |
| msg             | Output message                                         | string    | "Hello world" | "Nice to meet you,&nbsp;" |
| comment         | should method get a javadoc comment                    | boolean   | false         | true                      |
| weight          | factor to be applied to compile and junit test weights | double    | 1.25          | 1.0                       |
| difficulty      | how difficult is the variant?                          | double    | 0.8           | 1.4                       |

One of the problems in specifying variable programming tasks is, that the variation point values depend on each other. E. g. the variation point *difficulty* depends on the *param* and the *comment* variation points. Later in this white paper we will deal with this and propose a compact specification of all valid variants of all variation points of a task template.

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

The file `tpl.xml` (*tpl* is an abbreviation for *template*) contains a *template-spec* element as the root element. This element includes the following child elements (all mandatory):
- a *var-spec* element specifies the variation points and the set of all valid combinations of variation point values ([section 2.3](#23-specification-of-all-valid-variants-of-all-variation-points-of-a-task-template)), 
- a *default-value* element specifies a valid combination of variation point values ​​as default ([section 2.2](#22-variant---variation-point-values)), 
- a *mat-spec* element specifies the materializations, i. e. how to generate a particular task variant from the `task.xml` file and it's referenced artifacts and from a valid combination of variation point values ([section 3](#3-materialization-specification)). 

### 1.3 Task instance

A task variant - also called a task *instance* - is derived from a task template by selecting values for variation points and by applying materializations. The format specified in this white paper seeks to allow any system of the grading process chain to instantiate a task template - at least, if the task has no artifacts that are specific to some Grader or some programming language: the frontend learning management system (LMS), the backend grading system (Grader), and also a system in between serving as a broker or middleware between frontend and backend. 


## 2 Variability specification

### 2.1 Variation point

The *vp* element defines a variation point. Variation points have a name (attribute *key*) and a type (attribute *type*). Basic variation point types are *integer*, *boolean*, *string*, etc. The variation point type *table* denotes a table type whose columns are recursively defined as variation points. The variation point type *double* also defines an *accuracy* attribute as the smallest value that can be distinguished from 0.

A *var-spec* element (XSD type *root-type*) usually defines several variation points, which are summarized in a *cvp* element (*composite variation point*).

###### Code

The following XML fragment specifies the variation points of the above example.

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

Variation points can accept values (so-called *variants*). A primitive variant is represented by one of the following XSD types:

- *vi-type*: represents an integer variant
- *vd-type*: represents a double variant
- *vs-type*: represents a string variant
- *vb-type*: represents a boolean variant
- *vc-type*: represents a character variant

A bundle of all variants belonging to a set of variation points is stored in an element of XSD type *cv-type* (*composite variant*). The *default-value* element of type *cv-type* specifies the default values. 

A special variant is that of a table variation point:

- *vt-type*: represents a table variant. The rows of a table variant are members of a list, where the list is expressed as a *value* element of XSD type *cv-list-type*, which is a sequence of *element* elements of XSD type *cv-type*.

###### Code

The following XML fragment specifies variant A from the above example as the default value:

```xml
<v:default-value>
  <v:string>Program</v:string>
  <v:boolean>false</v:boolean>
  <v:string>Hello world</v:string>
  <v:boolean>false</v:boolean>
  <v:double>1.25</v:double>
  <v:double>0.8</v:double>
</v:default-value>
```

### 2.3 Specification of all valid variants of all variation points of a task template

The set of all valid variants of all the variation points of a task template can become so large due to combination effects that it does not seem reasonable to list all combinations separately. Instead we specify the set of all valid variants (*root-type*) as a tree of the basic operations cartesian product, union set, and derivative function:

- The leaves of the tree usually specify values of individual variation points:
  * the *val* element specifies a single value, e. g. 
    ```xml
	<v:val>
	  <v:boolean>true</v:boolean>
	</v:val>
	```
  * the *collect* element specifies a set of values, e. g. 
    ```xml
	<v:collect>
	  <v:string>Message</v:string>
	  <v:string>Program</v:string>
	  <v:string>Welcome</v:string>
	</v:collect>
	```
  * the *range* element specifies a range of values, e. g. 
    ```xml
	<v:range>
	  <v:first-integer>2</v:first-integer>
	  <v:last-integer>10</v:last-integer>
	  <v:count>5</v:count>
	</v:range>
	```
	specifies the set { 2, 4, 6, 8 ,10 }
- A common case is that of having several individual variation points, that are independent of each other. These variation points are combined into an overall specification of all the variation points by a *combine-group* element as a cartesian product. In this case, the *var-spec* element has a *combine-group* element as it's only tree child:
  ```xml
  <v:var-spec>
    <v:combine-group>
      <v:collect> <!-- This lists all values of "class" vp -->
        <v:string>Message</v:string>
        <v:string>Program</v:string>
        <v:string>Welcome</v:string>
      </v:collect>
      <v:collect> <!-- This lists all values of "comment" vp -->
        <v:boolean>true</v:boolean>
        <v:boolean>false</v:boolean>
      </v:collect>
    </v:combine-group>
    <v:cvp>
      <v:vp key="class" type="string"></v:vp>
      <v:vp key="comment" type="boolean"></v:vp>
    </v:cvp>
  </v:var-spec>
  ```
- Deviating from the standard case of independent variation points, the valid variants can conversely be specified as the union set (*collect-group*) at the root. In this case, tuples also occur as values ​​on the leaves (*combine*):
  ```xml
  <v:var-spec>
    <v:collect-group>
      <v:combine>
        <v:boolean>false</v:boolean> <!-- a value of "param" vp -->
        <v:string>Hello world</v:string> <!-- a corresponding "msg" value -->
      </v:combine>
      <v:combine>
        <v:boolean>true</v:boolean>  <!-- another value of "param" vp -->
        <v:string>Nice to meet you, </v:string> <!-- a corresponding "msg" value -->
      </v:combine>
    </v:collect-group>
    <v:cvp>
      <v:vp key="param" type="boolean"></v:vp>
      <v:vp key="msg" type="string"></v:vp>
    </v:cvp>
  </v:var-spec>
  ```
 
- By nesting *combine-group* and *collect-group* elements hierarchically, we can specify any combinations of parts of the space of all valid variants, whereby in the *key* elements (defined in XSD type *reordering-node-type*) the names of the variation points spanning the respective subspace are named:
  ```xml
  <v:var-spec>
    <v:combine-group>
      <v:collect> <!-- This lists all values of "class" vp -->
        <v:string>Message</v:string>
        <v:string>Program</v:string>
        <v:string>Welcome</v:string>
      </v:collect>
      <v:collect-group> <!-- This lists all valid combinations of "param" and "msg" -->
        <v:combine>
          <v:boolean>false</v:boolean>
          <v:string>Hello world</v:string>
        </v:combine>
        <v:combine-group> <!-- The "param" value true corresponds with three possible "msg" values -->
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
        </v:combine-group>
        <v:key>param</v:key>
        <v:key>msg</v:key>
      </v:collect-group>
      <v:collect> <!-- This lists all values of "comment" vp -->
        <v:boolean>true</v:boolean>
        <v:boolean>false</v:boolean>
      </v:collect>
      <v:key>class</v:key>
      <v:key>param</v:key>
      <v:key>msg</v:key>
      <v:key>comment</v:key>
    </v:combine-group>
    <v:cvp>
      <v:vp key="class" type="string"></v:vp>
      <v:vp key="param" type="boolean"></v:vp>
      <v:vp key="msg" type="string"></v:vp>
      <v:vp key="comment" type="boolean"></v:vp>
    </v:cvp>
  </v:var-spec>
  ```

- At the leaves, as children of *val*, *collect*, *combine*, and *range*, specific values ​​as *vi-type*, *vd-type*, *vs-type*, etc. elements are specified. The order of these values matches that of the order of variation points defined in the *cvp* element. The type of these values matches the types of the variation points specified in the *cvp* element. In the case of variation points of simple data types these are primitive values. In the case of a table variation point a *vt-type* element recursively contains a tree specification with a *spec* element (XSD type *table-type*). 
- The data model allows specifying the valid variants of a variation point as a derivation function of other variation points implemented in javascript. The *derive* element contains a *js-source* child element specifying a javascript function `apply` with a single object parameter. The object parameter provides other variation point values as input values. The return type of that function (attribute *aggregate-type*) is one of the following:
  * either a single *value*, e. g. `function apply(obj) { return obj.foo + 42; }`,
  * a javascript array (*collection*), e. g. `function apply(obj) { return [obj.foo, obj.bar, obj.baz]; }`,
  * or a javascript object defining a *range* of values, e. g. `function apply(obj) { return {first:2, last:10, count:obj.foo}; }`. 
  
  The following example shows two *derive* elements:
  ```xml
  <v:var-spec>
    <v:combine-group>
      <v:collect> <!-- This lists all values of "param" vp -->
        <v:boolean>false</v:boolean>
        <v:boolean>true</v:boolean>
      </v:collect>
      <v:collect> <!-- This lists all values of "comment" vp -->
        <v:boolean>false</v:boolean>
        <v:boolean>true</v:boolean>
      </v:collect>
      <v:derive aggregate-type="value"> <!-- This derives an appropriate "weight" value -->
        <v:jsSource>function apply(obj) { return obj.comment ? 1.0 : 1.25; }</v:jsSource>
      </v:derive>
      <v:derive aggregate-type="value"> <!-- This derives an appropriate "difficulty" value -->
        <v:jsSource>function apply(obj) { return (obj.comment ? 1.0 : 0.8) * (obj.param ? 1.4 : 1.0); }</v:jsSource>
      </v:derive>
    </v:combine-group>
    <v:cvp>
      <v:vp key="param" type="boolean"></v:vp>
      <v:vp key="comment" type="boolean"></v:vp>
      <v:vp key="weight" accuracy="0.0" type="double"></v:vp>
      <v:vp key="difficulty" accuracy="0.0" type="double"></v:vp>
    </v:cvp>
  </v:var-spec>
- Finally, the data model allows reusing subtrees (*def*, *ref*). A *def* element specifies any subset of the set of all valid vp values. The defined subset gets an identifier that can be included anywhere in the rest of the specification by a *ref* element:
  ```xml
  <v:var-spec>
    <v:def id="id1">
      <v:collect-group>
        <v:combine>
          <v:string>Hello world</v:string> <!-- a value of "msg" vp -->
          <v:boolean>false</v:boolean> <!-- the corresponding "param" value -->
        </v:combine>
        <v:combine>
          <v:string>Nice to meet you, </v:string> <!-- another value of "msg" vp -->
          <v:boolean>true</v:boolean>  <!-- the corresponding "param" value -->
        </v:combine>
      </v:collect-group>
      <v:key>msg</v:key>
      <v:key>param</v:key>
    </v:def>
    <v:collect-group>
      <v:combine-group>
        <v:combine>
          <v:boolean>true</v:boolean> <!-- a value of "comment" vp -->
          <v:double>1.0</v:double> <!-- the corresponding value of "weight" vp -->
        </v:combine>
        <v:ref id="id1"></v:ref> <!-- include the whole subset "id1" from above -->
      </v:combine-group>
      <v:combine-group>
        <v:combine>
          <v:boolean>false</v:boolean> <!-- another value of "comment" vp -->
          <v:double>1.25</v:double> <!-- the corresponding value of "weight" vp -->
        </v:combine>
        <v:ref id="id1"></v:ref> <!-- include the whole subset "id1" from above -->
      </v:combine-group>
      <v:key>comment</v:key>
      <v:key>weight</v:key>
      <v:key>msg</v:key>
      <v:key>param</v:key>
    </v:collect-group>
    <v:cvp>
      <v:vp key="param" type="boolean"></v:vp>
      <v:vp key="msg" type="string"></v:vp>
      <v:vp key="comment" type="boolean"></v:vp>
      <v:vp key="weight" accuracy="0.0" type="double"></v:vp>
    </v:cvp>
  </v:var-spec>
  ```
  As can be seen in this example, *collect-group*, *combine-group* and *def* nodes can have *key* child elements defining a node-local reordering of the keys.

## 3 Materialization specification

A *mat-spec* element (materialization specification) is a sequence of *materialization* elements. Each such element associates artifacts with so-called materialization methods. Artifacts can be any part of the task template. Common artifacts are the `task.xml` file or additional files that are included with the zip archive. An artifact may address the content of a file, but also the file name or it's mere existence. Artifacts are considered "aspects" of the task template that can be altered by *methods*. Artifacts have types. A file content artifact of a text file has string type while an existence artifact has boolean type.

A common *method* that can be applied to an artifact of string type is a *search and replace* operation. A common method that can be applied to an artifact of boolean type is a *set* operation, that sets the artifact to true (meaning preserving existence) or false (meaning deletion).

Besides the above string and boolean artifact examples, this standard defines artifacts of numeric type. Grading weights like the "25%" of the compile test in the introductory example are numeric artifacts. Such artifacts can be altered by applying an arithmetic operation (addition, subtraction, multiplication, ...) to it.

### 3.1 Materialization artifacts

A *mat-artifact* element has mandatory *id* and *artifact-type* attributes. Currently the following artifact-types are available:

- *task-xml*: denotes the content of the `task.xml` file. This artifact is of data type string.
- *attached-txt-file-contents*: denotes the content of one or more text files included with the zip archive. The files are referenced either by nested *file-id* elements or by nested *path* elements. The *path* element expects a path relative to the zip archive root as it's element content. This artifact is of data type string.
- *file-names*: denotes the file names of one or more files included with the zip archive. The files are referenced either by nested *file-id* elements or by nested *path* elements. This artifact is of data type string.
- *file-existences*: denotes the existence of one or more files included with the zip archive. The files are referenced either by nested *file-id* elements or by nested *path* elements. This artifact is of data type boolean.
- *grading-node-weights*: denotes the weight of one or more *test-ref* or *combine-ref* elements inside the grading-hints hierarchy of the task. This artifact is of data type double. The respective parts of the grading-hints hierarchy are referenced by nested *ref* elements. Each such *ref* element consists of
  * a mandatory *type* attribute (valid values are *test* and *combine*)
  * a mandatory *ref* attribute referencing the corresponding *test-ref* or *combine-ref* element inside the grading-hints hierarchy of the task.
  * a *sub-ref* attribute that is optional for type=test and forbidden for type=combine. The sub-ref attribute references the corresponding *test-ref* element inside the grading-hints hierarchy.
- *grading-node-existences*: denotes the existence of one or more *test-ref* or *combine-ref* elements inside the grading-hints hierarchy of the task. As for *grading-node-weights* the *ref* child elements refer the respective parts of the grading-hints hierarchy. This artifact is of data type boolean.
- *other*: This is an extension point for any Grader-specific artifacts. This artifact defines it's data type in an additional *data-type* attribute.


### 3.2 Example

In order to give an example, we first list some relevant parts of the task template that belongs to the introductory example. We assume an imaginary Grader that is capable of running compilation, JUnit and PMD tests.

First, the content of the zip archive of the task template is listed as follows. We include the usual `task.xml` file and several attachments:

```text
├── task.xml
├── attachment
│   ├── Grader.java            (needed for the JUnit test)
│   ├── __class__.java         (template given to students)
│   └── ruleset1.xml           (for the PMD test)
├── samplesolutions
│   └── __class__.java         (a model solution)
└── tpl.xml
```

The content of the `task.xml` file specifies the task's description and several tests. Inside `task.xml` we use `§(...)§` to denote placeholders. This standard does not prescribe the syntax of placeholders as long as the leading and trailing sequences both consist of exactly two characters. Later as part of a materialization method we specify the specific prefix `§(` and the specific suffix `§)` used in the following example.

```xml
<?xml version="1.0" ?>
<p:task xmlns:p="urn:proforma:v2.0.1" xmlns:u="urn:proforma:tests:unittest:v1.1" uuid="..." lang="en">
  <p:title>Units</p:title>
  <p:description><![CDATA[
    <p>Write a static method <code>greet</code> as part of the Java class 
	<code>§(class)§</code> that returns the message 
	<code>&quot;§(msg)§§(#param)§&lt;name&gt;§(/param)§.&quot;</code> as a 
	string§(#param)§, where the value of <code>&lt;name&gt;</code> 
	is passed to <code>greet</code> as a method parameter§(/param)§.</p>
    <p>§(#comment)§Include javadoc comments with your method.§(/comment)§</p>
    <p>You should base your work on the prepared code template 
	<code>§(class)§.java</code>.</p>
  ]]></p:description>
  <p:internal-description/>
  <p:proglang version="8">java</p:proglang>
  <p:submission-restrictions max-size="10000">
    <p:file-restriction required="true" pattern-format="none">§(class)§.java</p:file-restriction>
    <p:file-restriction required="false" pattern-format="posix-ere">^/[^/]+\.java$</p:file-restriction>
  </p:submission-restrictions>
  <p:files>
    <p:file id="testdriver" used-by-grader="true" visible="no">
      <p:attached-txt-file encoding="UTF-8">attachment/Grader.java</p:attached-txt-file>
    </p:file>
    <p:file id="pmdcfg" used-by-grader="true" visible="no">
      <p:attached-txt-file encoding="UTF-8">attachment/ruleset1.xml</p:attached-txt-file>
    </p:file>
    <p:file id="template" used-by-grader="false" visible="yes">
      <p:attached-txt-file encoding="UTF-8">attachment/__class__.java</p:attached-txt-file>
    </p:file>
    <p:file id="ms" used-by-grader="false" visible="no">
      <p:attached-bin-file>samplesolutions/__class__.java</p:attached-bin-file>
    </p:file>
  </p:files>
  <p:model-solutions>
    <p:model-solution id="correct">
      <p:filerefs><p:fileref refid="ms"/></p:filerefs>
    </p:model-solution>
  </p:model-solutions>
  <p:tests>
    <p:test id="compile"><p:title>Compilation</p:title><p:test-type>java-compilation</p:test-type>
      <p:test-configuration/>
    </p:test>
    <p:test id="junit"><p:title>JUnit dynamic analysis</p:title><p:test-type>unittest</p:test-type>
      <p:test-configuration>
        <p:filerefs><p:fileref refid="testdriver"/></p:filerefs>
        <u:unittest framework="junit" version="4">
          <u:entry-point>dom.msg.grader.Grader</u:entry-point>
        </u:unittest>
      </p:test-configuration>
    </p:test>
    §(#comment)§
    <p:test id="pmd"><p:title>PMD static analysis</p:title><p:test-type>java-pmd</p:test-type>
      <p:test-configuration>
        <p:filerefs><p:fileref refid="pmdcfg"/></p:filerefs>
      </p:test-configuration>
    </p:test>
    §(/comment)§
  </p:tests>
  <p:grading-hints>
    <p:root function="sum">
      <p:test-ref ref="compile" weight="0.2">
        <p:title>Should successfully compile</p:title>
      </p:test-ref>
      <p:test-ref ref="junit" weight="0.6">
        <p:title>Should implement method greet in class §(class)§.</p:title>
      </p:test-ref>
      §(#comment)§
      <p:test-ref ref="pmd" weight="0.2">
        <p:title>Comments needed in front of methods and classes</p:title>
      </p:test-ref>
      §(/comment)§
    </p:root>
  </p:grading-hints>
  <p:meta-data/>
</p:task>
```

### 3.3 Materialization artifacts of the example

We illustrate materialization artifacts for the previous example. The following XML fragment of `tpl.xml` defines five artifacts:

```xml
  <v:mat-spec>
    <v:mat-artifacts>
      <v:mat-artifact id="a1" artifact-type="task-xml"/>
      <v:mat-artifact id="a2" artifact-type="attached-txt-file-contents">
        <v:file-id>template</v:file-id>
        <v:path>samplesolutions/__class__.java</v:path>
      </v:mat-artifact>
      <v:mat-artifact id="a3" artifact-type="file-names">
        <v:file-id>template</v:file-id>
        <v:path>samplesolutions/__class__.java</v:path>
      </v:mat-artifact>
      <v:mat-artifact id="a4" artifact-type="file-existences">
        <v:file-id>pmdcfg</v:file-id>
      </v:mat-artifact>
      <v:mat-artifact id="a5" artifact-type="grading-hints-weights">
        <v:ref ref="compile" type="test"/>
        <v:ref ref="junit" type="test"/>
      </v:mat-artifact>
    </v:mat-artifacts>
	[...]
  </v:mat-spec>
```

### 3.4 Materialization methods

A *mat-method* element has mandatory *id* and *method-type* attributes. Currently the following method-types are available:

- *mustache*: denotes the search-and-replace operation performed by the mustache template engine. The mustache is configured by optional *prefix* and *suffix* attributes. These attributes should specify a two-character prefix and a two-character suffix. If prefix or suffix are missing, the defaults are `{{` and `}}`, respectively.
- *set-vp-value*: denotes the operation of writing the actual value of a variation point to the artifact. The selected variation point should be specified in the *restrict-vp* child element. With this operation we can realize a file deletion operation, if we connect a *file-existences* artifact with a boolean variation point.
- *arithmetic-operation*: denotes the operation of adding, multiplying, etc. the value of a specified variation point to an artifact. This method reasonably can be used together with artifacts of a numeric type like the *grading-hints-weights* artifact. The selected variation point should be specified in the *restrict-vp* child element. With this operation we can realize the grading weights modification of the compile and JUnit tests in case of task variant A of the introductory example.
- *other*: This is an extension point for any Grader-specific methods.

### 3.5 Materialization methods of the example

We illustrate materialization methods for our example. The following XML fragment of `tpl.xml` defines four methods. The first and the second method are variations of the mustache method with different prefixes and suffixes. The first method *m1* is applied to the content of `task.xml` and to the content of the template and the model solution. The second method *m2* is applied to the file names of the `__class__.java` files. The method *m3* reads the value of the *comment* variation point. If the value is false, then the artifact *a4* is set to false, i. e. the PMD configuration file is removed from the zip archive's `attachments` folder. Finally the method *m4* multiplies the value 1.25 to the grading weights of the compile and junit tests, if the *comment* variation point value is false.

```xml
  <v:mat-spec>
    <v:mat-artifacts> ... as above ... 
    </v:mat-artifacts>
    <v:mat-methods>
      <v:mat-method id="m1" method-type="mustache" prefix="§(" suffix=")§"/>
      <v:mat-method id="m2" method-type="mustache" prefix="__" suffix="__">
        <v:restrict-vp>class</v:restrict-vp>
      </v:mat-method>
      <v:mat-method id="m3" method-type="set-vp-value">
        <v:restrict-vp>comment</v:restrict-vp>
      </v:mat-method>
      <v:mat-method id="m4" method-type="arithmetic-operation" operator="mul">
        <v:restrict-vp>weight</v:restrict-vp>
      </v:mat-method>
    </v:mat-methods>
    <v:materializations>
      <v:materialization>
        <v:artifact-id>a1</v:artifact-id>
        <v:artifact-id>a2</v:artifact-id>
        <v:method-id>m1</v:method-id>
      </v:materialization>
      <v:materialization>
        <v:artifact-id>a3</v:artifact-id>
        <v:method-id>m2</v:method-id>
      </v:materialization>
      <v:materialization>
        <v:artifact-id>a4</v:artifact-id>
        <v:method-id>m3</v:method-id>
      </v:materialization>
      <v:materialization>
        <v:artifact-id>a5</v:artifact-id>
        <v:method-id>m4</v:method-id>
      </v:materialization>
    </v:materializations>
  </v:mat-spec>
```

### 3.6 Materializations

A *materialization* element includes one or more *artifact-id* elements and one or more *method-id* elements. These elements refer to the respective artifacts and methods. 

The *mat-spec* element has three child elements:
- *mat-artifacts*: includes all *mat-artifact* elements
- *mat-methods*: includes all *mat-method* elements
- *materializations*: includes all *materialization* elements

When an LMS, a middleware or a Grader executes the specifications in the *mat-spec* element, it is important to note, that the `task.xml` file from the task template usually is not a valid XML file. This is, because there might be mustache-statements in `task.xml` that corrupt the XML wellformedness. The *artifact-type*s of [section 3.1](#31-materialization-artifacts) differ in their dependency on a valid `task.xml` file. All but the first artifact type *task-xml* do include references to information that is stored in the `task.xml` file. Hence, artifacts need to be materialized in proper order: 
1. In a first phase the *task-xml* artifact must get materialized, so that the resulting materialized `task.xml` file can be loaded as a valid XML file. 
2. After that in a second phase, the other artifacts are materialized. The references from the second phase's artifacts (*file-id*, *test-ref*, etc.) to task elements are interpreted as references to elements of the materialized version of the `task.xml`, that has been created at the end of the first phase.

### 3.7 Grader-specific materializations

The above artifacts and methods might not suffice the requirements of any programming task. Consider a task defining some Grader-specific binary files, whose generated content depends on the values of some variation points. These files must be re-generated by the Grader itself. A task template including such specific artifacts can include specific materialization instructions:

- The materialization artifact type *other* can be used together with any non-standard information in an xs:any element. Mandatory attributes for *other* artifacts are:
  * the attribute *operates-on-task-xml* specifies, whether the artifact should be executed together with the *task-xml* artifact in the first phase (true) or not (false).
  * the attribute *data-type* specifies the data type, just like [all standard artifacts](#31-materialization-artifacts) have their own data type.  
- the materialization method type *other* can be used together with any non-standard information in an xs:any element,
- the *mat-spec* element can include any non-standard information in an xs:any element.


## 4 Task instances

When materializing a task template, the resulting task instance should include the values of all variation points in order to allow a Grader to access the values e. g. for reporting purposes. In this specification we propose to include the variation points and their selected values as part of a task instance's `task.xml` file in the *meta-data* section. The meta-data element should include a *cvvp* element, that has the following content:
- a *cvp* element specifying the variation points
- a *cv* element specifying the selected values.

### 4.1 Example

The `task.xml` file of task variant A of our introductory example includes the following XML fragment:

```xml
  <p:meta-data>
    <v:cvvp>
      <v:cvp>
        <v:vp key="class" type="string"></v:vp>
        <v:vp key="param" type="boolean"></v:vp>
        <v:vp key="msg" type="string"></v:vp>
        <v:vp key="comment" type="boolean"></v:vp>
        <v:vp key="weight" accuracy="0.0" type="double"></v:vp>
        <v:vp key="difficulty" accuracy="0.0" type="double"></v:vp>
      </v:cvp>
      <v:cv>
        <v:string>Program</v:string>
        <v:boolean>false</v:boolean>
        <v:string>Hello world</v:string>
        <v:boolean>false</v:boolean>
        <v:double>1.25</v:double>
        <v:double>0.8</v:double>
      </v:cv>
    </v:cvvp>
  </p:meta-data>
```

