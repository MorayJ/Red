# [Red ORM](https://github.com/FCO/Red) Architechture

[Red](https://github.com/FCO/Red) is an ORM for [Raku](https://raku.org) that tries to mimic [Raku](https://raku.org)'s [`Seq`](https://docs.raku.org/type/Seq)'s API but for querying databases.
[Red](https://github.com/FCO/Red) implements a custom Metamodel based on [Metamodel::ClassHOW](https://docs.raku.org/type/Metamodel::ClassHOW).
You use its new [`model`](https://github.com/FCO/Red/blob/master/lib/Red.pm6#L41) keyword to describe your table and its relations as a Raku class.

The [Red Metamodel](https://github.com/FCO/Red/blob/master/lib/MetamodelX/Red/Model.pm6) exports a meta-method called [`all`](https://github.com/FCO/Red/blob/master/lib/MetamodelX/Red/Model.pm6#L283)
or [`rs`](https://github.com/FCO/Red/blob/master/lib/MetamodelX/Red/Model.pm6#L280), which returns an instance of a class called [`Red::ResultSeq`](https://github.com/FCO/Red/blob/master/lib/Red/ResultSeq.pm6).
[`Red::ResultSeq`](https://github.com/FCO/Red/blob/master/lib/Red/ResultSeq.pm6) is essentially a 
specialization of Raku’s [`Seq`](https://docs.raku.org/type/Seq) type for data in the database. So

```raku
MyModel.^all
```

represents all rows in the `MyModel` table, and 

```raku
MyModel.^all.grep: *.col1 > 3
```

represents all rows in the `MyModel` table where the value of `col1` is higher than 3. The `grep` method (and most of the other 
[`Red::ResultSeq`](https://github.com/FCO/Red/blob/master/lib/Red/ResultSeq.pm6) methods) returns a new `ResultSeq`.

A `ResultSeq` stores a [`Red::AST`](https://github.com/FCO/Red/tree/master/lib/Red/AST) tree and each method that returns a new [`Red::ResultSeq`](https://github.com/FCO/Red/blob/master/lib/Red/ResultSeq.pm6) creates a new
[`Red::AST`](https://github.com/FCO/Red/blob/master/lib/Red/AST) tree adding some more [`Red::AST`](https://github.com/FCO/Red/tree/master/lib/Red/AST) nodes.

For example, the [`.map`](https://github.com/FCO/Red/blob/master/lib/Red/ResultSeq.pm6#L308) method runs the [`Callable`](https://docs.raku.org/type/Callable) code passing the model's object type as the only parameter for that
[`Callable`](https://docs.raku.org/type/Callable). It has some logic to convert what the [`Callable`](https://docs.raku.org/type/Callable) does to [`Red::AST`](https://github.com/FCO/Red/tree/master/lib/Red/AST) and creates a new
[`Red::ResultSeq`](https://github.com/FCO/Red/blob/master/lib/Red/ResultSeq.pm6) combining the [`Red::AST`](https://github.com/FCO/Red/tree/master/lib/Red/AST) from the previous one with the one generated by the
[`.map`](https://github.com/FCO/Red/blob/master/lib/Red/ResultSeq.pm6#L308). (It's planned to change as soon as it's possible to create custom compile passes with [RakuAST](https://www.youtube.com/watch?v=91uaaSyrKm0).)

Once a [`Red::ResultSeq`](https://github.com/FCO/Red/blob/master/lib/Red/ResultSeq.pm6) is iterated, it uses the current [`Red::Driver`](https://github.com/FCO/Red/tree/master/lib/Red/Driver) to do two things:

1. translate the [`Red::AST`](https://github.com/FCO/Red/tree/master/lib/Red/AST) tree to SQL according to the database variant (each [`Red::AST`](https://github.com/FCO/Red/tree/master/lib/Red/AST) has its own translation)
1. connect to the database and run the query

Once the database responds, it gets the response and returns a [`Seq`](https://docs.raku.org/type/Seq) with an object for each row.