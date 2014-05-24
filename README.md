This is a fork of CXML, an XML parser written in Common Lisp. Its
interface is a superset of CXML’s, so refer to the
[CXML documentation][cxml] for usage.

This fork was originally created as a last resort, to fix a
longstanding, critical bug in Klacks, namely its broken handling of
namespaces.

(Klacks maintained a namespace stack, but never popped it when
namespaces went out of scope. This meant that, when the default
namespace was changed, it never got changed back. Thus Klacks could
not, say, parse Feedburner feeds.)

Later, this fork was extended to provide restarts for common
well-formedness violations.

For most such violations, there is only one reasonable way to proceed;
this is made available as a `continue` restart.

    (handler-bind ((cxml:well-formedness-violation #'continue))
      (cxml:parse ...))

This is enough to handle many characteristic problems with XML in the
wild: leading and trailing junk, unescaped ampersands, illegal
characters, and DTDs (a feature which is largely indistinguishable
from a bug).

#### Undefined entities

Undefined entities are signaled as `undefined-entity`, a subtype of
`well-formedness-violation`. The name of the undefined entity can be
read with `undefined-entity-name`. You can omit the entity with
`continue`, or manually supply an expansion with `use-value`.

#### Undeclared namespaces

Undeclared namespace prefixes are signaled as `undeclared-namespace`,
a subtype of `well-formedness-violation`. The prefix of the namespace
can be read with `undeclared-namespace-prefix`.

The only restart provided for `undeclared-namespace` is `store-value`:
to recover, you must provide a URI for the namespace. The XML is then
re-written exactly as if the element had the appropriate `xmlns`
attribute.

#### A note about conformance

Although I do not care much about conformance, I do not think that
providing restarts can be said to be non-conforming. My invoking a
restart is no different than my stopping, editing the document, and
re-submitting it to the parser, except that it saves time. The fact
that the XML spec was written in an era when *programming language*
was a euphemism for *Java* does not mean we should have to write Java
in Lisp to deal with XML.

[CXML]: http://common-lisp.net/project/cxml/
