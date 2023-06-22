# Notes for myself

## ToDos
- [ ] Create very minimal tokenizer with `nom` for string literal
- [ ] Create license file and upload to GH
- [ ] Check license for Graphviz and PlantUML


## Notes
* Based on the ChatGPT a minimal graphviz port would "only" need to create the AST from the PlantUML DSL and then translated to a graphviz dotfile. That's super interesting I always wanted to learn about tokenization, parsing, etc
* Mh, apparently rendering is done by PlantUML not Graphviz. So, apparently, the dot files don't  describe the diagram exhaustively.

## Research
### Asking ChatGPT
> How does plantuml render graphs?

PlantUML is a tool that allows you to create diagrams and charts using a simple text-based syntax. When you write PlantUML code, it is translated into an intermediate format called the Abstract Syntax Tree (AST). The AST is then processed to generate a final image of the diagram.

The image rendering process in PlantUML involves several steps. First, the AST is converted into a graph layout using a layout algorithm, such as Graphviz or GEF (Generic Eclipse Framework) layout. The layout algorithm determines the positions of the nodes and edges in the diagram.

Next, the layout information is passed to a renderer that generates an image of the diagram. PlantUML supports several image formats, including PNG, SVG, and EPS. The renderer creates the final image by drawing the nodes and edges using the specified styles and colors.

PlantUML also supports various options for customizing the output, such as setting the size of the image, changing the background color, and adding annotations. Overall, PlantUML provides a convenient way to create diagrams and charts from simple text-based syntax, making it a popular tool among developers and technical writers.

> is the layout algorithm implemented in plantuml or is graphviz used?

PlantUML can use both Graphviz and its own internal layout engine to perform the layout of diagrams.

When using Graphviz, PlantUML generates a Graphviz DOT file from the input PlantUML code, and then uses Graphviz to perform the layout. This requires Graphviz to be installed on the system where PlantUML is running.

On the other hand, if Graphviz is not available or if you prefer to use an alternative layout engine, PlantUML provides its own layout engine, called "Spray". This is a simple layout engine that can perform basic layout operations, such as positioning nodes and arranging edges. Spray is written in Java and is part of the PlantUML distribution.

In summary, PlantUML supports both Graphviz and its own layout engine for performing the layout of diagrams, and users can choose the one that works best for their needs.

> What does it take to create the AST from the PlantUML code?

Creating the Abstract Syntax Tree (AST) from the PlantUML code involves parsing the text and converting it into a tree structure that represents the diagram. The following steps are typically involved:

1. Tokenization: The first step is to split the input PlantUML code into a sequence of tokens, where each token represents a single unit of meaning, such as a keyword, identifier, or symbol. The tokenization process is usually done using regular expressions or a parser generator tool.

2. Parsing: The next step is to use a parser to analyze the sequence of tokens and generate a tree structure that represents the diagram. The parser examines the sequence of tokens according to a set of grammar rules that define the syntax of the PlantUML language. The parser can be implemented using a parser generator tool, such as ANTLR or JavaCC.

3. AST Construction: Once the parser has identified the individual components of the PlantUML code, it can use this information to construct the AST. The AST is a tree structure that represents the diagram, where each node in the tree corresponds to a diagram element, such as a shape or a connection. Each node in the AST contains information about the element it represents, such as its position, size, and style.

4. Validation: After constructing the AST, the code may go through a validation process to ensure that it conforms to certain rules and constraints. For example, the validator may check that there are no circular dependencies between elements, or that the same element is not defined twice.

Overall, creating the AST from the PlantUML code involves a series of steps that parse, analyze, and transform the input text into a tree structure that represents the diagram.

> Does something similar exist but written in Rust?

**The following is completely hallucinated by ChatGPT! I didn't find any rust library called drumstick**

Yes, there are several diagramming tools written in Rust that use a text-based syntax similar to PlantUML and Mermaid.

One popular Rust-based diagramming tool is "drumstick", which is inspired by PlantUML and provides a similar syntax for creating diagrams. Drumstick supports a variety of diagram types, including flowcharts, UML diagrams, and network diagrams, and can generate diagrams in various output formats, including PNG, SVG, and PDF. Drumstick is actively developed and maintained, and is available under the Apache 2.0 license.

Another Rust-based diagramming tool is "asciiflow", which is a web-based tool for creating ASCII art-style diagrams. Asciiflow provides a simple and intuitive interface for creating diagrams using keyboard shortcuts and drag-and-drop operations, and can export diagrams as plain text or PNG files. Asciiflow is also open source and can be used under the MIT license.

Overall, Rust-based diagramming tools are still relatively new and less widely adopted than PlantUML and Mermaid, but they offer an interesting alternative for developers who prefer to work with Rust and want to create diagrams using a text-based syntax.

> Can you point me to the source code file where tokenization happens in plantuml

Sure! The tokenization process in PlantUML happens in the net/sourceforge/plantuml/Token.java file, which is part of the PlantUML core library.

In this file, you'll find the Token class, which represents a single token in the PlantUML syntax. The Token class has various fields and methods that are used to store and manipulate the token data, such as the token type, value, and position in the input stream.

The tokenization process itself is implemented in the net.sourceforge.plantuml.core.Diagram.java file, which is also part of the PlantUML core library. In this file, you'll find the Diagram class, which represents a single PlantUML diagram. The Diagram class has a scan method, which takes the input string and tokenizes it into a sequence of Token objects.

The scan method uses a combination of regular expressions and custom parsing logic to tokenize the input string. It scans the input string character by character, looking for specific patterns that match the syntax of PlantUML elements, such as comments, keywords, identifiers, and literals.

Overall, the tokenization process in PlantUML is a complex and critical component of the tool, and it involves multiple files and classes in the PlantUML core library. If you're interested in learning more about how the tokenization process works in PlantUML, I would recommend exploring the source code further.

> Can you create a minimal tokenizer in Rust using the `nom` create?

```rust
use nom::{
    branch::alt,
    bytes::complete::{escaped, is_not, tag},
    character::complete::{alpha1, alphanumeric1, char, multispace0, multispace1},
    combinator::{map, opt, recognize},
    sequence::{delimited, pair},
    IResult,
};

#[derive(Debug, PartialEq)]
enum Token<'a> {
    StartUml,
    EndUml,
    StartClass,
    EndClass,
    StartMethod,
    EndMethod,
    Identifier(&'a str),
    Literal(&'a str),
}

fn parse_identifier(input: &str) -> IResult<&str, &str> {
    recognize(pair(alpha1, opt(alphanumeric1)))(input)
}

fn parse_literal(input: &str) -> IResult<&str, &str> {
    escaped(is_not("\\\""), '\\', char('\"'))(input)
}

fn parse_start_uml(input: &str) -> IResult<&str, Token> {
    map(tag("@"), |_| Token::StartUml)(input)
}

fn parse_end_uml(input: &str) -> IResult<&str, Token> {
    map(tag("end"), |_| Token::EndUml)(input)
}

fn parse_start_class(input: &str) -> IResult<&str, Token> {
    map(tag("{"), |_| Token::StartClass)(input)
}

fn parse_end_class(input: &str) -> IResult<&str, Token> {
    map(tag("}"), |_| Token::EndClass)(input)
}

fn parse_start_method(input: &str) -> IResult<&str, Token> {
    map(tag("("), |_| Token::StartMethod)(input)
}

fn parse_end_method(input: &str) -> IResult<&str, Token> {
    map(tag(")"), |_| Token::EndMethod)(input)
}

fn parse_identifier_token(input: &str) -> IResult<&str, Token> {
    map(parse_identifier, Token::Identifier)(input)
}

fn parse_literal_token(input: &str) -> IResult<&str, Token> {
    map(delimited(char('"'), parse_literal, char('"')), Token::Literal)(input)
}

fn parse_whitespace(input: &str) -> IResult<&str, ()> {
    map(multispace1, |_| ())(input)
}

fn parse_token(input: &str) -> IResult<&str, Token> {
    alt((
        parse_start_uml,
        parse_end_uml,
        parse_start_class,
        parse_end_class,
        parse_start_method,
        parse_end_method,
        parse_identifier_token,
        parse_literal_token,
    ))(input)
}

fn parse_tokens(input: &str) -> IResult<&str, Vec<Token>> {
    let (input, _) = multispace0(input)?;
    let (input, tokens) = nom::multi::many0(pair(parse_whitespace, parse_token))(input)?;
    let (input, _) = multispace0(input)?;
    Ok((input, tokens.into_iter().map(|(_, t)| t).collect()))
}

fn tokenize(input: &str) -> Result<Vec<Token>, nom::Err<nom::error::Error<&str>>> {
    let (_, tokens) = parse_tokens(input)?;
    Ok(tokens)
}
```
