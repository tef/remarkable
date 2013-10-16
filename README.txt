"Why I hate your document markup format" 

--------------------------------------------------------------------------------

Abstract

I have to write worksheets. Friends have to write technical documentation. I 
would like a pleasant way to write and manage documents that can be translated 
into web ready, ebook, and print formats. Without crying.

And yes, I have seen that XKCD. So let's look at the options:

Markdown:

	- Considerable amounts of ASCII art. 
	- Everyone does it differently.
	- It's a HTML superset. Generating non HTML is painful.
	- Whitespace sensitive to the point of tears. Indents must be 4 spaces 
	  in some variants. Line continuations are 2 trailing spaces.

LaTeX:

	- Turing Complete.
	- Weirdo error messages. You can't do that in horizontal mode. Overful
	  hbox.
	- Hard to recall edge cases of syntax.
	- Won't do EPUB, HTML, pleasantly. Tied to print output.
	- Possible to extend, but only in weirdo arcane macro language.
	- It's not a document format bob, it's a turing complete 
	  macro system. Did you just tell me to go fuck myself?
	- Unicode.

HTML:

	- Clunky at times.
	- Won't generate references, indices, tables of contents.
	- Manual Demarcation of Paragraphs

Restructured Text:

	- Some ASCII art (header titles, etc)
	- Extensible blocks but hard to add inline markup, without using
	  substitutions.
	- Table syntax involves shunting.
	- Zero tests or specs or grammars.

Those problems in depth:

	ASCII Art involves lots of pissing around to make the text render
	properly. For ex: In RST I cannot change a header name without
	adding extra - underneath. It also means that words_like_this can
	be mangled, as well as accidental formatting.

	A lack of a spec or grammar means each variant handles text
	differently. Similarly for a lack of tests. Every implementation
	of markdown handles things differently.

	Without directives or extensible blocks, commands vary by
	implementation. In Markdown, tables, or code blocks take different
	forms.

	Semantic whitespace can be useful, but exact numbers of whitespace, 
	or trailing whitespace are a pain in the butt.

What I desire:

	- A Text File renders as a text file. I.e linebreaks work, 
	  unless escaped
	- Inline and Blocks can be extended and parameterized
	- All commands can be expressed without ascii art, although
	  some will have shorthand.
	- Well defined grammar

Proposed Grammar: 

Rule 0: \ is the command character. \\ is a literal \ 

Rule 1: Line breaks are line breaks, unless preceded with a \

Rule 2: Markup directives: \name[args][args][...]
	where args are optional.

	These will be page elements which do not change surrounding text
	I.e \pagebreak, \contents

	Or these commands will affect the entire document, i.e \author
	To only affect specfic text, you need an inline or block command. 
	
Rule 3: Inline Markup: \name[args][...]{text}, 
	where args are optional. 
	
	These commands should only apply to the text inside {}. There is
	no LaTeX style \begin or \end. Text inside {}'s can span multiple
	lines.

Rule 4: Block Markup:  \name[args][...]:: followed by a
        consistently indented block, ignoring blank lines.

	i.e similar to how RST/GFM works.

Rule 5: Markup arguments have two forms: Positional and named

	[foo] passes the arg "foo". [name=foo], is the named form.

Rule 6: Shorthand will be defined interms of existing markup:

	As to which ascii art gets included, i'm open to a mix of
	RST and Markdown conventions. For example:

	"#*n title"  for headers, i.e \header[n]{title}
	*bold* for \bold{....}, similar for _foo..._

	These should not include having to count whitespace,
	using trailing whitespace, or precision ascii-art.

This seems to be relatively easy to write, and relatively easy to parse.
The grammar isn't tied to the document formatting, and is pretty easy to
remember the core rules. The bikeshed will be which ascii art shorthand
gets included, and the names of commands, but this could be set.

In theory we could use a directive, ala \shorthand[markdown] to control
which commands are used.
