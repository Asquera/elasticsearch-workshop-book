# Elasticsearch Book

Elasticsearch training material &amp; examples as a mdbook.


## Build

The book is using [mdbook](https://github.com/rust-lang/mdBook) to generate a static site. To build the book locally install [Rust](https://www.rust-lang.org/tools/install) first, then use [cargo](https://doc.rust-lang.org/cargo/) to install the [mdbook](https://github.com/rust-lang/mdBook) command line tool.

To install mdbook with cargo run:

```bash
cargo install mdbook
```

To render the book locally use the `mdbook` CLI executable:

```bash
mdbook build
```

it reads the [SUMMARY.md](./src/SUMMARY.md) file to understand the structure of the book. It takes all markdown files and outputs static htlm pages. To view the book open the generated [index](./book/index.html) page.
