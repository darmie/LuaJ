![alt text](https://github.com/Tidal-Loop/LuaJ/blob/master/icon.gif "LuaJ") 
#LuaJ
* * *

#Getting Started with LuaJ.

James Roseborough, Ian Farmer, Version 3.0

<small>Copyright © 2009-2014 Luaj.org. (Also Nathan, Marcus, and Pratik for the crazy hack job with the chainsaw that makes it make it through proguard on Android without warnings.) Freely available under the terms of the [Luaj license](http://sourceforge.net/dbimage.php?id=196142).</small>

* * *

[introduction](#1) · [examples](#2) · [concepts](#3) · [libraries](#4) · [luaj api](#5) · [parser](#6) · [building](#7) · [downloads](#8) · [release notes](#9)

# 1 - <a name="1">Introduction</a>

## Goals of Luaj

Luaj is a lua interpreter based on the 5.2.x version of lua with the following goals in mind:

*   Java-centric implementation of lua vm built to leverage standard Java features.
*   Lightweight, high performance execution of lua.
*   Multi-platform to be able to run on JME, JSE, or JEE environments.
*   Complete set of libraries and tools for integration into real-world projects.
*   Dependable due to sufficient unit testing of vm and library features.

## Luaj version and Lua Versions

### Luaj 3.0.x

Support for lua 5.2.x features:

*   _ENV environments model.
*   yield from pcall or metatags.
*   Bitwise operator library.

It also includes miscellaneous improvements over luaj 2.0.x:

*   Better thread safety.
*   More compatible table behavior.
*   Better coroutine-related garbage collection.
*   Maven integration.
*   Better debug reporting when using closures.
*   Line numbers in parse syntax tree.

### Luaj 2.0.x

Support for lua 5.1.x features, plus:

*   Support for compiling lua source code into Java source code.
*   Support for compiling lua bytecode directly into Java bytecode.
*   Stackless vm design centered around dynamically typed objects.
*   Good alignment with C API (see [names.csv](names.csv) for details)
*   Implementation of weak keys and values, and all metatags.

### Luaj 1.0.x

Support for most lua 5.1.x features.

## Performance

Good performance is a major goal of luaj. The following table provides measured execution times on a subset of benchmarks from [the computer language benchmarks game](http://shootout.alioth.debian.org/) in comparison with the standard C distribution.

| Project  | Version | Mode             | Benchmark execution time (sec) |               |             |          | Language | Sample Command                                                  |
|----------|---------|------------------|--------------------------------|---------------|-------------|----------|----------|-----------------------------------------------------------------|
|          |         |                  | binarytrees (15)               | fannkuch (10) | nbody (1e6) | nsieve 9 |          |                                                                 |
| luaj     | 3.0     | -b (luajc)       | 2.980                          | 5.073         | 16.794      | 11.274   | Java     | java -cp luaj-jse-3.0.1.jar;bcel-5.2.jar lua -b fannkuch.lua 10 |
|          |         | -n (interpreted) | 12.838                         | 23.290        | 36.894      | 15.163   |          | java -cp luaj-jse-3.0.1.jar lua -n fannkuch.lua 10              |
| lua      | 5.1.4   |                  | 17.637                         | 16.044        | 15.201      | 5.477    | C        | lua fannkuch.lua 10                                             |
| jill     | 1.0.1   |                  | 44.512                         | 54.630        | 72.172      | 20.779   | Java     |                                                                 |
| kahlua   | 1.0     | jse              | 22.963                         | 63.277        | 68.223      | 21.529   | Java     |                                                                 |
| mochalua | 1.0     |                  | 50.457                         | 70.368        | 82.868      | 41.262   | Java     |                                                                 |


Luaj in interpreted mode performs well for the benchmarks, and even better when the lua-to-java-bytecode (luajc) compiler is used, and actually executes _faster_ than C-based lua in some cases. It is also faster than Java-lua implementations Jill, Kahlua, and Mochalua for all benchmarks tested.

# 2 - <a name="2">Examples</a>

## Run a lua script in Java SE

From the main distribution directory line type:

<pre>	java -cp lib/luaj-jse-3.0.jar lua examples/lua/hello.lua
</pre>

You should see the following output:

<pre>	hello, world
</pre>

To see how luaj can be used to acccess most Java API's including swing, try:

<pre>	java -cp lib/luaj-jse-3.0.jar lua examples/lua/swingapp.lua
</pre>

Links to sources:	[examples/lua/hello.lua](examples/lua/hello.lua) [examples/lua/swingapp.lua](examples/lua/swingapp.lua)

## Compile lua source to lua bytecode

From the main distribution directory line type:

<pre>	java -cp lib/luaj-jse-3.0.jar luac examples/lua/hello.lua
	java -cp lib/luaj-jse-3.0.jar lua luac.out
</pre>

The compiled output "luac.out" is lua bytecode and should run and produce the same result.

## Compile lua source or bytecode to java bytecode

Luaj can compile lua sources or binaries directly to java bytecode if the bcel library is on the class path. From the main distribution directory line type:

<pre>	ant bcel-lib
	java -cp "lib/luaj-jse-3.0.jar;lib/bcel-5.2.jar" luajc -s examples/lua -d . hello.lua
	java -cp "lib/luaj-jse-3.0.jar;." lua -l hello
</pre>

The output _hello.class_ is Java bytecode, should run and produce the same result. There is no runtime dependency on the bcel library, but the compiled classes must be in the class path at runtime, unless runtime jit-compiling via luajc and bcel are desired (see later sections).

Lua scripts can also be run directly in this mode without precompiling using the _lua_ command with the **_-b_** option and providing the _bcel_ library in the class path:

<pre>	java -cp "lib/luaj-jse-3.0.jar;lib/bcel-5.2.jar" lua -b examples/lua/hello.lua
</pre>

## Run a script in a Java Application

A simple hello, world example in luaj is:

<pre>	import org.luaj.vm2.*;
	import org.luaj.vm2.lib.jse.*;

	Globals globals = JsePlatform.standardGlobals();
	LuaValue chunk = globals.load("print 'hello, world'");
	chunk.call();

</pre>

Loading from a file is done via Globals.loadFile():

<pre>	LuaValue chunk = globals.loadfile("examples/lua/hello.lua");
</pre>

Chunks can also be loaded from a `Reader` as text source

<pre>	chunk = globals.load(new StringReader("print 'hello, world'"), "main.lua");
</pre>

or an InputStream to be loaded as text source "t", or binary lua file "b":

<pre>	chunk = globals.load(new FileInputSStream("examples/lua/hello.lua"), "main.lua", "bt"));
</pre>

A simple example may be found in [examples/jse/SampleJseMain.java](examples/jse/SampleJseMain.java)


You must include the library **lib/luaj-jse-3.0.jar** in your class path.

## Run a script in a MIDlet

For MIDlets the _JmePlatform_ is used instead:

<pre>	import org.luaj.vm2.*;
	import org.luaj.vm2.lib.jme.*;

	Globals globals = JmePlatform.standardGlobals();
	LuaValue chunk = globals.loadfile("examples/lua/hello.lua");
	chunk.call();
</pre>

The file must be a resource within within the midlet jar for the loader to find it. Any files included via _require()_ must also be part of the midlet resources.

A simple example may be found in [examples/jme/SampleMIDlet.java](examples/jme/SampleMIDlet.java)


You must include the library **lib/luaj-jme-3.0.jar** in your midlet jar.

An ant script to build and run the midlet is in [build-midlet.xml](build-midlet.xml)


You must install the wireless toolkit and define _WTK_HOME_ for this script to work.

## Run a script using JSR-223 Dynamic Scripting

The standard use of JSR-223 scripting engines may be used:

<pre>	ScriptEngineManager mgr = new ScriptEngineManager();
	ScriptEngine e = mgr.getEngineByName("luaj");
	e.put("x", 25);
	e.eval("y = math.sqrt(x)");
	System.out.println( "y="+e.get("y") );
</pre>

You can also look up the engine by language "lua" or mimetypes "text/lua" or "application/lua".

All standard aspects of script engines including compiled statements are supported.

You must include the library **lib/luaj-jse-3.0.jar** in your class path.

A working example may be found in [examples/jse/ScriptEngineSample.java](examples/jse/ScriptEngineSample.java)


To compile and run it using Java 1.6 or higher:

<pre>	javac -cp lib/luaj-jse-3.0.jar examples/jse/ScriptEngineSample.java
	java -cp "lib/luaj-jse-3.0.jar;examples/jse" ScriptEngineSample
</pre>

## Excluding the lua bytecode compiler

By default, the compiler is included whenever _standardGlobals()_ or _debugGlobals()_ are called. Without a compiler, files can still be executed, but they must be compiled elsewhere beforehand. The "luac" utility is provided in the jse jar for this purpose, or a standard lua compiler can be used.

To exclude the lua-to-lua-bytecode compiler, do not call _standardGlobals()_ or _debugGlobals()_ but instead initialize globals with including only those libraries that are needed and omitting the line:

<pre>	org.luaj.vm2.compiler.LuaC.install(globals);
</pre>

## Including the LuaJC lua-bytecode-to-Java-bytecode compiler

To compile from lua to Java bytecode for all lua loaded at runtime, install the LuaJC compiler into a _globals_ object use:

<pre>	org.luaj.vm2.jse.luajc.LuaJC.install(globals);
</pre>

This will compile all lua bytecode into Java bytecode, regardless of if they are loaded as lua source or lua binary files.

The requires _bcel_ to be on the class path, and the ClassLoader of JSE or CDC.

# 3 - <a name="3">Concepts</a>

## Globals

The old notion of platform has been replaced with creation of globals. The [Globals](http://luaj.sourceforge.net/api/3.0/org/luaj/vm2/Globals.html) class holds global state needed for executing closures as well as providing convenience functions for compiling and loading scripts.

## Platform

To simplify construction of Globals, and encapsulate differences needed to support the diverse family of Java runtimes, luaj uses a Platform notion. Typically, a platform is used to construct a Globals, which is then provided as a global environment for client scripts.

### JsePlatform

The [JsePlatform](http://luaj.sourceforge.net/api/3.0/org/luaj/vm2/lib/jse/JsePlatform.html) class can be used as a factory for globals in a typical Java SE application. All standard libraries are included, as well as the luajava library. The default search path is the current directory, and the math operations include all those supported by Java SE.

#### Android

Android applications should use the JsePlatform, and can include the [Luajava](#luajava) library to simplify access to underlying Android APIs. A specialized Globals.finder should be provided to find scripts and data for loading. See [examples/android/src/android/LuajView](https://github.com/Tidal-Loop/LuaJ/blob/master/examples/android/src/android/LuajView.java) for an example that loads from the "res" Android project directory. The ant build script is [examples/android/build.xml](examples/android/build.xml).

#### Applet

Applets in browsers should use the JsePlatform. The permissions model in applets is highly restrictive, so a specialization of the [Luajava](#luajava) library must be used that uses default class loading. This is illustrated in the sample Applet [examples/jse/SampleApplet.java](examples/jse/SampleApplet.java), which can be built using [build-applet.xml](build-applet.xml).

### JmePlatform

The [JmePlatform](http://luaj.sourceforge.net/api/3.0/org/luaj/vm2/lib/jme/JmePlatform.html) class can be used to set up the basic environment for a Java ME application. The default search path is limited to the jar resources, and the math operations are limited to those supported by Java ME. All libraries are included except luajava, and the os, io, and math libraries are limited to those functions that can be supported on that platform.

#### MIDlet

MIDlets require the JmePlatform. The JME platform has several limitations which carry over to luaj. In particular Globals.finder is overridden to load as resources, so scripts should be colocated with class files in the MIDlet jar file. [Luajava](#luajava) cannot be used. Camples code is in [examples/jme/SampleMIDlet.java](examples/jme/SampleMIDlet.java), which can be built using [build-midlet.xml](build-midlet.xml).

## Thread Safety

Luaj 3.0 can be run in multiple threads, with the following restrictions:

*   Each thread created by client code must be given its own, distinct Globals instance
*   Each thread must not be allowed to access Globals from other threads
*   Metatables for Number, String, Thread, Function, Boolean, and and Nil are shared and therefore should not be mutated once lua code is running in any thread.

For an example of loading allocating per-thread Globals and invoking scripts in multiple threads see [examples/jse/SampleMultiThreaded.java](examples/jse/SampleMultiThreaded.java)

As an alternative, the JSR-223 scripting interface can be used, and should always provide a separate Globals instance per script engine instance by using a ThreadLocal internally.

# 4 - <a name="4">Libraries</a>

## Standard Libraries

Libraries are coded to closely match the behavior specified in See [standard lua documentation](http://www.lua.org/manual/5.1/) for details on the library API's

The following libraries are loaded by both _JsePlatform.standardGlobals()_ and _JmePlatform.standardGlobals()_:

<pre>	base
	bit32
	coroutine
	io
	math
	os
	package
	string
	table
</pre>

The _JsePlatform.standardGlobals()_ globals also include:

<pre>	luajava
</pre>

The _JsePlatform.debugGlobals()_ and _JsePlatform.debugGlobals()_ functions produce globals that include:

<pre>	debug
</pre>

### I/O Library

The implementation of the _io_ library differs by platform owing to platform limitations.

The _JmePlatform.standardGlobals()_ instantiated the io library _io_ in

<pre>	src/jme/org/luaj/vm2/lib/jme/JmeIoLib.java
</pre>

The _JsePlatform.standardGlobals()_ includes support for random access and is in

<pre>	src/jse/org/luaj/vm2/lib/jse/JseIoLib.java
</pre>

### OS Library

The implementation of the _os_ library also differs per platform.

The basic _os_ library implementation us used by _JmePlatform_ and is in:

<pre>	src/core/org/luaj/lib/OsLib.java
</pre>

A richer version for use by _JsePlatform_ is :

<pre>	src/jse/org/luaj/vm2/lib/jse/JseOsLib.java
</pre>

Time is a represented as number of seconds since the epoch, and locales are not implemented.

### Coroutine Library

The _coroutine_ library is implemented using one JavaThread per coroutine. This allows _coroutine.yield()_ can be called from anywhere, as with the yield-from-anywhere patch in C-based lua.

Luaj uses WeakReferences and the OrphanedThread error to ensure that coroutines that are no longer referenced are properly garbage collected. For thread safety, OrphanedThread should not be caught by Java code. See [LuaThread](http://luaj.sourceforge.net/api/3.0/org/luaj/vm2/LuaThread.html) and [OrphanedThread](http://luaj.sourceforge.net/api/3.0/org/luaj/vm2/OrphanedThread.html) javadoc for details.

### Debug Library

The _debug_ library is not included by default by _JmePlatform.standardGlobals()_ or _JsePlatform.standardGlobsls()_ . The functions _JmePlatform.debugGlobals()_ and _JsePlatform.debugGlobsls()_ create globals that contain the debug library in addition to the other standard libraries. To install dynamically from lua use java-class-based require::

<pre>	require 'org.luaj.vm2.lib.DebugLib'
</pre>

The _lua_ command line utility includes the _debug_ library by default.

### <a name="luajava">The Luajava Library</a>

The _JsePlatform.standardGlobals()_ includes the _luajava_ library, which simplifies binding to Java classes and methods. It is patterned after the original [luajava project](http://www.keplerproject.org/luajava/).

The following lua script will open a swing frame on Java SE:

<pre>	jframe = luajava.bindClass( "javax.swing.JFrame" )
	frame = luajava.newInstance( "javax.swing.JFrame", "Texts" );
	frame:setDefaultCloseOperation(jframe.EXIT_ON_CLOSE)
	frame:setSize(300,400)
	frame:setVisible(true)
</pre>

See a longer sample in _examples/lua/swingapp.lua_ for details, including a simple animation loop, rendering graphics, mouse and key handling, and image loading. Or try running it using:

<pre>	java -cp lib/luaj-jse-3.0.jar lua examples/lua/swingapp.lua
</pre>

The Java ME platform does not include this library, and it cannot be made to work because of the lack of a reflection API in Java ME.

The _lua_ connand line tool includes _luajava_.

# 5 - <a name="5">LuaJ API</a>

## API Javadoc

The javadoc for the main classes in the LuaJ API are on line at [docs/api](docs/api/index.html)


You can also build a local version from sources using

<pre>	 ant doc
</pre>

## LuaValue and Varargs

All lua value manipulation is now organized around [LuaValue](docs/api/org/luaj/vm2/LuaValue.html) which exposes the majority of interfaces used for lua computation. [org.luaj.vm2.LuaValue](docs/api/org/luaj/vm2/LuaValue.html)


### Common Functions

_LuaValue_ exposes functions for each of the operations in LuaJ. Some commonly used functions and constants include:

<pre>	call();               // invoke the function with no arguments
	call(LuaValue arg1);  // call the function with 1 argument
	invoke(Varargs arg);  // call the function with variable arguments, variable return values
	get(int index);       // get a table entry using an integer key
	get(LuaValue key);    // get a table entry using an arbitrary key, may be a LuaInteger
	rawget(int index);    // raw get without metatable calls
	valueOf(int i);       // return LuaValue corresponding to an integer
	valueOf(String s);    // return LuaValue corresponding to a String
	toint();              // return value as a Java int
	tojstring();          // return value as a Java String
	isnil();              // is the value nil
	NIL;                  // the value nil
	NONE;                 // a Varargs instance with no values	 
</pre>

## Varargs

The interface [Varargs](http://luaj.sourceforge.net/api/3.0/org/luaj/vm2/Varargs.html) provides an abstraction for both a variable argument list and multiple return values. For convenience, _LuaValue_ implements _Varargs_ so a single value can be supplied anywhere variable arguments are expected. [org.luaj.vm2.Varargs](http://luaj.sourceforge.net/api/3.0/org/luaj/vm2/Varargs.html)

### Common Functions

_Varargs_ exposes functions for accessing elements, and coercing them to specific types:

<pre>	narg();                 // return number of arguments
	arg1();                 // return the first argument
	arg(int n);             // return the nth argument
	isnil(int n);           // true if the nth argument is nil
	checktable(int n);      // return table or throw error
	optlong(int n,long d);  // return n if a long, d if no argument, or error if not a long
</pre>

See the [Varargs](docs/api/org/luaj/vm2/Varargs.html) API for a complete list.

## LibFunction

The simplest way to implement a function is to choose a base class based on the number of arguments to the function. LuaJ provides 5 base classes for this purpose, depending if the function has 0, 1, 2, 3 or variable arguments, and if it provide multiple return values. [org.luaj.vm2.lib.ZeroArgFunction](docs/api/org/luaj/vm2/lib/ZeroArgFunction.html), [org.luaj.vm2.lib.OneArgFunction](docs/api/org/luaj/vm2/lib/OneArgFunction.html), [org.luaj.vm2.lib.TwoArgFunction](docs/api/org/luaj/vm2/lib/TwoArgFunction.html), [org.luaj.vm2.lib.ThreeArgFunction](docs/api/org/luaj/vm2/lib/ThreeArgFunction.html), [org.luaj.vm2.lib.VarArgFunction](docs/api/org/luaj/vm2/lib/VarArgFunction.html)


Each of these functions has an abstract method that must be implemented, and argument fixup is done automatically by the classes as each Java function is invoked.

An example of a function with no arguments but a useful return value might be:

<pre>	pubic class hostname extends ZeroArgFunction {
		public LuaValue call() {
			return valueOf(java.net.InetAddress.getLocalHost().getHostName());
		}
	}
</pre>

The value _env_ is the environment of the function, and is normally supplied by the instantiating object whenever default loading is used.

Calling this function from lua could be done by:

<pre>
	local hostname = require( 'hostname' )
</pre>

while calling this function from Java would look like:

<pre>
	new hostname().call();
</pre>

Note that in both the lua and Java case, extra arguments will be ignored, and the function will be called. Also, no virtual machine instance is necessary to call the function. To allow for arguments, or return multiple values, extend one of the other base classes.

## Libraries of Java Functions

When require() is called, it will first attempt to load the module as a Java class that implements LuaFunction. To succeed, the following requirements must be met:

*   The class must be on the class path with name, _modname_.
*   The class must have a public default constructor.
*   The class must inherit from LuaFunction.

If luaj can find a class that meets these critera, it will instantiate it, cast it to _LuaFunction_ then call() the instance with two arguments: the _modname_ used in the call to require(), and the environment for that function. The Java may use these values however it wishes. A typical case is to create named functions in the environment that can be called from lua.

A complete example of Java code for a simple toy library is in [examples/jse/hyperbolic.java](examples/jse/hyperbolic.java)

<pre>import org.luaj.vm2.LuaValue;
import org.luaj.vm2.lib.*;

public class hyperbolic extends TwoArgFunction {

	public hyperbolic() {}

	public LuaValue call(LuaValue modname, LuaValue env) {
		LuaValue library = tableOf();
		library.set( "sinh", new sinh() );
		library.set( "cosh", new cosh() );
		env.set( "hyperbolic", library );
		return library;
	}

	static class sinh extends OneArgFunction {
		public LuaValue call(LuaValue x) {
			return LuaValue.valueOf(Math.sinh(x.checkdouble()));
		}
	}

	static class cosh extends OneArgFunction {
		public LuaValue call(LuaValue x) {
			return LuaValue.valueOf(Math.cosh(x.checkdouble()));
		}
	}
}
</pre>

In this case the call to require invokes the library itself to initialize it. The library implementation puts entries into a table, and stores this table in the environment.

The lua script used to load and test it is in [examples/lua/hyperbolicapp.lua](examples/lua/hyperbolicapp.lua)

<pre>	require 'hyperbolic'

	print('hyperbolic', hyperbolic)
	print('hyperbolic.sinh', hyperbolic.sinh)
	print('hyperbolic.cosh', hyperbolic.cosh)

	print('sinh(0.5)', hyperbolic.sinh(0.5))
	print('cosh(0.5)', hyperbolic.cosh(0.5))
</pre>

For this example to work the code in _hyperbolic.java_ must be compiled and put on the class path.

## Closures

Closures still exist in this framework, but are optional, and are only used to implement lua bytecode execution, and is generally not directly manipulated by the user of luaj.

See the [org.luaj.vm2.LuaClosure](docs/api/org/luaj/vm2/org/luaj/vm2/LuaClosure.html) javadoc for details on using that class directly.

# 6 - <a name="6">Parser</a>

## Javacc Grammar

A Javacc grammar was developed to simplify the creation of Java-based parsers for the lua language. The grammar is specified for [javacc version 5.0](https://javacc.dev.java.net/) because that tool generates standalone parsers that do not require a separate runtime.

A plain undecorated grammer that can be used for validation is available in [grammar/Lua51.jj](grammar/Lua51.jj) while a grammar that generates a typed parse tree is in [grammar/LuaParser.jj](grammar/LuaParser.jj)

## Creating a Parse Tree from Lua Source

The default lu compiler does a single-pass compile of lua source to lua bytecode, so no explicit parse tree is produced.

To simplify the creation of abstract syntax trees from lua sources, the LuaParser class is generated as part of the JME build. To use it, provide an input stream, and invoke the root generator, which will return a Chunk if the file is valid, or throw a ParseException if there is a syntax error.

For example, to parse a file and print all variable names, use code like:

<pre>	try {
		String file = "main.lua";
		LuaParser parser = new LuaParser(new FileInputStream(file));
		Chunk chunk = parser.Chunk();
		chunk.accept( new Visitor() {
			public void visit(Exp.NameExp exp) {
				System.out.println("Name in use: "+exp.name.name
					+" line "+exp.beginLine
					+" col "+exp.beginColumn);
			}
		} );
	} catch ( ParseException e ) {
		System.out.println("parse failed: " + e.getMessage() + "\n"
			+ "Token Image: '" + e.currentToken.image + "'\n"
			+ "Location: " + e.currentToken.beginLine + ":" + e.currentToken.beginColumn
			         + "-" + e.currentToken.endLine + "," + e.currentToken.endColumn);
	}
</pre>

An example that prints locations of all function definitions in a file may be found in [examples/jse/SampleParser.java](examples/jse/SampleParser.java)


See the [org.luaj.vm2.ast package](docs/api/org/luaj/vm2/ast/package-summary.html) javadoc for the API relating to the syntax tree that is produced.

# 7 - <a name="7">Building and Testing</a>

## <a name="maven">Maven integration</a>

The main jar files are now deployed in the maven central repository. To use them in your maven-based project, list them as a dependency:

For JSE projects, add this dependency for the luaj-jse jar:

<pre>   <dependency>
      <groupId>org.luaj</groupId>
      <artifactId>luaj-jse</artifactId>
      <version>3.0</version>
   </dependency>
</pre>

while for JME projects, use the luaj-jme jar:

<pre>   <dependency>
      <groupId>org.luaj</groupId>
      <artifactId>luaj-jme</artifactId>
      <version>3.0</version>
   </dependency>
</pre>

An example skelton maven pom file for a skeleton project is in [examples/maven/pom.xml](examples/maven/pom.xml)

## Building the jars

An ant file is included in the root directory which builds the libraries by default.

Other targets exist for creating distribution file an measuring code coverage of unit tests.

## Unit tests

The main luaj JUnit tests are organized into a JUnit 3 suite:

<pre>	test/junit/org/luaj/vm2/AllTests.lua
</pre>

Unit test scripts can be found in these locations

<pre>	test/lua/*.lua
	test/lua/errors/*.lua
	test/lua/perf/*.lua
	test/lua/luaj3.0-tests.zip
</pre>

## Code coverage

A build script for running unit tests and producing code coverage statistics is in [build-coverage.xml](build-coverage.xml)


It relies on the cobertura code coverage library.

# 8 - <a name="8">Downloads</a>

## Downloads and Project Pages

Downloads for all version available on SourceForge or LuaForge. Sources are hosted on SourceForge and available via sourceforge.net [SourceForge Luaj Project Page](http://luaj.sourceforge.net/), [SourceForge Luaj Download Area](http://sourceforge.net/project/platformdownload.php?group_id=197627)


The jar files may also be downloaded from the maven central repository, see [Maven Integration](#maven).

Files are no longer hosted at LuaForge.

# 9 - <a name="9">Release Notes</a>

## Main Changes by Version

|

|   **2.0** |

*   Initial release of 2.0 version

 |
|   **2.0.1** |

*   Improve correctness of singleton construction related to static initialization
*   Fix nan-related error in constant folding logic that was failing on some JVMs
*   JSR-223 fixes: add META-INF/services entry in jse jar, improve bindings implementation

 |
|   **2.0.2** |

*   JSR-223 bindings change: non Java-primitives will now be passed as LuaValue
*   JSR-223 enhancement: allow both ".lua" and "lua" as extensions in getScriptEngine()
*   JSR-223 fix: use system class loader to support using luaj as JRE extension
*   Improve selection logic when binding to overloaded functions using luajava
*   Enhance javadoc, put it [in distribution](docs/api/index.html) and [on line](http://luaj.sourceforge.net/api/3.0/index.html)
*   Major refactor of luajava type coercion logic, improve method selection.
*   Add lib/luaj-sources-2.0.2.jar for easier integration into an IDE such as Netbeans

 |
|   **2.0.3** |

*   Improve coroutine state logic including let unreferenced coroutines be garbage collected
*   Fix lua command vararg values passed into main script to match what is in global arg table
*   Add arithmetic metatag processing when left hand side is a number and right hand side has metatable
*   Fix load(func) when mutiple string fragments are supplied by calls to func
*   Allow access to public members of private inner classes where possible
*   Turn on error reporting in LuaParser so line numbers ar available in ParseException
*   Improve compatibility of table.remove()
*   Disallow base library setfenv() calls on Java functions

 |
|   **3.0-alpha1** |

*   Convert internal and external API's to match lua 5.2.x environment changes
*   Add bit32 library
*   Add explicit Globals object to manage global state, especially to imrpove thread safety
*   Drop support for lua source to java surce (lua2java) in favor of direct java bytecode output (luajc)
*   Remove compatibility functions like table.getn(), table.maxn(), table.foreach(), and math.log10()
*   Add ability to create runnable jar file from lua script with sample build file build-app.xml

 |
|   **3.0-alpha2** |

*   Supply environment as second argument to LibFunction when loading via require()

 |
|   **3.0-alpha3** |

*   Fix bug 3597515 memory leak due to string caching by simplifying caching logic.
*   Fix bug 3565008 so that short substrings are backed by short arrays.
*   Fix bug 3495802 to return correct offset of substrings from string.find().
*   Add artifacts to Maven central repository.
*   Limit pluggable scripting to use compatible bindings and contexts, implement redirection.

 |
|   **3.0-beta1** |

*   Fix bug that didn't read package.path from environment.
*   Fix pluggable scripting engine lookup, simplify implementation, and add unit tests.
*   Coerce script engine eval() return values to Java.
*   Fix Lua to Java coercion directly on Java classes.
*   Fix Globals.load() to call the library with an empty modname and the globals as the environment.
*   Fix hash codes of double.
*   Fix bug in luajava overload resolution.
*   Fix luastring bug where parsing did not check for overflow.
*   Fix luastring bug where circular dependency randomly caused NullPointerException.
*   Major refactor of table implementation.
*   Improved behavior of next() (fixes issue #7).
*   Existing tables can now be made weak (fixes issue #16).
*   More compatible allocation of table entries in array vs. hash (fixes issue #8).

 |
|   **3.0-beta2** |

*   Fix os.time() to return a number of seconds instead of milliseconds.
*   Implement formatting with os.date(), and table argument for os.time().
*   LuaValue.checkfunction() now returns LuaFunction.
*   Refactor APIs related to compiling and loading scripts to provide methods on Globals.
*   Add API to compile from Readers as well as InputStreams.
*   Add optional -c encoding flag to lua, luac, and luajc tools to control source encoding.
*   Let errors thrown in debug hooks bubble up to the running coroutine.
*   Make error message handler function in xpcall per-thread instead of per-globals.
*   Establish "org.luaj.debug" and "org.luaj.luajc" system properties to configure scripting engine.

 |
|   **3.0** |

*   Fix maven sample code.
*   Add sample code for Android Application that uses luaj.
*   Add sample code for Applet that uses luaj.
*   Fix balanced match for empty string (fixes issue #23).
*   Pass user-supplied ScriptContext to script engine evaluation (fixes issue #21).
*   Autoflush and encode written bytes in script contexts (fixes issue #20).
*   Rename Globals.FINDER to Globals.finder.
*   Fix bug in Globals.UTF8Stream affecting loading from Readers.
*   Add buffered input for compiling and loading of scripts.
*   In CoerceJavaToLua.coerse(), coerce byte[] to LuaString (fixes issue #31).
*   In CoerceJavaToLua.coerse(), coerce LuaValue to same value (fixes issue #29).
*   Fix line number reporting in debug stack traces (fixes issue #30).

 |

 |

## Known Issues

### Limitations

*   debug code may not be completely removed by some obfuscators
*   tail calls are not tracked in debug information
*   mixing different versions of luaj in the same java vm is not supported
*   values associated with weak keys may linger longer than expected
*   behavior of luaj when a SecurityManager is used has not been fully characterized
*   negative zero is treated as identical to integer value zero throughout luaj
*   lua compiled into java bytecode using luajc cannot use string.dump() or xpcall()
*   number formatting with string.format() is not supported

### File Character Encoding

Source files can be considered encoded in UTF-8 or ISO-8859-1 and results should be as expected, with literal string contianing quoted characters compiling to the same byte sequences as the input. For a non ASCII-compatible encoding such as EBSDIC, however, there are restrictions:

*   supplying a Reader to Globals.load() is preferred over InputStream variants
*   using FileReader or InputStreamReader to get the default OS encoding should work in most cases
*   string literals with quoted characters may not produce the expected values in generated code
*   command-line tools lua, luac, and luajc will require _-c Cp037_ to specify the encoding

These restrictions are mainly a side effect of how the language is defined as allowing byte literals within literal strings in source files. Code that is generated on the fly within lua and compiled with lua's _load()_ function should work as expected, since these strings will never be represented with the host's native character encoding.
