# simplejparsec
A simple parsing example using JParsec

[JParsec](https://github.com/jparsec/jparsec) is a good parsing library, but it can be challenging to learn.

This project has two sample junit tests that show you how to get a basic statement parser going.

### Two Modes of Operation

JParsec has two modes of operating.  In the [Overview] (https://github.com/jparsec/jparsec/wiki/Overview) page these are referred to as
on lexical analysis vs. syntactical analysis.

The first mode is for very simple parsing.  Most of the time you will use the second mode ("syntactic analysis").  One thing to be aware
of is that the same API classes are used for both modes, but some methods only work in one mode.

### AST Nodes
If you look at FirstTest.java

```java
	private interface Expression {
	}

	public static class SingleExpression implements Expression {
		public String s;
		
		public SingleExpression(String s) {
			this.s = s;
		}
	}
	public static class DoubleExpression implements Expression {
		public String s;
		public String s2;
		
		public DoubleExpression(String s, String s2) {
			this.s = s;
			this.s2 = s2;
		}
	}
````d
