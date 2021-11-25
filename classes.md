
# Writing Good Classes
As long as we're going to be writing in a language with classes, we should
know how to write good classes.
This might seem unnecessary, but believe me I've seen some bad classes.

### What is a Class?
At its simplest, a class is just another name for a data structure.
It should contain some fields which together make up the data the class is
meant to store.

Crucially, a class should store _all_ the data it's meant to contain.
If a class contains a reference to an object that it didn't explicitly create
or copy, then it doesn't _actually_ store that data, and so that shouldn't
be a field of the class.

This is an example of two well-used class:
```c#
class MyClass1 {
    private int field1;

    private MyClass2 field2;

    public MyClass1(string path) {
        field1 = 0;
        field2 = new MyClass2(path);
    }

    public string doSomething() {
        return field2.doSomething();
    }
}

class MyClass2 {
    private StreamReader field3;

    public MyClass2(string path) {
        field3 = File.OpenText(path);
    }

    public string doSomething() {
        return field3.ReadLine();
    }
}
```

Contrast that against this example of a poorly-used class:
```c#
class MyClass3 {
    private int field4;

    private StreamReader field5;

    private MyClass4 field6;

    public MyClass3(string path) {
        field4 = 0;
        field5 = File.OpenText(path);
        field6 = new MyClass4(field5);
    }

    public string doSomething() {
        return field6.doSomething();
    }
}

class MyClass4 {
    private StreamReader field7;

    public MyClass4(FileStream fs) {
        field7 = fs;
    }

    public string doSomething() {
        return field7.ReadLine();
    }
}
```

What's wrong with the second example?
Notice that `MyClass4` stores a _reference_ to `field5`.
The class isn't actually storing the _data_ of that field; that's done in
`MyClass3`.


### So, When is a Class?
We only have a good justification for a class when there is some data that
we want to store together, and that this structure will be the
_exclusive owner_ of.
If we store a reference to data managed elsewhere, then that ownership
isn't exclusive, and that data should not be a field of the class.

So what do we do when our class needs data that it doesn't own, like in
`MyClass4`?
The answer is elegantly simple: we provide the external data explicitly
only when it's needed.
The classes `MyClass3` and `MyClass4` _should_ be written
```c#
class MyClass3 {
    private int field4;

    private StreamReader field5;

    private MyClass4 field6;

    public MyClass3(string path) {
        field4 = 0;
        field5 = File.OpenText(path);
        field6 = new MyClass4();
    }

    public string doSomething() {
        return field6.doSomething();
    }
}

class MyClass4 {
    public string doSomething(StreamReader fs) {
        return fs.ReadLine();
    }
}
```

You might notice that `MyClass4` has no fields now.
That's exactly right; it's not a meaningful class anymore because it has no
data of its own.
We can completely remove the class if we want.

If we're really desperate to retain the function `MyClass4.doSomething()`,
we can rewrite it as a static function and treat `MyClass4` as its own mini
namespace:
```c#
class MyClass4 {
    public static string doSomething(StreamReader fs) {
        retunr fs.ReadLine();
    }
}
```


###
