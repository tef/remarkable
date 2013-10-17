"Why I hate your document markup format" 
--------------------------------------------------------------------------------

I have to write worksheets. Friends have to write technical documentation. I 
would like a pleasant way to write and manage documents that can be translated 
into web ready, ebook, and print formats. Without crying.

And yes, I have seen that XKCD. So let's look at the options:


Existing Solutions
--------------------------------------------------------------------------------
Markdown:

	- Considerable amounts of ASCII art. 
	- Everyone does it differently.
	- It's a HTML superset. Generating non HTML is painful.
	- Whitespace sensitive to the point of tears. Indents must be 4 spaces 
	  in some variants. Line continuations are 2 trailing spaces.
	- Zero tests or specs or grammars.

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

Scribble
	http://docs.racket-lang.org/scribble/
	- Tied to racket.

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

Proposal
--------------------------------------------------------------------------------

Goals:

	- Simple and short core grammar that is easy to remember. 
	- Easy to write a parser for the core grammar.
	- Easy to define new commands without inventing new syntax.
	- Commands can take parameters, rather than needing new commands.
	- Commands can be inline, multiline, or take an indented block.
	- Commands can be raw, so the text contents do not need to be escaped.
	- A Text File renders as a text file. I.e linebreaks work by default.
 	- A set of built in commands for document/book markup.
 	- Easy to generate epub, HTML, PDF from. 
	- ASCII-Art markup will exist, but as shorthand for existing commands
	  within the core grammar. 
	- Shorthand will not require exact indentation, trailing whitespace
          or drawing ascii art.

Core Grammar: Plain text with embedded commands 

Rule 0: \ is the command/escape character. \\ is a literal \ 

Rule 1: Line breaks are line breaks, unless preceded with a \

Rule 2: Markup directives without text: \name[args][args][...]
	where args are optional, optionally followed by {} to end command.

	These will be page elements which do not change surrounding text
	I.e \pagebreak, \contents, or \u[65] for unicode escapes.

	Or these commands will affect the entire document, i.e \author
	To only affect specfic text, you need an inline or block command. 
	
Rule 3: Inline Markup: \name[args][...]{text}, \name[args]{{{text}}} 
	where args are optional. 
	
	These commands should only apply to the text inside {}. There is
	no LaTeX style \begin or \end. Text inside {}'s can span multiple
	lines.

	Optionally, {{{,}}} can be used instead. 

Rule 4: Block Markup:  \name[args][...]:: followed by a
        consistently indented block, ignoring blank lines.

	i.e similar to how RST/GFM works.

	Why ::? I'm copying it from Restructured Text.

Rule 5: Markup arguments have two forms: Positional and named

	[foo] passes the arg "foo". [name=foo], is the named form.

Rule 6: Raw blocks \name!{text}
	The command name can be suffixed with a ! to tell the parser
	to ignore any commands inside. 

	\code![python]{{{print("Hello, world")}}}

Rule 7: Shorthand will be defined interms of existing markup:

	As to which ascii art gets included, i'm open to a mix of
	RST and Markdown conventions. For example:

	"#*n title"  for headers, i.e \header[n]{title}
	*bold* for \bold{....}, similar for _foo..._

	These should not include having to count whitespace,
	using trailing whitespace, or precision ascii-art.


Example of syntax: (No shorthand, command names may change)

	\h[1]{Introduction}

	I like markup, and I cannot lie. You other coders can't deny. \
	When a document walks in oh god what am I doing with my life \
	this is a waste.

	\code![python]::
		print("Hello, World")
	
	\list{
	Why
	Does
	It
	Hurt
	}

	

Bikeshedding
--------------------------------------------------------------------------------

Grammar tweaks and notes:
	
	\foo followed by {not text} needs to be
	 \foo\{not text} or \foo{}{not text}. 
	 
	Should the indent marker be '::' or otherwise.
		:: is used by Restructured Text.

	Should the raw block character be !, i.e \foo!{....}

	Should multiline, or raw  blocks be forced to use {{{, }}} instead
	of {}, to avoid ambiguity?

Command Names:

	I imagine it will be a mixture of HTML, LaTeX and ReStructured Text
	
	Example:
		\code[python]{}, \h[1], \b, \i{...}, \pagebreak
		\note, \footnote, \sidenote, \ref, \link, 
		\author, \title, \version, \list, \table, \raw

Shorthand:

	I imagine the shorthand it will be a mixture of Markdown and
	Restructured Text conventions.

	*bold*, _italic_, `fixed`
	# h1, ## h2, ### h3, and some shorthand for lists.

	But I don't think [foo](link) will be kept in. it will be
	\link[url]{text}.

Directives:
	
	We may have directives to turn on/off features like
	URLs into links, Smart Quotes,  

Macros:

	Although TeX is turing complete, I'm happy to pass the buck to
	the tool parsing the text, but it might be useful to have
	search/replace macros as a possible command:

	\shorthand[^(#+)(.*)]{\h[$1]{$2}}

	It would be pleasing to be able to define all of the shorthand
	in a prelude, rather than the parser, but debugging the output
	after preprocessing will be unpleasant.

	Markdown and WikiText are both defined as a series of Regular expressions
	Let's not go down that way. PLEASE.

Things I will have to do before this is close to usable
--------------------------------------------------------------------------------

[ ] Define core grammar
[ ] Write parser, tests
[ ] Define core commands for markup
[ ] Define shorthand
[ ] Build HTML, PDF, epub output

