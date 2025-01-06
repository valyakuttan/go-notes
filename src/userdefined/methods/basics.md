# Methods

**Go** does not have classes. However, you can define methods on types.

A method is a function with a special receiver argument.

we can define a method for any type except pointers and interfaces.

The receiver appears in its own argument list between the func
keyword and the method name.

In this example, the Abs method has a receiver of type Vertex named v.

Note: A method is just a function with a receiver argument.

You can only declare a method with a receiver whose type is defined in the
same package as the method. You cannot declare a method with a receiver
whose type is defined in another package.
