# xml

[![CI](https://github.com/alya-lang/xml/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/xml/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/xml?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fxml%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fxml%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

A fast, zero-dependency XML 1.0 parser and serializer for the Alya Programming Language.

---

## 🌟 Features

- ⚡ **High Performance**: Parses ~19,000 documents/sec, >3M lookups/sec and >400K serializations/sec.
- 📐 **Practical XML 1.0 Coverage**:
  - **Elements**: Nested elements, mixed text content, and self-closing tags (`<tls/>`).
  - **Attributes**: Single/double-quoted values, entity decoding, and whitespace normalization.
  - **Prolog & Misc**: `<?xml version="..." encoding="..."?>` declarations, UTF-8 BOM tolerance, comments (`<!-- -->`), processing instructions (`<?...?>`), and `DOCTYPE` blocks (including `[...]` internal subsets).
  - **CDATA**: `<![CDATA[ ... ]]>` sections kept verbatim.
  - **References**: Predefined entities (`&amp;`, `&lt;`, `&gt;`, `&quot;`, `&apos;`) plus decimal (`&#65;`) and hexadecimal (`&#x41;`) character references; unknown entities pass through literally.
  - **Path Navigation**: Dot-separated element queries (`catalog.book.title`), recursive `find_all`, and trimmed text helpers.
  - **Attribute Order**: Document order preserved on round-trip (`attr_order` tracking, duplicate collapsing).
  - **Strict Mode**: `check_wellformed` diagnostics with byte offsets plus `parse_strict` / `parse_doc_strict` gates.
  - **Namespaces**: Prefix/default `xmlns` resolution with scoped `find_ns` lookup (elements only).
- 🛠️ **Builder API**: Programmatically construct documents (`make_element`, `set_attr`, `add_child`, `set_text`) and serialize compact or pretty-printed.
- 🧪 **100% Verified**: 171 assertions passing, verified on Windows (Linux/macOS via CI).

---

## 📁 Project Architecture

```
xml/
├── alya.toml               # Package manifest
├── src/
│   ├── lib.alya            # Public API facade (parse, accessors, XmlDoc methods, file IO)
│   ├── parser.alya         # XML 1.0 parser (misc skipping, entities, elements, documents)
│   ├── serializer.alya     # Escaping, compact/pretty rendering & builder helpers
│   └── types.alya          # XmlNode/XmlDoc structs, XmlNodeType enum, name classifiers
├── examples/
│   └── demo.alya           # Comprehensive usage example
├── tests/
│   ├── test_basic.alya     # Full 14-part automated test suite (123 assertions)
│   ├── test_unicode.alya   # Unicode reference regression suite (12 assertions)
│   └── test_advanced.alya  # Attribute order, strict validation & namespace suite (36 assertions)
└── benches/
    └── bench_basic.alya    # Micro-benchmarks for parsing, lookup, and serialization
```

---

## 📦 Installation

Add `xml` to your `alya.toml`:

```toml
[dependencies]
xml = { git = "https://github.com/alya-lang/xml", branch = "main" }
```

Or install it directly via the Alya CLI:

```bash
alya add xml --git https://github.com/alya-lang/xml --branch main
alya install
```

### Package Features

| Feature | Default | Description |
|:---|:---:|:---|
| `io` | ✅ | File loading/saving (`load_file`, `load_doc`, `dump_file`). Without it only in-memory parse/serialize remain. |

```bash
# Full build (default)
alya install
alya test

# Slim build without file I/O
alya install --no-default-features
alya test --no-default-features
```

---

## 🚀 Quick Start

### 1. Parsing and Querying XML

```alya
import "xml"

function main()
    let text = "
<inventory region=\"eu-central\">
    <service id=\"api-1\" tls=\"true\">
        <name>gateway</name>
        <port>8080</port>
    </service>
</inventory>
"
    let root = xml::parse(text)
    say xml::get_attr(root, "region")      # "eu-central"
    say xml::text_at(root, "service.name") # "gateway"
    say xml::text_at(root, "service.port") # "8080"
end

main()
```

### 2. Declaration Metadata and Serialization

```alya
import "xml"

function main()
    let doc = xml::parse_doc("<?xml version=\"1.0\" encoding=\"UTF-8\"?><a>t</a>")
    say doc.version   # "1.0"
    say doc.encoding  # "UTF-8"
    say doc.to_string(1)
end

main()
```

### 3. Building Documents Programmatically

```alya
import "xml"

function main()
    let root = xml::make_element("server")
    xml::set_attr(root, "host", "0.0.0.0")
    xml::add_child(root, xml::set_text(xml::make_element("name"), "edge"))
    xml::add_child(root, xml::make_element("tls"))
    say xml::stringify(root, 1)
end

main()
```

---

## 📖 API Reference

| Symbol | Description |
|---|---|
| `parse(text)` | Parses XML into the root `XmlNode` (null when no element is present). |
| `parse_doc(text)` | Parses XML into an `XmlDoc` with `version`/`encoding` metadata. |
| `load_file(path)` | Reads a file and parses it into a root node. |
| `load_doc(path)` | Reads a file into an `XmlDoc`. |
| `is_valid(text)` | `1` when the text holds a well-formed element, `0` otherwise. |
| `check_wellformed(text)` | `""` when structurally valid, else a reason with byte offset. |
| `parse_strict(text)` | Root node, or null unless the input validates. |
| `parse_doc_strict(text)` | `XmlDoc`, or an empty document unless the input validates. |
| `get_attr(node, key, default="")` | Attribute value or default (null-safe). |
| `has_attr(node, key)` | `1` when the attribute exists, `0` otherwise. |
| `children(node)` | Direct child element array (empty when none). |
| `child_count(node)` | Direct child element count. |
| `children_by_name(node, name)` | Direct children filtered by tag name. |
| `first_child(node, name)` | First matching child, or null. |
| `text_of(node, default="")` | Raw text content or default. |
| `find_path(node, path)` | Dot-separated descent (`"a.b.c"`), first match per level. |
| `find_all(node, name)` | Recursive descendant collection in document order. |
| `local_name(qname)` | Local part after the first `:` (or the whole name). |
| `ns_prefix(qname)` | Prefix before the first `:` (or `""`). |
| `ns_scope(node)` | Prefix-to-URI map from the node's `xmlns` declarations. |
| `find_ns(node, uri, local)` | Recursive element search with inherited scope. |
| `text_at(node, path, default="")` | Trimmed text at a path, or default. |
| `XmlDoc.find(path)` | `find_path` from the document root. |
| `XmlDoc.find_all(name)` | `find_all` from the document root. |
| `XmlDoc.text_at(path, default="")` | `text_at` from the document root. |
| `XmlDoc.to_string(pretty=0)` | Serializes the document with declaration. |
| `stringify(node, pretty=0)` | Serializes a node tree (compact or indented). |
| `stringify_doc(doc, pretty=0)` | Serializes a document with declaration. |
| `dump_file(path, xml_str)` | Writes an XML string to disk. |
| `make_element(name, attrs, children, text)` | Element factory with null-safe containers. |
| `make_doc(root, version, encoding)` | Document factory with declaration defaults. |
| `set_attr(node, key, val)` | Sets an attribute, returns the node for chaining. |
| `add_child(node, child)` | Appends a child, returns the parent for chaining. |
| `set_text(node, text)` | Copy with replaced text content. |
| `escape_text(s)` | Escapes `&`, `<`, `>` for text content. |
| `unescape(s)` | Decodes entities and character references in a plain string. |
| `escape_attr(s)` | Escapes `&`, `<`, `"`, and whitespace controls for attributes. |
| `is_valid_name(name)` | `1` when the string is a valid XML name. |
| `declaration(version, encoding)` | Builds an `<?xml ... ?>` declaration string. |
| `XmlNodeType` | Node kind enum (`Element = 1`, `Text = 2`). |
| `XmlNode` | Element struct (`name`, `attrs`, `children`, `text`). |
| `XmlDoc` | Document struct (`root`, `version`, `encoding`). |

---

## 🧪 Running Tests & Benchmarks

Run the automated test suite using `alya test`:

```bash
alya test
```

Generate static API documentation:

```bash
alya doc . -o docs --markdown
```

Run the benchmark suite:

```bash
alya run benches/bench_basic.alya
```

Run the example demo:

```bash
alya run examples/demo.alya
```

Check code formatting:

```bash
alya fmt . --check
```

Run static code linter:

```bash
alya lint . --check
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository and clone it locally
2. Install dependencies:
   ```bash
   alya install
   ```
3. Create your feature branch (`git checkout -b feature/my-feature`)
4. Verify tests and formatting before opening a PR:
   ```bash
   alya test
   ```
5. Commit your changes (`git commit -m "feat: add feature"`) and open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
