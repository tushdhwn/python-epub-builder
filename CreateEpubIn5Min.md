# Introduction #

Being used here is the [simpler API](http://code.google.com/p/python-epub-builder/source/browse/trunk/ez_epub.py), which is considerably easier than the [full API](http://code.google.com/p/python-epub-builder/source/browse/trunk/epub.py). You can still do a lot of things with it, such as multiple levels of chapter/section/subsection, adding a book cover, and CSS styling.
# Details #

Gutenburg's books are poorly structured; the transcribers use whatever text formatting rules they want. The results are books whose text cannot be reflown to small mobile device screens, and cannot be easily consumed by computer processing systems. It is unbelievable that the project creators were so short-sighted to not define and enforce a book text formatting standard (one like wiki markup language or LaTeX would suffice). In the end programmers need to suffer to clean up this mess if they want to do anything.

Enough for the complaints. [Pride & Prejudice](http://www.gutenberg.org/etext/1342) is actually not too bad. There is no indentation, and paragraphs are separated by empty lines, so I can write a short parser under 5 minutes. Each chapter (identified by a simple regular expression) corresponds to a 'section', and the paragraphs inside it are kept as a list. To detect the start and end of the book, I'll need line numbers to be manually specified. Not intelligent huh? What can you expect out of a 5-minute code work?

```
def parseBook(path, startLineNum, endLineNum):
    PATTERN = re.compile(r'Chapter \d+$')
    sections = []
    paragraph = ''
    fin = open(path)
    lineNum = 0
    for line in fin:
        lineNum += 1
        if lineNum < startLineNum:
            continue
        if endLineNum > 0 and lineNum > endLineNum:
            break
        line = line.strip()
        if PATTERN.match(line):
            section = ez_epub.Section()
            section.title = line
            sections.append(section)
        elif line == '':
            if paragraph != '':
                section.text.append(paragraph)
                paragraph = ''
        else:
            if paragraph != '':
                paragraph += ' '
            paragraph += line
    if paragraph != '':
        section.text.append(paragraph)
    return sections
```

Remaining is easy:
```
    book = ez_epub.Book()
    book.title = 'Pride and Prejudice'
    book.authors = ['Jane Austen']
    book.sections = parseBook(r'D:\epub\1342.txt', 38, 13061)
    book.make(r'D:\epub\%s' % book.title)
```

End result can be downloaded [here](http://code.google.com/p/python-epub-builder/downloads/detail?name=Pride+and+Prejudice.epub). So you see, making EPUB is itself quite trivial, and the difficult part is to write a parser to work with unstructured text. I'd challenge you to convert this Gutenburg book [The Art of War](http://www.gutenberg.org/etext/132) to EPUB. I'd rather buy from Amazon's Kindle store(note) than to spend long hours to write heuristics for making sense out of the unruly text.

In the above simplified example, each paragraph is just a string. If you want to offer more styling (e.g. to handle the `_`this\_is\_italic`_` convention in Gutenburg books), you can make paragraph a list of (string segment, css class name). For details, see the more elaborated source code [here](https://code.google.com/p/python-epub-builder/source/browse/trunk/ez_epub_example.py)

(note) Kindle does not support EPUB, but you can use [Calibre](http://calibre-ebook.com/) to convert EPUB to MOBI, which is compatible to Kindle (do you know MobiPocket was acquired by Amazon?)