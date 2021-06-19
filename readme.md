
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>

<h3 align='center'>
  XO
</h3>
<p align='center'>
  The XO Specification Language
</p>

<br/>
<br/>
<br/>

### Introduction

The XO Specification Language (pronounced "_show_"), is a little more than a markup language, tending toward a programming language. In fact, it can be used for a programming language. It is a way to model information and computation in an easy to read and write format, suitable for hierarchical note taking and other means of capturing data down into structured form.

### Examples

Here are a few examples from existing code. The first is how you might define a "stack" (a "package" of XO code). The second is the first part of the Tao Te Ching captured in a tree, and the third how you might write a simple fibonacci function either iteratively or recursively. These examples are DSLs designed for a specific purposes. As you will see in the syntax section, the XO language is independent of a DSL and simply defines some simple idioms for defining trees of code and text.

A package definition:

```
stack @mountbuild/stone
  brief |Stone Compiler|
  brand |Stone|
  brand |Computation|
  brand |Philosophy|
  brand |Information|
  brand |Platform|
  brand |White Label|
  touch |Lance Pollard <lp@elk.fm>|
  mount ./mount
```

The first block of the Tao Te Ching:

```
block |道德经|
  block |第一章|
    block |道可道，非恆道；|
    block |名可名，非恆名。|
    block |無名天地之始；|
    block |有名萬物之母。|
    block |故，|
    block |恆無，欲也，以觀其妙；|
    block |恆有，欲也，以觀其徼。|
    block |此兩者同出而異名，|
    block |同謂之玄。|
    block |玄之又玄，眾妙之門。|
```

And the fibonacci functions:

```
force find-fibonacci-via-loop
  start i

  state g, 0
  state o, 1
  state d

  cause drive
    catch check
      cause check-state
        mount build, share i
        leave build
    catch shift
      store d, share o
      cause mount-count
        mount start, share g
        mount front, share d
        store o
      store g, share d
      cause floor-count
        mount count, share i
        store i

  leave build, share g

force find-fibonacci-via-recursion
  start i
  start g, fault 0
  start o, fault 1

  cause check-blank
    mount block, share i
    catch match
      leave build, share g
    catch fault
      cause clear-count
        mount start, share i
        mount front, 1
        store d
      cause mount-count
        mount start, share o
        mount front, share g
        store t
      cause find-fibonacci-via-recursion
        mount i, share d
        mount g, share o
        mount o, share t
        leave build
```

Now we will go into the actual specification of the syntax.

### Syntax

The XO specification language is a minimal markup language that is transformable into code. It has the following syntax.

#### Terms

The first thing to cover are _terms_. They are composed of _words_, separated by dashes, into what are called _keys_. A word is composed of lowercase ascii letters or numbers. So the following are all keys of a term.

```
xo
hello-world
foo-bar-baz
```

They get compiled into objects, such as this one:

```json
{
  "type": "term",
  "name": "hi"
}
```


You can nest them arbitrarily into trees, and there are no rules on what can be nested into what else. You can nest whatever you need to do model your data in a nice and clean DSL.

A more complex example is:

```
hello world
```

Getting compiled to:

```json
{
  "type": "term",
  "name": "hello",
  "children": [
    {
      "type": "term",
      "name": "world"
    }
  ]
}
```

You can go as far as you want:

```
this is a tree
```

That is the same thing as this:

```
this
  is
    a
      tree
```

You can write multiple nodes on a line separated by comma:

```
this is, also a tree, and a tree
```

The same as:

```
this is
  also
    a tree
      and a tree
```

You can put things in parentheses too to make it easier to write on one line:

```
add(a, subtract(b, c))
```

The same as:

```
add a, subtract b, c
```

#### Numbers

You can use numbers in the system too:

```
add 1, 2
```

Which is compiled to:

```json
{
  "type": "term",
  "name": "add",
  "children": [
    {
      "type": "number",
      "value": 1
    },
    {
      "type": "number",
      "value": 2
    }
  ]
}
```

A more complex example would be:

```
add 1, subtract 2, 3
```

#### Templates

A more complex structure is the _template_. They are composed of a weaving of _strings_ and _terms_. A string is a contiguous sequence of arbitrary unicode (utf-8).

A simple template composed only of a string is:

```
write |hello world|
```

Which gets compiled to:

```json
{
  "type": "term",
  "name": "write",
  "children": [
    {
      "type": "template",
      "children": [
        {
          "type": "string",
          "value": "hello world"
        }
      ]
    }
  ]
}
```

Or multiline strings:

```
text |
  This is a long paragraph.

  And this is another paragraph.
|
```

Or even:

```
text
  |
    This is a long paragraph.

    And this is another paragraph.
  |
```

Note, there are no "comments" in the system. Comments are just strings we don't care about in code. So if you end up transforming XO into code, you would just get rid of any parts of the model with text nodes you consider "comments".

Then we can add interpolation into the template, by referencing terms wrapped in colons:

```
write |:hello-world:|
```

That is compiled into:

```json
{
  "type": "term",
  "name": "write",
  "children": [
    {
      "type": "template",
      "children": [
        {
          "type": "term",
          "name": "hello-world"
        }
      ]
    }
  ]
}
```

A more robust example might be:

```
moon |The moon has a period of roughly :days: days.|
```

```json
{
  "type": "term",
  "name": "moon",
  "children": [
    {
      "type": "template",
      "children": [
        {
          "type": "string",
          "name": "The moon has a period of roughly "
        },
        {
          "type": "term",
          "name": "days"
        },
        {
          "type": "string",
          "name": " days"
        }
      ]
    }
  ]
}
```

You can also do complex inline terms or multiline nested terms.

```
i |
  am a :
    complex
      nested mutiline-term
  :
|

and |I am a :complex inline-term:|
```

Note though, you can still use the colon symbols in regular text without ambiguity, you just need to prefix them with backslashes.

```
i |am \:colons\: included in the actual string|
```

If the code is simple enough, you can just use 1 colon, as long as it is clear what the end is from context:

```
text |I am :code(|text|)|
text |There are :size days a year|
```

#### Codes

You can write specific code points, or _codes_, by prefixing the number sign / hash symbol along with a letter representing the code type, followed by the code.

```
i #b0101, am bits
i #u2665, am unicode
i #haaaaaa, am hex
i #o123, am octal
```

```json
{
  "type": "code",
  "variant": "b",
  "value": "0101"
}
```

These can also be used directly in a template:

```
i |am the symbol #u2665|
```

This makes it so you can reference obscure symbols by their numerical value, or write bits and things like that. Note though, these just get compiled down to the following, so the code handler would need to resolve them properly in the proper context.

#### Paths

Because paths are so common in programming, they don't need to be treated as strings but can be written directly.

```
load @some/path
load ./relative/path.png
load /an-absolute/other/path.js
```

Let's see how each of these are compiled:

```json
{
  "type": "term",
  "name": "load",
  "children": [
    {
      "type": "template",
      "children": [
        {
          "type": "string",
          "value": "@some/path"
        }
      ]
    }
  ]
}
```

```json
{
  "type": "term",
  "name": "load",
  "children": [
    {
      "type": "template",
      "children": [
        {
          "type": "string",
          "value": "./relative/path.png"
        }
      ]
    }
  ]
}
```

```json
{
  "type": "term",
  "name": "load",
  "children": [
    {
      "type": "template",
      "children": [
        {
          "type": "string",
          "value": "/an-absolute/other/path.js"
        }
      ]
    }
  ]
}
```

That is, they are just special strings. You can interpolate on them like strings as well with square brackets.

#### Selectors

Selectors are like drilling down into terms. They look like paths, but they are really drilling down into terms, if you think of it that way.

```
get foo/bar
```

This gets compiled to:

```json
{
  "type": "term",
  "name": "get",
  "children": [
    {
      "type": "selector",
      "children": [
        {
          "type": "node",
          "name": "foo"
        },
        {
          "type": "node",
          "name": "bar"
        }
      ]
    }
  ]
}
```

You can interpolate on these as well, like doing array index lookup.

```
get node/children[i]/name
```

This gets compiled to:

```json
{
  "type": "term",
  "name": "get",
  "children": [
    {
      "type": "selector",
      "children": [
        {
          "type": "node",
          "name": "node"
        },
        {
          "type": "node",
          "name": "children",
          "children": [
            {
              "type": "node",
              "name": "i"
            }
          ]
        },
        {
          "type": "node",
          "name": "name"
        }
      ]
    }
  ]
}
```

The interpolations can be nested as well, and chained. Here is a complex example:

```
get foo/bar[x][o/children[i]/name]/value
```

### Discussion

With just these parts, you have a syntax for a robust programming language. Note, there are very little "special" syntaxes outside of the core term tree. There are no "operators" like binary operators such as `+` and `-`, or `&&`, or anything like that. There are just terms, templates, strings, numbers, codes, paths, and selectors.

You can write code or data in the same way. The key is figuring out the right DSL, and how to transform it into a core data model. For the purposes of this repo, the compiler gives you a tree of JSON. You are then free to transform it however you'd like and make it into data, code, or whatever else. It is general enough to serve that purpose.

<h3 id="license">License</h3>

Copyright 2021 <a href='https://mount.build'>Mount</a>

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

<h3 id="mount">Mount</h3>

XO is being developed by the folks at [Mount](https://mount.build), a California-based project for helping humanity master information and computation. Mount started off in the winter of 2008 as a spark of an idea, to forming a company 10 years later in the winter of 2018, to a seed of a project just beginning its development phases. Mount funds tone-script's development. It is entirely bootstrapped by working full time and running [Etsy](https://etsy.com/shop/mountbuild) and [Amazon](https://www.amazon.com/s?rh=p_27%3AMount+Build) shops. Also find us on [Facebook](https://www.facebook.com/mountbuild), [Twitter](https://twitter.com/mountbuild), and [LinkedIn](https://www.linkedin.com/company/mountbuild). Check out our other GitHub projects as well!

<br/>
<br/>
