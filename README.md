<!---
Created from:
* the original README file at https://github.com/javacc/javacc
* the documentation https://github.com/javacc/javacc/www
* an example README.md file from https://github.com/apache/flink
-->

# JavaCC

JavaCC is an open source compiler compiler. It takes a grammar specification as input and produces a standalone parser written in Java.

This README is meant as a brief overview of the core features and how to set things up to get yourself started with JavaCC. For a fully detailed documentation, please
see [https://javacc.org/](https://javacc.org/).

## <a name="toc"></a>Table of Contents

- [Overview](#overview)
   * [Features](#features)
   * [Example](#example)
- [Getting Started](#getting-started)
   * [Building JavaCC from source](#building-from-source)
   * [Developing JavaCC](#developing)
   * [Download](#download)
   * [Binary distribution](#binary-distribution)
- [Community](#community)
   * [Support](#support)
   * [Documentation](#documentation)
   * [Powered by JavaCC](#powered-by)
<!--- - [Contributing](#contributing) -->
- [License](#license)

## <a name="overview"></a>Overview

### <a name="features"></a>Features

* JavaCC generates top-down ([recursive descent](https://en.wikipedia.org/wiki/Recursive_descent_parser)) parsers as opposed to bottom-up parsers generated by [YACC](https://en.wikipedia.org/wiki/Yacc)-like tools. This allows the use of more general grammars, although [left-recursion](https://en.wikipedia.org/wiki/Left_recursion) is disallowed. Top-down parsers have a number of other advantages (besides more general grammars) such as being easier to debug, having the ability to parse to any [non-terminal](https://en.wikipedia.org/wiki/Terminal_and_nonterminal_symbols) in the grammar, and also having the ability to pass values (attributes) both up and down the parse tree during parsing.

* By default, JavaCC generates an `LL(1)` parser. However, there may be portions of grammar that are not `LL(1)`. JavaCC offers the capabilities of syntactic and semantic lookahead to resolve shift-shift ambiguities locally at these points. For example, the parser is `LL(k)` only at such points, but remains `LL(1)` everywhere else for better performance. Shift-reduce and reduce-reduce conflicts are not an issue for top-down parsers.

* JavaCC generates parsers that are 100% pure Java, so there is no runtime dependency on JavaCC and no special porting effort required to run on different machine platforms.

* JavaCC allows [extended BNF](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_form) specifications - such as `(A)*`, `(A)+` etc - within the lexical and the grammar specifications. Extended BNF relieves the need for left-recursion to some extent. In fact, extended BNF is often easier to read as in `A ::= y(x)*` versus `A ::= Ax|y`.

* The lexical specifications (such as regular expressions, strings) and the grammar specifications (the BNF) are both written together in the same file. It makes grammars easier to read since it is possible to use regular expressions inline in the grammar specification, and also easier to maintain.

* The [lexical analyzer](https://en.wikipedia.org/wiki/Lexical_analysis) of JavaCC can handle full Unicode input, and lexical specifications may also include any Unicode character. This facilitates descriptions of language elements such as Java identifiers that allow certain Unicode characters (that are not ASCII), but not others.

* JavaCC offers [Lex](https://en.wikipedia.org/wiki/Lex_(software))-like lexical state and lexical action capabilities. Specific aspects in JavaCC that are superior to other tools are the first class status it offers concepts such as `TOKEN`, `MORE`, `SKIP` and state changes. This allows cleaner specifications as well as better error and warning messages from JavaCC.

* Tokens that are defined as *special tokens* in the lexical specification are ignored during parsing, but these tokens are available for processing by the tools. A useful application of this is in the processing of comments.

* Lexical specifications can define tokens not to be case-sensitive either at the global level for the entire lexical specification, or on an individual lexical specification basis.

* JavaCC comes with JJTree, an extremely powerful tree building pre-processor.

* JavaCC also includes JJDoc, a tool that converts grammar files to documentation files, optionally in HTML.

* JavaCC offers many options to customize its behavior and the behavior of the generated parsers. Examples of such options are the kinds of Unicode processing to perform on the input stream, the number of tokens of ambiguity checking to perform etc.

* JavaCC error reporting is among the best in parser generators. JavaCC generated parsers are able to clearly point out the location of parse errors with complete diagnostic information.

* Using options `DEBUG_PARSER`, `DEBUG_LOOKAHEAD`, and `DEBUG_TOKEN_MANAGER`, users can get in-depth analysis of the parsing and the token processing steps.

* The JavaCC release includes a wide range of examples including Java and HTML grammars. The examples, along with their documentation, are a great way to get acquainted with JavaCC.


### <a name="example"></a>Example

This example recognizes matching braces followed by zero or more line terminators and then an end of file.

Examples of legal strings in this grammar are:

  `"{}"`, `"{{{{{}}}}}"` etc.

Examples of illegal strings are:

  `"{{{{"`, `"{}{}"`, `"{}}"`, `"{{}{}}"`, `"{ }"`, `"{x}"` etc.

#### Grammar
```
PARSER_BEGIN(Example)

/** Simple brace matcher. */
public class Example {

  /** Main entry point. */
  public static void main(String args[]) throws ParseException {
    Example parser = new Example(System.in);
    parser.Input();
  }

}

PARSER_END(Example)

/** Root production. */
void Input() :
{}
{
  MatchedBraces() ("\n"|"\r")* <EOF>
}

/** Brace matching production. */
void MatchedBraces() :
{}
{
  "{" [ MatchedBraces() ] "}"
}
```

#### Output
```
$ java Example
{{}}<return>
<control-d>

$ java Example
{x<return>
Lexical error at line 1, column 2.  Encountered: "x"
TokenMgrError: Lexical error at line 1, column 2.  Encountered: "x" (120), after : ""
        at ExampleTokenManager.getNextToken(ExampleTokenManager.java:146)
        at Example.getToken(Example.java:140)
        at Example.MatchedBraces(Example.java:51)
        at Example.Input(Example.java:10)
        at Example.main(Example.java:6)

$ java Example
{}}<return>
ParseException: Encountered "}" at line 1, column 3.
Was expecting one of:
    <EOF>
    "\n" ...
    "\r" ...
        at Example.generateParseException(Example.java:184)
        at Example.jj_consume_token(Example.java:126)
        at Example.Input(Example.java:32)
        at Example.main(Example.java:6)
```


## <a name="getting-started"></a>Getting Started

Follow the steps here to get started with JavaCC.

This guide will walk you through locally building the project, running an existing example, and setup to start developing and testing your own JavaCC application.

### <a name="building-from-source"></a>Building JavaCC from source

Prerequisites for building JavaCC:

* Git
* Ant (we require version 1.5.3 or above - you can get ant from [http://ant.apache.org](http://ant.apache.org))
* Maven
* Java 8 (Java 9 and 10 are not yet supported)

```
$ git clone https://github.com/javacc/javacc.git
$ cd javacc
$ ant
```

This will build the `javacc.jar` file in the `target/` directory


### <a name="developing"></a>Developing JavaCC

<!---
The JavaCC committers use IntelliJ IDEA to develop the JavaCC codebase.
-->
Minimal requirements for an IDE are:
* Support for Java
* Support for Maven with Java


#### IntelliJ IDEA

The IntelliJ IDE supports Maven out of the box and offers a plugin for JavaCC development.

* IntelliJ download: [https://www.jetbrains.com/idea/](https://www.jetbrains.com/idea/)
* IntelliJ JavaCC Plugin: [https://plugins.jetbrains.com/plugin/11431-javacc/](https://plugins.jetbrains.com/plugin/11431-javacc/)

<!---
Check out our [Setting up IntelliJ](https://ci.apache.org/projects/flink/flink-docs-master/flinkDev/ide_setup.html#intellij-idea) guide for details.
-->

#### Eclipse IDE

<!---
**NOTE:** From our experience, this setup does not work with Flink
due to deficiencies of the old Eclipse version bundled with Scala IDE 3.0.3 or
due to version incompatibilities with the bundled Scala version in Scala IDE 4.4.1.

**We recommend to use IntelliJ instead (see above)**
-->
* Eclipse download: [https://www.eclipse.org/ide/](https://www.eclipse.org/ide/)
* Eclipse JavaCC Plugin: [https://marketplace.eclipse.org/content/javacc-eclipse-plug](https://marketplace.eclipse.org/content/javacc-eclipse-plug)

### <a name="download"></a>Download

You can download JavaCC distributions from the release pages on [GitHub](https://github.com/javacc/javacc/releases).

To install JavaCC, navigate to the download directory and type:

```
$ unzip javacc-5.0.zip
or
$ tar xvf javacc-5.0.tar.gz
```

Once you have completed installation add the `bin/` directory in the JavaCC installation to your `PATH`. The JavaCC, JJTree, and JJDoc invocation scripts/executables reside in this directory.

### <a name="binary-distribution"></a>Binary distribution

You can access binaries on [Maven Central](https://mvnrepository.com/artifact/net.java.dev.javacc/javacc).

The binary distribution contains the JavaCC, JJTree and JJDoc sources, launcher scripts, example grammars and documentation. It also contains a bootstrap version of JavaCC needed to build JavaCC.

On Unix-based systems, you need to make sure the files in the `bin/` directory of the distribution are in your path.

<!---
#### Maven

Add the following dependency to your `pom.xml` file.

```
<dependency>
    <groupId>net.java.dev.javacc</groupId>
    <artifactId>javacc</artifactId>
    <version>7.0.4</version>
</dependency>
```

#### Gradle

Add the following to your `build.gradle` file.

```
repositories {
    mavenLocal()
    maven {
        url = 'https://mvnrepository.com/artifact/net.java.dev.javacc/javacc'
    }
}

dependencies {
    compile group: 'net.java.dev.javacc', name: 'javacc', version: '7.0.4'
}
```
-->

## <a name="community"></a>Community

JavaCC is by far the most popular parser generator used with Java applications, having an estimated user base of over 1,000 users and more than 100,000 downloads to date.

It is maintained by the developer community which includes the original authors and [Chris Ainsley](https://github.com/ainslec), [Tim Pizney](https://github.com/timp) and [Francis Andre](https://github.com/zosrothko).

### <a name="support"></a>Support

Don’t hesitate to ask!

Contact the developers and community on the [Google user group](https://groups.google.com/forum/#!forum/javacc-users) or email us at [JavaCC Support](mailto:support@javacc.org) if you need any help.

[Open an issue](https://github.com/javacc/javacc/issues) if you found a bug in JavaCC.

### <a name="documentation"></a>Documentation

The documentation of JavaCC is located on the website [https://javacc.org](https://javacc.org) or in the `www/` directory of the source code on [Github](https://github.com/javacc/javacc).

### <a name="powered-by"></a>Powered by JavaCC

JavaCC is used in many commercial applications and open source projects. The following list highlights a few notable JavaCC projects that run interesting use cases in production, with links to the relevant grammar specifications.

User                                                 | Use Case                                                       | Grammar File(s)
:--------------------------------------------------- |:-------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------:
[Apache ActiveMQ](https://activemq.apache.org/)      | Parsing JMS selector statements                                | [SelectorParser.jj](https://github.com/apache/activemq/blob/master/activemq-client/src/main/grammar/SelectorParser.jj), [HyphenatedParser.jj](https://github.com/apache/activemq-artemis/blob/master/artemis-selector/src/main/javacc/HyphenatedParser.jj)
[Apache Avro](https://avro.apache.org/)              | Parsing higher-level languages into Avro Schema                | [idl.jj](https://github.com/apache/avro/blob/master/lang/java/compiler/src/main/javacc/org/apache/avro/compiler/idl/idl.jj)
[Apache Calcite](https://calcite.apache.org/)        | Parsing SQL statements                                         | [Parser.jj](https://github.com/apache/calcite/blob/master/core/src/main/codegen/templates/Parser.jj)
[Apache Camel](https://camel.apache.org/)            | Parsing stored SQL templates                                   | [sspt.jj](https://github.com/apache/camel/blob/master/components/camel-sql/src/main/java/org/apache/camel/component/sql/stored/template/grammar/sspt.jj)
[Apache Jena](https://jena.apache.org/)              | Parsing queries written in SPARQL, ARQ, SSE, Turtle and JSON   | [sparql_10](https://github.com/apache/jena/blob/master/jena-arq/Grammar/Final/sparql_10-final.jj), [sparql_11](https://github.com/apache/jena/blob/master/jena-arq/Grammar/Final/sparql_11-final.jj), [arq.jj](https://github.com/apache/jena/blob/master/jena-arq/Grammar/arq.jj), [sse.jj](https://github.com/apache/jena/blob/master/jena-arq/Grammar/sse/sse.jj), [turtle.jj](https://github.com/apache/jena/blob/master/jena-arq/Grammar/turtle.jj), [json.jj](https://github.com/apache/jena/blob/master/jena-arq/Grammar/JSON/json.jj)
[Apache Lucene](https://lucene.apache.org/)          | Parsing search queries                                         | [QueryParser.jj](https://github.com/apache/lucene-solr/blob/master/lucene/queryparser/src/java/org/apache/lucene/queryparser/classic/QueryParser.jj)
[Apache Tomcat](https://tomcat.apache.org/)          | Parsing Expression Language (EL) and JSON                      | [ELParser.jjt](https://github.com/apache/tomcat/blob/master/java/org/apache/el/parser/ELParser.jjt), [JSONParser.jj](https://github.com/apache/tomcat/blob/master/java/org/apache/tomcat/util/json/JSONParser.jj)
[Apache Zookeeper](https://zookeeper.apache.org/)    | Optimising serialisation/deserialisation of Hadoop I/O records | [rcc.jj](https://github.com/apache/zookeeper/blob/master/zookeeper-jute/src/main/java/org/apache/jute/compiler/generated/rcc.jj)
[Java Parser](https://javaparser.org/)               | Parsing Java language files                                    | [java.jj](https://github.com/javaparser/javaparser/blob/master/javaparser-core/src/main/javacc/java.jj)

<!---
## <a name="contributing"></a>Contributing

This is an active open-source project. We are always open to people who want to use the system or contribute to it.
Contact us if you are looking for implementation tasks that fit your skills.
This article describes [how to contribute to Apache Flink](https://flink.apache.org/contributing/how-to-contribute.html).

https://blog.scottlowe.org/2015/01/27/using-fork-branch-git-workflow/

-->

## <a name="license"></a>License

JavaCC is an open source project release under the BSD License 2.0. The JavaCC project was originally developed at Sun Microsystems Inc. by [Sreeni Viswanadha](https://github.com/kaikalur) and [Sriram Sankar](https://twitter.com/sankarsearch).

***

Back to [ToC](#toc)
