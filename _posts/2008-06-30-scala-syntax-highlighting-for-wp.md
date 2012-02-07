--- 
wordpress_id: "27"
layout: blog_post
title: Scala Syntax Highlighting for WP
wordpress_url: http://martin.elwin.com/blog/?p=27
---
Writing the previous post I realized that the Wordpress plugin for syntax highligting I was using (Highlight Source Pro) didn't support Scala.

Instead I tried the WP-Syntax plugin, but this didn't support Symbol literals:

{% highlight scala-old %}
val sym = 'foobar
println("Symbol is: " + sym)
val other = 'barfoo
{% endhighlight %}

This seems to be because of single quote being interpreted as a quotation character. A fix seems to be removing the single quote from the Geshi language definition file `scala.php` in the `wp-syntax/geshi/geshi` directory. While there one can also specify a regex for symbols and give them their own color:

{% highlight scala %}
val sym = 'foobar
println("Symbol is: " + sym)
val other = 'barfoo
{% endhighlight %}

Also check out the <a href="https://lampsvn.epfl.ch/trac/scala/browser/scala-tool-support/trunk/src/geshi/scala.php">`scala.php` in the LAMP repository</a> - it seems to add some additional keywords and other colors. I used this and made the changes described above.

Full new `scala.php` for Geshi after the break.

<!--more-->

{% highlight php %}
<?php
/*************************************************************************************
 * scala.php
 * --------
 * Author: Geoffrey Washburn (washburn@acm.ogr)
 * Copyright: (c) 2004 Nigel McNie (http://qbnz.com/highlighter/)
 * Release Version: ???
 * Date Started: 2008/01/03
 *
 * Scala language file for GeSHi.
 *
 * CHANGES
 * -------
 * 2007/01/03
 *   -  Created by copying the Java highlighter
 *
 * TODO
 * -------------------------
 * * Finish
 *
 *************************************************************************************
 *
 *     This file is part of GeSHi.
 *
 *   GeSHi is free software; you can redistribute it and/or modify
 *   it under the terms of the GNU General Public License as published by
 *   the Free Software Foundation; either version 2 of the License, or
 *   (at your option) any later version.
 *
 *   GeSHi is distributed in the hope that it will be useful,
 *   but WITHOUT ANY WARRANTY; without even the implied warranty of
 *   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 *   GNU General Public License for more details.
 *
 *   You should have received a copy of the GNU General Public License
 *   along with GeSHi; if not, write to the Free Software
 *   Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA
 *
 ************************************************************************************/

$language_data = array (
        'LANG_NAME' => 'Scala',
        'COMMENT_SINGLE' => array(1 => '//'),   /* import statements are not comments! */
        'COMMENT_MULTI' => array('/*' => '*/'),
        'CASE_KEYWORDS' => GESHI_CAPS_NO_CHANGE,
        'QUOTEMARKS' => array('"', '"""'),
        'ESCAPE_CHAR' => '\\',
        'KEYWORDS' => array(
                1 => array(
                        /* Scala keywords, part 1: control flow */
                        'case', 'default', 'do', 'else', 'for',
                        'if', 'match', 'while'
                        ),
                2 => array(
                        /* Scala keywords, part 2 */
                        'return', 'throw',
                        'try', 'catch', 'finally',
                        'abstract', 'class', 'def', 'extends',
                        'final', 'forSome', 'implicit', 'import',
                        'lazy', 'new', 'object', 'override', 'package',
                        'private', 'protected',
                        'requires', 'sealed', 'super', 'this', 'trait', 'type',
                        'val', 'var', 'with', 'yield'
                        ),
                3 => array(
                        /* Scala keywords, part 3: standard value types */
                        'unit', 'Unit', 'boolean', 'Boolean', 'int', 'Int', 'Any', 'AnyVal', 'Nothing',
                        ),
                4 => array(
                        /* other reserved words in Scala: literals */
                        /* should be styled to look similar to numbers and Strings */
                        'false', 'null', 'true'
                        ),
                5 => array(
                        /* Scala reference types */
                        'AnyRef', 'Null', 'List', 'String', 'Integer', 'Option', 'Array'
                        )

                ),
        'SYMBOLS' => array(
                ':', '*', '&', '%', '!', ';', '<', '>', '?', '_', '=', '=>',
                '<-', '<:', '<%', '>:', '#', '@'
                ),
        'CASE_SENSITIVE' => array(
                GESHI_COMMENTS => true,
                /* all Scala keywords are case sensitive */
                1 => true, 2 => true, 3 => true, 4 => true, 5 => true ),
        'STYLES' => array(
                'KEYWORDS' => array(
                        1 => 'color: #b1b100;',
                        2 => 'color: #000000; font-weight: bold;',
                        3 => 'color: #993333;',
                        4 => 'color: #b13366;',
                        5 => 'color: #aaaadd;'
                        ),
                'SYMBOLS' => array(
                        0 => 'color: #FFAA00;'
                        ),
                'COMMENTS' => array(
                        1 => 'color: #808080; font-style: italic;',
                        'MULTI' => 'color: #808080; font-style: italic;'
                        ),
                'ESCAPE_CHAR' => array(
                        0 => 'color: #000099; font-weight: bold;'
                        ),
                'BRACKETS' => array(
                        0 => 'color: #66cc66;'
                        ),
                'STRINGS' => array(
                        0 => 'color: #ff0000;'
                        ),
                'NUMBERS' => array(
                        0 => 'color: #cc66cc;'
                        ),
                'METHODS' => array(
                        1 => 'color: #006600;',
                        2 => 'color: #006600;'
                        ),
                'SCRIPT' => array(
                        ),
                'REGEXPS' => array(
                        0 => 'color: #008000;'
                        )
                ),
        'URLS' => array(
                1 => '',
                2 => '',
                3 => '',
                4 => ''
                ),
        'OOLANG' => true,
        'OBJECT_SPLITTERS' => array(
                1 => '.'
                ),
        'REGEXPS' => array(
               0 => "'[a-zA-Z_][a-zA-Z0-9_]*"
                ),
        'STRICT_MODE_APPLIES' => GESHI_NEVER,
        'SCRIPT_DELIMITERS' => array(
                ),
        'HIGHLIGHT_STRICT_BLOCK' => array(
                )
);

?>
{% endhighlight %}
