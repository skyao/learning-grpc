字段
=========

> 注：内容翻译自官网文档 [Packages](https://developers.google.com/protocol-buffers/docs/reference/java-generated#fields)

除了在前一节描述的方法之外，protocol buffer编译器为每个字段生成访问器方法集合

In addition to the methods described in the previous section, the protocol buffer compiler generates a set of accessor methods for each field defined within the message in the .proto file. The methods that read the field value are defined both in the message class and its corresponding builder; the methods that modify the value are only defined in the builder only.

Note that method names always use camel-case naming, even if the field name in the .proto file uses lower-case with underscores (as it should). The case-conversion works as follows:

For each underscore in the name, the underscore is removed, and the following letter is capitalized.
If the name will have a prefix attached (e.g. "get"), the first letter is capitalized. Otherwise, it is lower-cased.
Thus, the field foo_bar_baz becomes fooBarBaz. If prefixed with get, it would be getFooBarBaz.

As well as accessor methods, the compiler generates an integer constant for each field containing its field number. The constant name is the field name converted to upper-case followed by _FIELD_NUMBER. For example, given the field optional int32 foo_bar = 5;, the compiler will generate the constant public static final int FOO_BAR_FIELD_NUMBER = 5;.






