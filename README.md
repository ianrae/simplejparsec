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
If you look at FirstTest.java you will see a base interface Expression and two classes that implement it.  These are AST nodes and will
be the output of your parser.

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
````

### Low-level grammar
Next you define the low-level grammar of tokens that make up your language.  Below we define some reserved words such as "readonly" and parsers for
integers and identifiers.

```java
	//low-level grammar
	private static final Terminals OPERATORS =
			Terminals.operators("=", "readonly", "var");

	//for integers and identifiers we need a PARSER and a TOKENIZER for each
	//here are the parsers
	public static Parser<String> identSyntacticParser = Terminals.Identifier.PARSER;
	public static Parser<String> integerSyntacticParser = Terminals.IntegerLiteral.PARSER;
	
	//and here are the tokenizers, combined into our single tokenizer
	static final Parser<?> TOKENIZER = Parsers.or(
					OPERATORS.tokenizer(),
					Terminals.IntegerLiteral.TOKENIZER, 
					Terminals.Identifier.TOKENIZER, 
					Terminals.Identifier.TOKENIZER);

	static final Parser<Void> IGNORED = Parsers.or(
			Scanners.JAVA_LINE_COMMENT,
			Scanners.JAVA_BLOCK_COMMENT,
			Scanners.WHITESPACES).skipMany();	
	
	private static Parser<String> readonly() {
		return OPERATORS.token("readonly").retn("readonly");
	}
	private static Parser<String> var() {
		return OPERATORS.token("var").retn("var");
	}
	private static Parser<String> eq() {
		return OPERATORS.token("=").retn("=");
	}
```

### High-level grammar
Next we define the high-level grammar, that parses the tokens from the low-level grammar.

There are examples here of several ways to convert the parsed token into an Expression object. 

```java
//parse single value into a SingleExpression
	private static Parser<SingleExpression> singleExpression01() {
		return Parsers.or(readonly())
				.map(new org.codehaus.jparsec.functors.Map<String, SingleExpression>() {
					@Override
					public SingleExpression map(String arg0) {
						return new SingleExpression(arg0);
					}
				});
	}
	
	//parse two tokens but only take the output of last one in sequence()
	private static Parser<SingleExpression> singleExpression02() {
		return Parsers.sequence(readonly(), var()).map(SingleExpression::new);
	}
	
	//parse two tokens and use the output of all tokens in sequence()
	private static Parser<DoubleExpression> doubleExpression01() {
		return Parsers.sequence(readonly(), var(), (String s, String s2) -> new DoubleExpression(s,s2));
	}
	//same thing but use Java 8 functional interface
	private static Parser<DoubleExpression> doubleExpression02() {
		return Parsers.sequence(readonly(), var(), DoubleExpression::new);
	}
	
	//same thing but use jparsec Mapper
	private static Parser<DoubleExpression> doubleExpression03() {
		Parser<DoubleExpression> d = Mapper.curry(DoubleExpression.class).sequence(readonly(), var());
		return d;
	}
```

### Use the parser

Finally we can use the parser using Parser.from and our TOKENIZER and IGNORED parser.

```java
	@Test
	public void testSingle1() {
		SingleExpression exp = singleExpression01().from(TOKENIZER, IGNORED).parse("readonly");
		assertEquals("readonly", exp.s);
	}
```

### GrammarTest.java
The second unit test shows how an AST can be built up from lower level expressions.  The FullExpression class represents
the parse output for "var x = 5"



