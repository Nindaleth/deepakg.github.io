---
layout: post
title:  "Nim in Action: Book errata for Nim v1.0.0"
date:   2019-09-28 16:37:00 +0200
categories: nim
---

The Nim programming language recently hit the 1.0.0 milestone and I decided to pick it up. Since Manning's [Nim in Action](https://www.manning.com/books/nim-in-action) came back in 2017, it naturally targets an older version of Nim. Fortunately, there haven't been big user-visible changes to Nim over these two years so it's still possible to use this book to learn. That said, as is inevitable with a new language that's still under development, some things *have* changed.

I am compiling an errata of sorts as I work my way through the book. All page numbers correspond to the PDF/print version of the book. While a lot of compiler error messages have changed, I'll be highlighting only the important ones.

## Chapter 1

### Listing 1.7

```
import sequtils, future, strutils
     let list = @["Dominik Picheta", "Andreas Rumpf", "Desmond Hume"]
     list.map(
       (x: string) -> (string, string) => (x.split[0], x.split[1])
     ).echo
```

gives a warning during compilation:

```
Warning: Use the new 'sugar' module instead; future is deprecated [Deprecated]
```

Simply changing `future` to `sugar` fixes it without requiring any other change to the code in this listing.


## Chapter 2

### Section 2.2.2

#### Page 32

> I added the echo at the top of fillString’s body, in order to show you that it’s executed at compile time. Try compiling the example using Aporia or in a terminal by executing nim c file.nim.

Be aware that Aporia IDE is now obsolete. The [project's page mentions VSCode](https://github.com/nim-lang/Aporia) as an alternative. I personally use emacs with nim-mode and find it more than adequate.

There are plenty of references to Aporia throughout the book. I won't highlight them in this post any further.

#### Page 33

> For more information about Nim’s conventions, take a look at the “Style Guide for Nim Code” on GitHub: https://github.com/nim-lang/Nim/wiki/Style-Guide-for-Nim-Code.

The Style Guide page has now moved to:

[https://nim-lang.org/docs/nep1.html](https://nim-lang.org/docs/nep1.html)

### Section 2.2.3

#### Page 38

```
doAssert getUserCity("Damien", "Lundi") == "Tokyo"
doAssert getUserCity(2) == "New York
```

It might be confusing to see that the book uses `assert` for previous code snippets but for this one, suddenly switches to `doAssert`.

The difference between `assert` and `doAssert` is that `assert` is ignored by the nim compiler when using the `-d:release` or `--assertions:off` command line switches, whereas `doAssert` isn't affected by them. [Source](https://nim-lang.org/docs/assertions.html). This is also explained later in the book in Chapter 3 (page 76).

### Section 2.3.1

#### Page 40

> Compilation will fail with “Error: index out of bounds.”

It's still a compile-time out of bounds error but the error message is now worded differently:

```
Error: index 5000 not in 0 .. 2
```

### Section 2.3.2

#### Page 41

> “You should see that your program crashes with the following output:

```
Traceback (most recent call last)
segfault.nim(2)          segfault
SIGSEGV: Illegal storage access. (Attempt to read from nil?)
```

This still segfaults but the error message is now different and definitely more helpful:

```
segfault.nim(2) segfault
/usr/local/Cellar/nim/1.0.0/nim/lib/system/fatal.nim(39) sysFatal
Error: unhandled exception: index out of bounds, the container is empty [IndexError]
```

#### Page 42

```
assert list[0] == nil
```

This construct now seems to be deprecated for strings. I get this compile-time error:

```
seq.nim(3, 16) Error: 'nil' is now invalid for 'string'; compile with --nilseqs:on for a migration period; usage of '==' is a user-defined error
```

Changing this to...

```
assert list[0] == ""
```

...works. So strings are now automatically initialised to "" rather than nil.

## Chapter 3

This chapter covers a *lot* of ground and by the end of it you have a working group chat server and a client that connects to it. I only ran into the following trivial issues.

### Listing 3.16

```
import asyncdispatch, asyncfile

var file = openAsync("/etc/passwd")
let dataFut = file.readAll()
dataFut.callback =
  proc (future: Future[string]) =
    echo(future.read())

asyncdispatch.runForever()
```

On macOS 10.13.6 (High Sierra), this ended in a runtime error after the callback finishes displaying the file's content:

```
/usr/local/Cellar/nim/1.0.0/nim/lib/pure/asyncdispatch.nim(1874) runForever
/usr/local/Cellar/nim/1.0.0/nim/lib/pure/asyncdispatch.nim(1569) poll
/usr/local/Cellar/nim/1.0.0/nim/lib/pure/asyncdispatch.nim(1286) runOnce
Error: unhandled exception: No handles or timers registered in dispatcher. [ValueError]
Error: execution of an external program failed: '/Users/deepakg/proj/async_file
```

I still haven't figured out if the author deliberately intended to demonstrate something or if something changed in a version of Nim after the book came out. That said, the next listing (3.17) that uses the `{.async.}` pragma with the `await` keyword works just fine.

### Listing 3.17

```
import asyncdispatch, asyncfile

proc readFiles() {.async.} =
  var file = openAsync("/home/profile/test.txt", fmReadWrite)
  let data = await file.readAll()
  echo(data)
  await file.write("Hello!\n")
  file.close()

waitFor readFiles()
```

It might come as a surprise that the `echo(data)` call never prints anything, even when the file already exists and contains text. Additionally, the file contents are reset to `Hello!` after running the code.

This is because `fmReadWrite` file mode clears the existing data during the `openAsync` call. If you want to preserve and display the file contents, use `fmReadWriteExisting` file mode instead.

### Section 3.5.3

#### Page 92

> On UNIX-like operating systems such as Linux and Mac OS, the telnet application should be available by default.

On macOS 10.13 (High Sierra) that's not the case. But it's easily remedied by using [brew](https://brew.sh) to install it.

## Chapter 4

This chapter is a whirlwind tour of Nim's standard library. I did not find any issues. All listings compiled and ran with Nim 1.0.0 without any problems.

## Chapter 5

I did not find any issues. This chapter is about Nim's package manager: Nimble. I was able to follow along and create a toy package and use it. I ignored section 5.7 (publishing a package to github), but it should just work.

## Chapter 6

While chapter 3 gave a taste of concurrency in Nim and also touched upon parallelism (`spawn`), this chapter offers a deeper look into parallelism in Nim through its threading capabilities.

### Section 6.2.3

#### Page 159

> The ways exceptions behave in separate threads may be surprising. When a thread crashes with an unhandled exception, the application will crash with it. It doesn’t mat- ter whether you read the value of the FlowVar or not.

>> FUTURE VERSIONS This behavior will change in a future version of Nim, so that exceptions aren’t raised unless you read the value of the FlowVar.

This hasn't changed as of Nim 1.0.0

### Listing 6.20

#### Page 172
```
  responses.add(spawn parseChunk(buffer[0 .. <chunkLen])) oldBufferLen = readSize - chunkLen
  buffer[0 .. <oldBufferLen] = buffer[readSize - oldBufferLen .. ^1]

var mostPopular = newStats()
for resp in responses:
  let statistic = ^resp
  if statistic.countViews > mostPopular.countViews:
  mostPopular = statistic
```

The thing that puzzled me a little about this listing was the lack of an explicit `sync()` call (like one in Listing 6.7 on page 157). We spawn multiple threads in a loop to run `parseChunk` but we don't wait for them to finish. This is because the `for resp in responses` loop a couple of lines below, implicitly waits for each thread to finish when it retrieves the result using `^resp`.

### Listing 6.21

#### Page 173
```
import threadpool

var counter = 0

proc increment(x: int) =
  for i in 0 .. <x:
    var value = counter
    value.inc
    counter = value

spawn increment(10_000)
spawn increment(10_000)
sync()
echo(counter)
```

Now gives a compile-time deprecation warning.

```
shared_memory_race_condition.nim(6, 17) Warning: < is deprecated [Deprecated]
```

This is easily fixed by changing `0 .. <x` to `0..<x` or `0 ..< x` in the listing (no space between .. and <). Listing 6.23 and 6.26 have the exact same issue as they build upon listing 6.21.

## Chapter 7

### Section 7.2

#### Page 188

> You’ll also need to add bin = @["tweeter"] to the Tweeter.nimble file to let Nimble know which files in your package need to be compiled.

When you run nimble (v.0.11.0) that ships with Nim v.1.0.0, it'll ask you to choose your package type:

```
    Prompt: Package type?
        ... Library - provides functionality for other packages.
        ... Binary  - produces an executable for the end-user.
        ... Hybrid  - combination of library and binary
        ... For more information see https://goo.gl/cm2RX5
     Select Cycle with 'Tab', 'Enter' when done
   Choices:  library
           > binary <
             hybrid
```

The default selection for package type is `library` but if you select `binary`, the nimble file you get will already have a bin = @["Tweeter"] entry. Also, nimble now creates src/Tweeter.nim for you with some example code in it which is different from lowercase tweeter.nim that you create by hand in the book.