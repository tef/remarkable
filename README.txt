"Why I hate your document markup format" 
--------------------------------------------------------------------------------

I have to write worksheets. Friends have to write technical documentation. I 
would like a pleasant way to write and manage documents that can be translated 
into web ready, ebook, and print formats. Without crying.

And yes, I have seen that XKCD. So let's look at the options:

              +
  Extensible  | LaTeX
              |
              |          ReStructuredText  
              |
              |  HTML
  Fixed       |                    Markdown
              |
              +------------------------------+
                
                Verbose             Shorthand
       
Markdown is great for small comments, blog posts. LaTeX is good for writing
a book on Mathematics. RST is good for python documentation, and HTML is good
for webpages.

Shorthand is good for blog posts, and making markup "look just like text", but
it comes at a price.

- Some require lining up characters and precision formatting. i.e
  exact underlines. This turns into drawing ASCII art.

- Nesting becomes awful. To put lists within lists often involves
  extra formatting or indentation to get it to work. 

- Implicit markup leads to Accidental commands. Sentences which start with 
  a letter or number. Including a document can change header heirarchy.
  It becomes harder to see what is being parsed, and much harder to 
  just get it to work.

- Semantic whitespace can be useful, but exact numbers of whitespace, 
  or trailing whitespace are a pain in the butt.

- New commands all look different, and it can be hard to recall each 
  special parsing case, or introduce new forms, without creating 
  incompatible variants.

What I really desire is something with a handful of shorthand for very common
cases, and a uniform syntax for commands, rather than different syntax
for inline, block, and other document markup.

Existing Solutions
--------------------------------------------------------------------------------

Markdown:

	- Considerable amounts of ASCII art. 
	- Everyone does it differently.
	- It's a HTML superset. Generating non HTML is painful.
	- Whitespace sensitive to the point of tears. Indents must be 4 spaces 
	  in some variants. Line continuations are 2 trailing spaces.
	- Zero tests or specs or grammars.
        - Great for blog comments. Awful for books.

LaTeX:

	- Turing Complete. Syntax is mostly Quirks.
	- How do I escape symbols? OH GOD HELP ME.
	- Weirdo error messages. You can't do that in horizontal mode. Overful
	  hbox.
	- Hard to recall edge cases of syntax.
	- Won't do EPUB, HTML, pleasantly. Tied to print output.
	- Possible to extend, but only in weirdo arcane macro language.
	- It's not a document format bob, it's a turing complete 
	  macro system. Did you just tell me to go fuck myself?
	- Unicode.

HTML:

	- Clunky at times, some formatting built in.
	- Won't generate references, indices, tables of contents.
	- Manual Demarcation of Paragraphs, Zero Shorthand.

ReStructuredText:

	- Some ASCII art (header titles, etc)
        - Headers are defined implicitly, so including a document
          can change what level headers are.
	- Extensible blocks but hard to add inline markup, without using
	  substitutions, or roles.
	- Table syntax, Header names involves shunting characters around
        - Reliance on shorthand makes it hard to remember all the options and 
          rules.

Scribble:

	http://docs.racket-lang.org/scribble/
	- Tied to racket.



Winner?

        ReStructuredText is the closest to what I desire, but the reliance
        on ascii markup causes headaches in longer documents.   

        Although some shorthand is useful, being able to unambiguously
        markup text is useful too. 

        What I desire is something similar in scope to ReStructuredText
        but with a standard mechanism for markup directives.
        

Proposal
--------------------------------------------------------------------------------

Goals:

	- Simple and short core grammar that is easy to remember, and to parse.
 	- Easy to generate epub, HTML, PDF from. 

	- A Text File renders as a text file. I.e linebreaks work by default.
	- Optionally, Commands take arguments, and/or a text block.
	- Text blocks can be inline, multiline, or an indented block.
	- Text blocks can have their contents marked as raw, and left unparsed.

	- A set of built in commands for common markup
	- ASCII-Art markup will exist, but as shorthand for existing commands
	  within the core grammar. 
	- Shorthand will not require exact indentation, trailing whitespace
          or drawing ascii art.

Non Goals:
	- Turing Complete. 
	  A larger attack surface, and makes it harder to process documents.
	- Macros
	  This is not a priority, but something that is being considered.

Overview:

	Double linebreaks mark paragraphs.
	Commands are \arg[name][..] with an optional text block, either
	- {text}
	- {{{text}}}
	- :: <indented block or blank lines>
	To turn off intepretation of text argument, use a !, \cmd!{text}
	Ascii-Art directives are shorthand for \cmd{...}

Example of syntax: (No shorthand, command names may change)

	\h[1]{Introduction}

	I like markup, and I cannot lie. You other coders can't deny. 
	When a document walks in oh god what am I doing with my life 
	this is a waste.

	\code![python]::
		print("Hello, World")
	
		print("Still in the python block")

	\list{
	Why
	Does
	It
	Hurt
	}

Syntax Rules:

Rule 0: \ is the command/escape character. \\ is a literal \ 

Rule 1: Two line breaks are the end of a paragraph.

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

	Why ::? I'm copying it from ReStructuredText.

Rule 5: Markup arguments have two forms: Positional and named

	[foo] passes the arg "foo". [name=foo], is the named form.

	We should declare a grammar for arguments. I imagine it will be
	text, where \] can be used to insert a ]

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

	Escapes should work still, so \* won't be interpreted as bold.

	Shorthand sets may be tied to directives, i.e \use[markdown, rst]



Bikeshedding
--------------------------------------------------------------------------------

Grammar tweaks and notes:
	
	\foo followed by {not text} needs to be
	 \foo\{not text} or \foo{}{not text}. 
	 
	Should the indent marker be '::' or otherwise.
		:: is used by ReStructuredText.

	Should the raw block character be !, i.e \foo!{....}

	Should multiline, or raw  blocks be forced to use {{{, }}} instead
	of {}, to avoid ambiguity?

Command Names:

	I imagine it will be a mixture of HTML, LaTeX and ReStructuredText
	
	Example:
		\code[python]{}, \h[1], \b, \i{...}, \pagebreak
		\note, \footnote, \sidenote, \ref, \link, 
		\author, \title, \version, \list, \table, \raw

Shorthand:

	I imagine the shorthand it will be a mixture of Markdown and
	ReStructuredText conventions.

	*bold*, _italic_, `fixed`
	# h1, ## h2, ### h3, and some shorthand for lists.

	But I don't think [foo](link) will be kept in. it will be
	\link[url]{text}.

	One linebreak = a linebreak
	Two linebreaks = a paragraph, ala GFM

Directives:
	
	We may have directives to turn on/off features like
	URLs into links, Smart Quotes,  

Turing Complete:

	The goal is to write a declarative markup, akin to HTML,
	ReStructuredText, or markdown. Rather than a turing-complete
	language like TeX, PDF or Postscript.

	This is something that isn't a priority, but would only
	be considered if other techniques fail.

	Turing completeness is an attack surface. I do not
	want it to be required to parse, or render the document.

Macros/Substitutions:

	We should be able to do RST style substitutions, for common 
	phrases, like \replace[\foo]{Foo Corp Inc}, but this doesn't
	require turing completeness.

	Ideally, we should be able to define the ascii art shorthand
	in a prelude, rather than baking it in to the parser. This
	would allow users to switch between RST and Markdown flavours
	using directives

	\shorthand[line][pattern]{replacement}
	\shorthand[inline][pattern]{replacement}

	I.e we can define limited substitutions based around common
	markup patterns: *word*, -lists

	However, I do not think regular expressions would be a pleasant
	way to write, test, or debug markup. We should learn the lessons
	of WikiText and Markdown.

	The goal is to be able to have the ascii-art shorthand
	without baking it into the grammar.
	
	Templating is closely related too.

Linebreaks:
	manual linebreaking is awful, but automatic ones aren't great either
	welp

Things I will have to do before this is close to usable
--------------------------------------------------------------------------------

[ ] Define core grammar
[ ] Write parser, tests
[ ] Define core commands for markup
[ ] Define shorthand
[ ] Build HTML, PDF, epub output
[ ] Pandoc support


