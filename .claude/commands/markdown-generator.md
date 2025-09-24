# Markdown-Generator

Claude Code slash command to convert text files to well-formatted Markdown or improve existing Markdown files.

**File scope:** `$ARGUMENTS`

## Process

1. **For .txt files:** Create a new .md file with the same name and convert the text content to proper Markdown format
2. **For .md files:** Update the existing file with improved Markdown formatting and structure
3. **For other text content:** Transform into clean, readable Markdown with appropriate:
   - Headings and subheadings
   - Lists and formatting
   - Code blocks where applicable
   - Proper spacing and structure

## Guidelines

- Use semantic Markdown structure (proper heading hierarchy)
- Apply consistent formatting throughout
- Add table of contents for longer documents
- Use code blocks for technical content
- Ensure proper line spacing and readability
- Reference the syntax guide below for any formatting questions

**IMPORTANT:**
- Use the index below to navigate the reference guide efficiently and avoid context issues
- **Preserve original content:** Unless the user specifically requests only grammatical fixes, do NOT rewrite or change the actual text content. Focus solely on Markdown formatting and structure while keeping the user's original words and meaning intact

---

# Markdown Syntax Reference Guide

## Quick Reference Index

### Basic Syntax
- [Headings](#headings) - Create titles and section headers
- [Paragraphs](#paragraphs) - Format text blocks
- [Line Breaks](#line-breaks) - Control line spacing
- [Emphasis](#emphasis) - Bold and italic text
  - [Bold](#bold) - Make text bold
  - [Italic](#italic) - Make text italic
  - [Bold and Italic](#bold-and-italic) - Combine both
- [Blockquotes](#blockquotes) - Quote text blocks
- [Lists](#lists) - Organize items
  - [Ordered Lists](#ordered-lists) - Numbered lists
  - [Unordered Lists](#unordered-lists) - Bulleted lists
  - [Adding Elements in Lists](#adding-elements-in-lists) - Enhanced list items
- [Code](#code) - Inline and block code
  - [Code Blocks](#code-blocks) - Multi-line code
- [Horizontal Rules](#horizontal-rules) - Section dividers
- [Links](#links) - Web and internal links
  - [Reference-style Links](#reference-style-links) - Cleaner link formatting
- [Images](#images) - Pictures and graphics
- [Escaping Characters](#escaping-characters) - Display literal characters
- [HTML](#html) - Direct HTML usage

### Extended Syntax
- [Tables](#tables) - Data in rows and columns
- [Fenced Code Blocks](#fenced-code-blocks) - Enhanced code formatting
  - [Syntax Highlighting](#syntax-highlighting) - Language-specific coloring
- [Footnotes](#footnotes) - Reference notes
- [Heading IDs](#heading-ids) - Custom anchor links
- [Definition Lists](#definition-lists) - Term definitions
- [Strikethrough](#strikethrough) - Cross out text
- [Task Lists](#task-lists) - Checkboxes and todos
- [Emoji](#emoji) - Emoticons and symbols
- [Highlight](#highlight) - Highlighted text
- [Subscript](#subscript) - Lower positioned text
- [Superscript](#superscript) - Higher positioned text
- [Automatic URL Linking](#automatic-url-linking) - Auto-convert URLs

### Best Practices Quick Reference
- [Heading Best Practices](#heading-best-practices)
- [Paragraph Best Practices](#paragraph-best-practices)
- [Line Break Best Practices](#line-break-best-practices)
- [Bold Best Practices](#bold-best-practices)
- [Italic Best Practices](#italic-best-practices)
- [Bold and Italic Best Practices](#bold-and-italic-best-practices)
- [Blockquotes Best Practices](#blockquotes-best-practices)
- [Ordered List Best Practices](#ordered-list-best-practices)
- [Unordered List Best Practices](#unordered-list-best-practices)
- [Horizontal Rule Best Practices](#horizontal-rule-best-practices)
- [Link Best Practices](#link-best-practices)
- [HTML Best Practices](#html-best-practices)

---

## Overview

Nearly all Markdown applications support the basic syntax outlined in the original Markdown design document. There are minor variations and discrepancies between Markdown processors — those are noted inline wherever possible.

## Headings

To create a heading, add number signs (#) in front of a word or phrase. The number of number signs you use should correspond to the heading level. For example, to create a heading level three (<h3>), use three number signs (e.g., ### My Header).
Markdown 	HTML 	Rendered Output
# Heading level 1 	<h1>Heading level 1</h1> 	
Heading level 1
## Heading level 2 	<h2>Heading level 2</h2> 	
Heading level 2
### Heading level 3 	<h3>Heading level 3</h3> 	
Heading level 3
#### Heading level 4 	<h4>Heading level 4</h4> 	
Heading level 4
##### Heading level 5 	<h5>Heading level 5</h5> 	
Heading level 5
###### Heading level 6 	<h6>Heading level 6</h6> 	
Heading level 6

### Alternate Syntax

Alternatively, on the line below the text, add any number of == characters for heading level 1 or -- characters for heading level 2.
Markdown 	HTML 	Rendered Output
Heading level 1
=============== 	<h1>Heading level 1</h1> 	
Heading level 1
Heading level 2
--------------- 	<h2>Heading level 2</h2> 	
Heading level 2

### Heading Best Practices

Markdown applications don’t agree on how to handle a missing space between the number signs (#) and the heading name. For compatibility, always put a space between the number signs and the heading name.
✅  Do this 	❌  Don't do this
# Here's a Heading

	#Here's a Heading

You should also put blank lines before and after a heading for compatibility.
✅  Do this 	❌  Don't do this
Try to put a blank line before...

# Heading

...and after a heading. 	Without blank lines, this might not look right.
# Heading
Don't do this!

## Paragraphs

To create paragraphs, use a blank line to separate one or more lines of text.
Markdown 	HTML 	Rendered Output
I really like using Markdown.

I think I'll use it to format all of my documents from now on. 	<p>I really like using Markdown.</p>

<p>I think I'll use it to format all of my documents from now on.</p> 	

I really like using Markdown.

I think I'll use it to format all of my documents from now on.

### Paragraph Best Practices

Unless the paragraph is in a list, don’t indent paragraphs with spaces or tabs.
Note: If you need to indent paragraphs in the output, see the section on how to indent (tab).
✅  Do this 	❌  Don't do this
Don't put tabs or spaces in front of your paragraphs.

Keep lines left-aligned like this.

	    This can result in unexpected formatting problems.

  Don't add tabs or spaces in front of paragraphs.

## Line Breaks

To create a line break or new line (<br>), end a line with two or more spaces, and then type return.
Markdown 	HTML 	Rendered Output
This is the first line.  
And this is the second line. 	<p>This is the first line.<br>
And this is the second line.</p> 	

This is the first line.
And this is the second line.

### Line Break Best Practices

You can use two or more spaces (commonly referred to as “trailing whitespace”) for line breaks in nearly every Markdown application, but it’s controversial. It’s hard to see trailing whitespace in an editor, and many people accidentally or intentionally put two spaces after every sentence. For this reason, you may want to use something other than trailing whitespace for line breaks. If your Markdown application supports HTML, you can use the <br> HTML tag.

For compatibility, use trailing white space or the <br> HTML tag at the end of the line.

There are two other options I don’t recommend using. CommonMark and a few other lightweight markup languages let you type a backslash (\) at the end of the line, but not all Markdown applications support this, so it isn’t a great option from a compatibility perspective. And at least a couple lightweight markup languages don’t require anything at the end of the line — just type return and they’ll create a line break.
✅  Do this 	❌  Don't do this
First line with two spaces after.  
And the next line.

First line with the HTML tag after.<br>
And the next line.

	First line with a backslash after.\
And the next line.

First line with nothing after.
And the next line.

## Emphasis

You can add emphasis by making text bold or italic.

### Bold

To bold text, add two asterisks or underscores before and after a word or phrase. To bold the middle of a word for emphasis, add two asterisks without spaces around the letters.
Markdown 	HTML 	Rendered Output
I just love **bold text**. 	I just love <strong>bold text</strong>. 	I just love bold text.
I just love __bold text__. 	I just love <strong>bold text</strong>. 	I just love bold text.
Love**is**bold 	Love<strong>is</strong>bold 	Loveisbold

#### Bold Best Practices

Markdown applications don’t agree on how to handle underscores in the middle of a word. For compatibility, use asterisks to bold the middle of a word for emphasis.
✅  Do this 	❌  Don't do this
Love**is**bold 	Love__is__bold

### Italic

To italicize text, add one asterisk or underscore before and after a word or phrase. To italicize the middle of a word for emphasis, add one asterisk without spaces around the letters.
Markdown 	HTML 	Rendered Output
Italicized text is the *cat's meow*. 	Italicized text is the <em>cat's meow</em>. 	Italicized text is the cat’s meow.
Italicized text is the _cat's meow_. 	Italicized text is the <em>cat's meow</em>. 	Italicized text is the cat’s meow.
A*cat*meow 	A<em>cat</em>meow 	Acatmeow

#### Italic Best Practices

Markdown applications don’t agree on how to handle underscores in the middle of a word. For compatibility, use asterisks to italicize the middle of a word for emphasis.
✅  Do this 	❌  Don't do this
A*cat*meow 	A_cat_meow

### Bold and Italic

To emphasize text with bold and italics at the same time, add three asterisks or underscores before and after a word or phrase. To bold and italicize the middle of a word for emphasis, add three asterisks without spaces around the letters.
Markdown 	HTML 	Rendered Output
This text is ***really important***. 	This text is <em><strong>really important</strong></em>. 	This text is really important.
This text is ___really important___. 	This text is <em><strong>really important</strong></em>. 	This text is really important.
This text is __*really important*__. 	This text is <em><strong>really important</strong></em>. 	This text is really important.
This text is **_really important_**. 	This text is <em><strong>really important</strong></em>. 	This text is really important.
This is really***very***important text. 	This is really<em><strong>very</strong></em>important text. 	This is reallyveryimportant text.
Note: The order of the em and strong tags might be reversed depending on the Markdown processor you're using.

#### Bold and Italic Best Practices

Markdown applications don’t agree on how to handle underscores in the middle of a word. For compatibility, use asterisks to bold and italicize the middle of a word for emphasis.
✅  Do this 	❌  Don't do this
This is really***very***important text. 	This is really___very___important text.

## Blockquotes

To create a blockquote, add a > in front of a paragraph.

> Dorothy followed her through many of the beautiful rooms in her castle.

The rendered output looks like this:

    Dorothy followed her through many of the beautiful rooms in her castle.

### Blockquotes with Multiple Paragraphs

Blockquotes can contain multiple paragraphs. Add a > on the blank lines between the paragraphs.

> Dorothy followed her through many of the beautiful rooms in her castle.
>
> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.

The rendered output looks like this:

    Dorothy followed her through many of the beautiful rooms in her castle.

    The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.

### Nested Blockquotes

Blockquotes can be nested. Add a >> in front of the paragraph you want to nest.

> Dorothy followed her through many of the beautiful rooms in her castle.
>
>> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.

The rendered output looks like this:

    Dorothy followed her through many of the beautiful rooms in her castle.

        The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.

### Blockquotes with Other Elements

Blockquotes can contain other Markdown formatted elements. Not all elements can be used — you’ll need to experiment to see which ones work.

> #### The quarterly results look great!
>
> - Revenue was off the chart.
> - Profits were higher than ever.
>
>  *Everything* is going according to **plan**.

The rendered output looks like this:

    The quarterly results look great!

        Revenue was off the chart.
        Profits were higher than ever.

    Everything is going according to plan.

Blockquotes Best Practices

For compatibility, put blank lines before and after blockquotes.
✅  Do this 	❌  Don't do this
Try to put a blank line before...

> This is a blockquote

...and after a blockquote. 	Without blank lines, this might not look right.
> This is a blockquote
Don't do this!

## Lists

You can organize items into ordered and unordered lists.

### Ordered Lists

To create an ordered list, add line items with numbers followed by periods. The numbers don’t have to be in numerical order, but the list should start with the number one.
Markdown 	HTML 	Rendered Output
1. First item
2. Second item
3. Third item
4. Fourth item 	<ol>
  <li>First item</li>
  <li>Second item</li>
  <li>Third item</li>
  <li>Fourth item</li>
</ol> 	

    First item
    Second item
    Third item
    Fourth item

1. First item
1. Second item
1. Third item
1. Fourth item 	<ol>
  <li>First item</li>
  <li>Second item</li>
  <li>Third item</li>
  <li>Fourth item</li>
</ol> 	

    First item
    Second item
    Third item
    Fourth item

1. First item
8. Second item
3. Third item
5. Fourth item 	<ol>
  <li>First item</li>
  <li>Second item</li>
  <li>Third item</li>
  <li>Fourth item</li>
</ol> 	

    First item
    Second item
    Third item
    Fourth item

1. First item
2. Second item
3. Third item
    1. Indented item
    2. Indented item
4. Fourth item 	<ol>
  <li>First item</li>
  <li>Second item</li>
  <li>Third item
    <ol>
      <li>Indented item</li>
      <li>Indented item</li>
    </ol>
  </li>
  <li>Fourth item</li>
</ol> 	

    First item
    Second item
    Third item
        Indented item
        Indented item
    Fourth item

#### Ordered List Best Practices

CommonMark and a few other lightweight markup languages let you use a parenthesis ()) as a delimiter (e.g., 1) First item), but not all Markdown applications support this, so it isn't a great option from a compatibility perspective. For compatibility, use periods only.

✅  Do this 	❌  Don't do this
1. First item
2. Second item 	1) First item
2) Second item

### Unordered Lists

To create an unordered list, add dashes (-), asterisks (*), or plus signs (+) in front of line items. Indent one or more items to create a nested list.
Markdown 	HTML 	Rendered Output
- First item
- Second item
- Third item
- Fourth item 	<ul>
  <li>First item</li>
  <li>Second item</li>
  <li>Third item</li>
  <li>Fourth item</li>
</ul> 	

    First item
    Second item
    Third item
    Fourth item

* First item
* Second item
* Third item
* Fourth item 	<ul>
  <li>First item</li>
  <li>Second item</li>
  <li>Third item</li>
  <li>Fourth item</li>
</ul> 	

    First item
    Second item
    Third item
    Fourth item

+ First item
+ Second item
+ Third item
+ Fourth item 	<ul>
  <li>First item</li>
  <li>Second item</li>
  <li>Third item</li>
  <li>Fourth item</li>
</ul> 	

    First item
    Second item
    Third item
    Fourth item

- First item
- Second item
- Third item
    - Indented item
    - Indented item
- Fourth item 	<ul>
  <li>First item</li>
  <li>Second item</li>
  <li>Third item
    <ul>
      <li>Indented item</li>
      <li>Indented item</li>
    </ul>
  </li>
  <li>Fourth item</li>
</ul> 	

    First item
    Second item
    Third item
        Indented item
        Indented item
    Fourth item

#### Starting Unordered List Items With Numbers

If you need to start an unordered list item with a number followed by a period, you can use a backslash (\) to escape the period.
Markdown 	HTML 	Rendered Output
- 1968\. A great year!
- I think 1969 was second best. 	<ul>
  <li>1968. A great year!</li>
  <li>I think 1969 was second best.</li>
</ul> 	

    1968. A great year!
    I think 1969 was second best.

#### Unordered List Best Practices

Markdown applications don’t agree on how to handle different delimiters in the same list. For compatibility, don’t mix and match delimiters in the same list — pick one and stick with it.
✅  Do this 	❌  Don't do this
- First item
- Second item
- Third item
- Fourth item 	+ First item
* Second item
- Third item
+ Fourth item

### Adding Elements in Lists

To add another element in a list while preserving the continuity of the list, indent the element four spaces or one tab, as shown in the following examples.
Tip: If things don't appear the way you expect, double check that you've indented the elements in the list four spaces or one tab.
#### Paragraphs

* This is the first list item.
* Here's the second list item.

    I need to add another paragraph below the second list item.

* And here's the third list item.

The rendered output looks like this:

    This is the first list item.

    Here’s the second list item.

    I need to add another paragraph below the second list item.
    And here’s the third list item.

#### Blockquotes

* This is the first list item.
* Here's the second list item.

    > A blockquote would look great below the second list item.

* And here's the third list item.

The rendered output looks like this:

    This is the first list item.

    Here’s the second list item.

        A blockquote would look great below the second list item.

    And here’s the third list item.

Code Blocks

Code blocks are normally indented four spaces or one tab. When they’re in a list, indent them eight spaces or two tabs.

1. Open the file.
2. Find the following code block on line 21:

        <html>
          <head>
            <title>Test</title>
          </head>

3. Update the title to match the name of your website.

The rendered output looks like this:

    Open the file.

    Find the following code block on line 21:

     <html>
       <head>
         <title>Test</title>
       </head>

    Update the title to match the name of your website.

Images

1. Open the file containing the Linux mascot.
2. Marvel at its beauty.

    ![Tux, the Linux mascot](/assets/images/tux.png)

3. Close the file.

The rendered output looks like this:

    Open the file containing the Linux mascot.

    Marvel at its beauty.

    Tux, the Linux mascot
    Close the file.

Lists

You can nest an unordered list in an ordered list, or vice versa.

1. First item
2. Second item
3. Third item
    - Indented item
    - Indented item
4. Fourth item

The rendered output looks like this:

    First item
    Second item
    Third item
        Indented item
        Indented item
    Fourth item

## Code

To denote a word or phrase as code, enclose it in backticks (`).
Markdown 	HTML 	Rendered Output
At the command prompt, type `nano`. 	At the command prompt, type <code>nano</code>. 	At the command prompt, type nano.

### Escaping Backticks

If the word or phrase you want to denote as code includes one or more backticks, you can escape it by enclosing the word or phrase in double backticks (``).
Markdown 	HTML 	Rendered Output
``Use `code` in your Markdown file.`` 	<code>Use `code` in your Markdown file.</code> 	Use `code` in your Markdown file.

### Code Blocks

To create code blocks, indent every line of the block by at least four spaces or one tab.

    <html>
      <head>
      </head>
    </html>

The rendered output looks like this:

<html>
  <head>
  </head>
</html>

Note: To create code blocks without indenting lines, use fenced code blocks.

## Horizontal Rules

To create a horizontal rule, use three or more asterisks (***), dashes (---), or underscores (___) on a line by themselves.

***

---

_________________

The rendered output of all three looks identical:
Horizontal Rule Best Practices

For compatibility, put blank lines before and after horizontal rules.
✅  Do this 	❌  Don't do this
Try to put a blank line before...

---

...and after a horizontal rule. 	Without blank lines, this would be a heading.
---
Don't do this!

## Links

To create a link, enclose the link text in brackets (e.g., [Duck Duck Go]) and then follow it immediately with the URL in parentheses (e.g., (https://duckduckgo.com)).

My favorite search engine is [Duck Duck Go](https://duckduckgo.com).

The rendered output looks like this:

My favorite search engine is Duck Duck Go.
Note: To link to an element on the same page, see linking to heading IDs. To create a link that opens in a new tab or window, see the section on link targets.
Adding Titles

You can optionally add a title for a link. This will appear as a tooltip when the user hovers over the link. To add a title, enclose it in quotation marks after the URL.

My favorite search engine is [Duck Duck Go](https://duckduckgo.com "The best search engine for privacy").

The rendered output looks like this:

My favorite search engine is Duck Duck Go.
URLs and Email Addresses

To quickly turn a URL or email address into a link, enclose it in angle brackets.

<https://www.markdownguide.org>
<fake@example.com>

The rendered output looks like this:

https://www.markdownguide.org
fake@example.com
Formatting Links

To emphasize links, add asterisks before and after the brackets and parentheses. To denote links as code, add backticks in the brackets.

I love supporting the **[EFF](https://eff.org)**.
This is the *[Markdown Guide](https://www.markdownguide.org)*.
See the section on [`code`](#code).

The rendered output looks like this:

I love supporting the EFF.
This is the Markdown Guide.
See the section on code.
Reference-style Links

Reference-style links are a special kind of link that make URLs easier to display and read in Markdown. Reference-style links are constructed in two parts: the part you keep inline with your text and the part you store somewhere else in the file to keep the text easy to read.
Formatting the First Part of the Link

The first part of a reference-style link is formatted with two sets of brackets. The first set of brackets surrounds the text that should appear linked. The second set of brackets displays a label used to point to the link you’re storing elsewhere in your document.

Although not required, you can include a space between the first and second set of brackets. The label in the second set of brackets is not case sensitive and can include letters, numbers, spaces, or punctuation.

This means the following example formats are roughly equivalent for the first part of the link:

    [hobbit-hole][1]
    [hobbit-hole] [1]

Formatting the Second Part of the Link

The second part of a reference-style link is formatted with the following attributes:

    The label, in brackets, followed immediately by a colon and at least one space (e.g., [label]: ).
    The URL for the link, which you can optionally enclose in angle brackets.
    The optional title for the link, which you can enclose in double quotes, single quotes, or parentheses.

This means the following example formats are all roughly equivalent for the second part of the link:

    [1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle
    [1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle "Hobbit lifestyles"
    [1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle 'Hobbit lifestyles'
    [1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle (Hobbit lifestyles)
    [1]: <https://en.wikipedia.org/wiki/Hobbit#Lifestyle> "Hobbit lifestyles"
    [1]: <https://en.wikipedia.org/wiki/Hobbit#Lifestyle> 'Hobbit lifestyles'
    [1]: <https://en.wikipedia.org/wiki/Hobbit#Lifestyle> (Hobbit lifestyles)

You can place this second part of the link anywhere in your Markdown document. Some people place them immediately after the paragraph in which they appear while other people place them at the end of the document (like endnotes or footnotes).
An Example Putting the Parts Together

Say you add a URL as a standard URL link to a paragraph and it looks like this in Markdown:

In a hole in the ground there lived a hobbit. Not a nasty, dirty, wet hole, filled with the ends
of worms and an oozy smell, nor yet a dry, bare, sandy hole with nothing in it to sit down on or to
eat: it was a [hobbit-hole](https://en.wikipedia.org/wiki/Hobbit#Lifestyle "Hobbit lifestyles"), and that means comfort.

Though it may point to interesting additional information, the URL as displayed really doesn’t add much to the existing raw text other than making it harder to read. To fix that, you could format the URL like this instead:

In a hole in the ground there lived a hobbit. Not a nasty, dirty, wet hole, filled with the ends
of worms and an oozy smell, nor yet a dry, bare, sandy hole with nothing in it to sit down on or to
eat: it was a [hobbit-hole][1], and that means comfort.

[1]: <https://en.wikipedia.org/wiki/Hobbit#Lifestyle> "Hobbit lifestyles"

In both instances above, the rendered output would be identical:

    In a hole in the ground there lived a hobbit. Not a nasty, dirty, wet hole, filled with the ends of worms and an oozy smell, nor yet a dry, bare, sandy hole with nothing in it to sit down on or to eat: it was a hobbit-hole, and that means comfort.

and the HTML for the link would be:

<a href="https://en.wikipedia.org/wiki/Hobbit#Lifestyle" title="Hobbit lifestyles">hobbit-hole</a>

Link Best Practices

Markdown applications don’t agree on how to handle spaces in the middle of a URL. For compatibility, try to URL encode any spaces with %20. Alternatively, if your Markdown application supports HTML, you could use the a HTML tag.
✅  Do this 	❌  Don't do this
[link](https://www.example.com/my%20great%20page)

<a href="https://www.example.com/my great page">link</a>

	[link](https://www.example.com/my great page)

Parentheses in the middle of a URL can also be problematic. For compatibility, try to URL encode the opening parenthesis (() with %28 and the closing parenthesis ()) with %29. Alternatively, if your Markdown application supports HTML, you could use the a HTML tag.
✅  Do this 	❌  Don't do this
[a novel](https://en.wikipedia.org/wiki/The_Milagro_Beanfield_War_%28novel%29)

<a href="https://en.wikipedia.org/wiki/The_Milagro_Beanfield_War_(novel)">a novel</a>

	[a novel](https://en.wikipedia.org/wiki/The_Milagro_Beanfield_War_(novel))

## Images

To add an image, add an exclamation mark (!), followed by alt text in brackets, and the path or URL to the image asset in parentheses. You can optionally add a title in quotation marks after the path or URL.

![The San Juan Mountains are beautiful!](/assets/images/san-juan-mountains.jpg "San Juan Mountains")

The rendered output looks like this:

The San Juan Mountains are beautiful!
Note: To resize an image, see the section on image size. To add a caption, see the section on image captions.
Linking Images

To add a link to an image, enclose the Markdown for the image in brackets, and then add the link in parentheses.

[![An old rock in the desert](/assets/images/shiprock.jpg "Shiprock, New Mexico by Beau Rogers")](https://www.flickr.com/photos/beaurogers/31833779864/in/photolist-Qv3rFw-34mt9F-a9Cmfy-5Ha3Zi-9msKdv-o3hgjr-hWpUte-4WMsJ1-KUQ8N-deshUb-vssBD-6CQci6-8AFCiD-zsJWT-nNfsgB-dPDwZJ-bn9JGn-5HtSXY-6CUhAL-a4UTXB-ugPum-KUPSo-fBLNm-6CUmpy-4WMsc9-8a7D3T-83KJev-6CQ2bK-nNusHJ-a78rQH-nw3NvT-7aq2qf-8wwBso-3nNceh-ugSKP-4mh4kh-bbeeqH-a7biME-q3PtTf-brFpgb-cg38zw-bXMZc-nJPELD-f58Lmo-bXMYG-bz8AAi-bxNtNT-bXMYi-bXMY6-bXMYv)

The rendered output looks like this:
An old rock in the desert
Escaping Characters

To display a literal character that would otherwise be used to format text in a Markdown document, add a backslash (\) in front of the character.

\* Without the backslash, this would be a bullet in an unordered list.

The rendered output looks like this:

* Without the backslash, this would be a bullet in an unordered list.
Characters You Can Escape

You can use a backslash to escape the following characters.
Character 	Name
\ 	backslash
` 	backtick (see also escaping backticks in code)
* 	asterisk
_ 	underscore
{ } 	curly braces
[ ] 	brackets
< > 	angle brackets
( ) 	parentheses
# 	pound sign
+ 	plus sign
- 	minus sign (hyphen)
. 	dot
! 	exclamation mark
| 	pipe (see also escaping pipe in tables)
## HTML

Many Markdown applications allow you to use HTML tags in Markdown-formatted text. This is helpful if you prefer certain HTML tags to Markdown syntax. For example, some people find it easier to use HTML tags for images. Using HTML is also helpful when you need to change the attributes of an element, like specifying the color of text or changing the width of an image.

To use HTML, place the tags in the text of your Markdown-formatted file.

This **word** is bold. This <em>word</em> is italic.

The rendered output looks like this:

This word is bold. This word is italic.
HTML Best Practices

For security reasons, not all Markdown applications support HTML in Markdown documents. When in doubt, check your Markdown application’s documentation. Some applications support only a subset of HTML tags.

Use blank lines to separate block-level HTML elements like <div>, <table>, <pre>, and <p> from the surrounding content. Try not to indent the tags with tabs or spaces — that can interfere with the formatting.

You can't use Markdown syntax inside block-level HTML tags. For example, <p>italic and **bold**</p> won't work.

# Extended Syntax

## Overview

The basic syntax outlined in the original Markdown design document added many of the elements needed on a day-to-day basis, but it wasn’t enough for some people. That’s where extended syntax comes in.

Several individuals and organizations took it upon themselves to extend the basic syntax by adding additional elements like tables, code blocks, syntax highlighting, URL auto-linking, and footnotes. These elements can be enabled by using a lightweight markup language that builds upon the basic Markdown syntax, or by adding an extension to a compatible Markdown processor.
Availability

Not all Markdown applications support extended syntax elements. You’ll need to check whether or not the lightweight markup language your application is using supports the extended syntax elements you want to use. If it doesn’t, it may still be possible to enable extensions in your Markdown processor.
Lightweight Markup Languages

There are several lightweight markup languages that are supersets of Markdown. They include basic syntax and build upon it by adding additional elements like tables, code blocks, syntax highlighting, URL auto-linking, and footnotes. Many of the most popular Markdown applications use one of the following lightweight markup languages:

    CommonMark
    GitHub Flavored Markdown (GFM)
    Markdown Extra
    MultiMarkdown
    R Markdown

Markdown Processors

There are dozens of Markdown processors available. Many of them allow you to add extensions that enable extended syntax elements. Check your processor’s documentation for more information.
## Tables

To add a table, use three or more hyphens (---) to create each column's header, and use pipes (|) to separate each column. For compatibility, you should also add a pipe on either end of the row.

| Syntax      | Description |
| ----------- | ----------- |
| Header      | Title       |
| Paragraph   | Text        |

The rendered output looks like this:
Syntax 	Description
Header 	Title
Paragraph 	Text

Cell widths can vary, as shown below. The rendered output will look the same.

| Syntax | Description |
| --- | ----------- |
| Header | Title |
| Paragraph | Text |

Tip: Creating tables with hyphens and pipes can be tedious. To speed up the process, try using the Markdown Tables Generator or AnyWayData Markdown Export. Build a table using the graphical interface, and then copy the generated Markdown-formatted text into your file.
Alignment

You can align text in the columns to the left, right, or center by adding a colon (:) to the left, right, or on both side of the hyphens within the header row.

| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| Header      | Title       | Here's this   |
| Paragraph   | Text        | And more      |

The rendered output looks like this:
Syntax 	Description 	Test Text
Header 	Title 	Here’s this
Paragraph 	Text 	And more
Formatting Text in Tables

You can format the text within tables. For example, you can add links, code (words or phrases in backticks (`) only, not code blocks), and emphasis.

You can’t use headings, blockquotes, lists, horizontal rules, images, or most HTML tags.
Tip: You can use HTML to create line breaks and add lists within table cells.
Escaping Pipe Characters in Tables

You can display a pipe (|) character in a table by using its HTML character code (&#124;).
## Fenced Code Blocks

The basic Markdown syntax allows you to create code blocks by indenting lines by four spaces or one tab. If you find that inconvenient, try using fenced code blocks. Depending on your Markdown processor or editor, you’ll use three backticks (```) or three tildes (~~~) on the lines before and after the code block. The best part? You don’t have to indent any lines!

```
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```

The rendered output looks like this:

{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}

Tip: Need to display backticks inside a code block? See this section to learn how to escape them.
Syntax Highlighting

Many Markdown processors support syntax highlighting for fenced code blocks. This feature allows you to add color highlighting for whatever language your code was written in. To add syntax highlighting, specify a language next to the backticks before the fenced code block.

```json
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```

The rendered output looks like this:

{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}

## Footnotes

Footnotes allow you to add notes and references without cluttering the body of the document. When you create a footnote, a superscript number with a link appears where you added the footnote reference. Readers can click the link to jump to the content of the footnote at the bottom of the page.

To create a footnote reference, add a caret and an identifier inside brackets ([^1]). Identifiers can be numbers or words, but they can’t contain spaces or tabs. Identifiers only correlate the footnote reference with the footnote itself — in the output, footnotes are numbered sequentially.

Add the footnote using another caret and number inside brackets with a colon and text ([^1]: My footnote.). You don’t have to put footnotes at the end of the document. You can put them anywhere except inside other elements like lists, block quotes, and tables.

Here's a simple footnote,[^1] and here's a longer one.[^bignote]

[^1]: This is the first footnote.

[^bignote]: Here's one with multiple paragraphs and code.

    Indent paragraphs to include them in the footnote.

    `{ my code }`

    Add as many paragraphs as you like.

The rendered output looks like this:

Here’s a simple footnote,1 and here’s a longer one.2

    This is the first footnote. ↩

    Here’s one with multiple paragraphs and code.

    Indent paragraphs to include them in the footnote.

    { my code }

    Add as many paragraphs as you like. ↩

## Heading IDs

Many Markdown processors support custom IDs for headings — some Markdown processors automatically add them. Adding custom IDs allows you to link directly to headings and modify them with CSS. To add a custom heading ID, enclose the custom ID in curly braces on the same line as the heading.

### My Great Heading {#custom-id}

The HTML looks like this:

<h3 id="custom-id">My Great Heading</h3>

Linking to Heading IDs

You can link to headings with custom IDs in the file by creating a standard link with a number sign (#) followed by the custom heading ID. These are commonly referred to as anchor links.
Markdown 	HTML 	Rendered Output
[Heading IDs](#heading-ids) 	<a href="#heading-ids">Heading IDs</a> 	Heading IDs

Other websites can link to the heading by adding the custom heading ID to the full URL of the webpage (e.g, [Heading IDs](https://www.markdownguide.org/extended-syntax#heading-ids)).
## Definition Lists

Some Markdown processors allow you to create definition lists of terms and their corresponding definitions. To create a definition list, type the term on the first line. On the next line, type a colon followed by a space and the definition.

First Term
: This is the definition of the first term.

Second Term
: This is one definition of the second term.
: This is another definition of the second term.

The HTML looks like this:

<dl>
  <dt>First Term</dt>
  <dd>This is the definition of the first term.</dd>
  <dt>Second Term</dt>
  <dd>This is one definition of the second term. </dd>
  <dd>This is another definition of the second term.</dd>
</dl>

The rendered output looks like this:

First Term
    This is the definition of the first term.
Second Term
    This is one definition of the second term.
    This is another definition of the second term.

## Strikethrough

You can strikethrough words by putting a horizontal line through the center of them. The result looks like this. This feature allows you to indicate that certain words are a mistake not meant for inclusion in the document. To strikethrough words, use two tilde symbols (~~) before and after the words.

~~The world is flat.~~ We now know that the world is round.

The rendered output looks like this:

The world is flat. We now know that the world is round.
## Task Lists

Task lists (also referred to as checklists and todo lists) allow you to create a list of items with checkboxes. In Markdown applications that support task lists, checkboxes will be displayed next to the content. To create a task list, add dashes (-) and brackets with a space ([ ]) in front of task list items. To select a checkbox, add an x in between the brackets ([x]).

- [x] Write the press release
- [ ] Update the website
- [ ] Contact the media

The rendered output looks like this:

Markdown task list
## Emoji

There are two ways to add emoji to Markdown files: copy and paste the emoji into your Markdown-formatted text, or type emoji shortcodes.
Copying and Pasting Emoji

In most cases, you can simply copy an emoji from a source like Emojipedia and paste it into your document. Many Markdown applications will automatically display the emoji in the Markdown-formatted text. The HTML and PDF files you export from your Markdown application should display the emoji.
Tip: If you're using a static site generator, make sure you encode HTML pages as UTF-8.
Using Emoji Shortcodes

Some Markdown applications allow you to insert emoji by typing emoji shortcodes. These begin and end with a colon and include the name of an emoji.

Gone camping! :tent: Be back soon.

That is so funny! :joy:

The rendered output looks like this:

Gone camping! ⛺ Be back soon.

That is so funny! 😂
Note: You can use this list of emoji shortcodes, but keep in mind that emoji shortcodes vary from application to application. Refer to your Markdown application's documentation for more information.
Highlight

This isn’t common, but some Markdown processors allow you to highlight text. The result looks like this. To highlight words, use two equal signs (==) before and after the words.

I need to highlight these ==very important words==.

The rendered output looks like this:

I need to highlight these very important words.

Alternatively, if your Markdown application supports HTML, you can use the mark HTML tag.

I need to highlight these <mark>very important words</mark>.

Subscript

This isn’t common, but some Markdown processors allow you to use subscript to position one or more characters slightly below the normal line of type. To create a subscript, use one tilde symbol (~) before and after the characters.

H~2~O

The rendered output looks like this:

H2O
Tip: Be sure to test this in your Markdown application before using it. Some Markdown applications use one tilde symbol before and after words not for subscript, but for strikethrough.

Alternatively, if your Markdown application supports HTML, you can use the sub HTML tag.

H<sub>2</sub>O

Superscript

This isn’t common, but some Markdown processors allow you to use superscript to position one or more characters slightly above the normal line of type. To create a superscript, use one caret symbol (^) before and after the characters.

X^2^

The rendered output looks like this:

X2

Alternatively, if your Markdown application supports HTML, you can use the sup HTML tag.

X<sup>2</sup>

## Automatic URL Linking

Many Markdown processors automatically turn URLs into links. That means if you type http://www.example.com, your Markdown processor will automatically turn it into a link even though you haven’t used brackets.

http://www.example.com

The rendered output looks like this:

http://www.example.com
Disabling Automatic URL Linking

If you don’t want a URL to be automatically linked, you can remove the link by denoting the URL as code with backticks.

`http://www.example.com`

The rendered output looks like this:

http://www.example.com