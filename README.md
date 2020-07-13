# Sana
[![Rust](https://github.com/suhr/sana/workflows/Rust/badge.svg)](https://github.com/suhr/sana/actions?query=branch%3Amaster)

Sana is a lexer generator for Rust. It provides a easy way to create a lexical analyzer for a language.

Unlike other lexer generator, Sana supports extended regular expressions, which allow you to express your intent more clearly. For example, you can write `"[[:punct:]]+" & !".*--.*"` to represent a sequence of punctuation characters which does not contain `--`.

At compile time, Sana:

- Constructs an deterministic state automata from the token definitions
- Generates IR from the automata
- (TODO) Compiles IR into Rust code

For an overview of the Sana architecture, see [DESIGN.md](./DESIGN.md).

# Example

```rust
use sana::{Sana, Spanned};

#[derive(Debug, Clone, Copy, PartialEq, Sana)]
enum Token {
    #[regex("[a-zA-Z_][a-zA-Z0-9_]*")]
    Ident,
    #[regex("[0-9]+")]
    Integer,

    #[token("let", priority = 1)]
    Let,
    #[token("=")]
    Equals,
    #[regex(";")]
    Semicolon,

    #[regex("[ \t\r\n]")]
    Whitespace,
    #[error]
    Error,
}

fn main() {
    let mut lexer = Token::lexer("let answer = 42;");

    assert_eq!(
        lexer.next(),
        Some(Spanned { value: Token::Let, start: 0, end: 3 })
    );
    assert_eq!(lexer.next(),
        Some(Spanned { value: Token::Whitespace, start: 3, end: 4 })
    );
    assert_eq!(lexer.next(),
        Some(Spanned { value: Token::Ident, start: 4, end: 10 })
    );
    assert_eq!(lexer.next(),
        Some(Spanned { value: Token::Whitespace, start: 10, end: 11 })
    );
    assert_eq!(
        lexer.next(),
        Some(Spanned { value: Token::Equals, start: 11, end: 12 })
    );
    assert_eq!(
        lexer.next(),
        Some(Spanned { value: Token::Whitespace, start: 12, end: 13 })
    );
    assert_eq!(
        lexer.next(),
        Some(Spanned { value: Token::Integer, start: 13, end: 15 })
    );
    assert_eq!(
        lexer.next(),
        Some(Spanned { value: Token::Semicolon, start: 15, end: 16 })
    );

    // No tokens left
    assert_eq!(
        lexer.next(),
        None
    );
}
```

You can find more examples in the [`examples`](./examples) directory.
