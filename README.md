# jflow jQuery plugin

jflow is a jQuery plugin which flows continuous html into a sequence of fixed size boxes within pages.

## why?

Letters, mailing labels, newspaper-like columns, duplex (double-sided printing), alternating left and right pages, pages with custom headers and footers...

The conventional html model doesn't flow text from one box into another in the manner of products like Adobe InDesign, Quark XPress or Microsoft Publisher. Instead it relies on boxes expanding to make room on one continuous page. This is fine for web pages, but if you want to end up with something to print it limits you to one block of text on each page controlled just by the margins.

Of course, if you don't start with html, you could make a PDF (with TCDPF for example). But a big advantage of using HTML as the input is you can deploy products like tinymce, Google Docs or markdown (with a md to html converter) for the end user to be able to edit fragments. If your input is HTML, it's then really hard to flow from box to box in arbitrary positions on a page because you don't know where the breaks should be, and the task quickly becomes on of implementing an entire HTML+CSS interpreter. But your user already has one, so we can use that instead.

## how?

There are some **examples** in the jflow directory. You can just open these examples in your browser: no need to have them on a web server.

Include jquery, and jflow.js from the jflow directory, for example:

    <script src='/jquery-2.1.4.min.js'></script>
    <script src='/jflow-0.1.1/jflow.js'></script>

The plugin is invoked like this

        $(pages).flow(contents, options)

but there are some assumptions about the document structure.

**pages** is a collection of elements representing pages into which content will be flowed. Each page contains (potentially amongst other things, such as a background letterhead) a set of boxes to take the content. Those boxes are identified by the selector ".box" by default, or whatever you provide as the box option (see below) - so your boxes typically have the class "box". They must be styled as overflow: hidden, so they are fixed size boxes which don't grow with the addition of content. Content is flowed into each of the boxes of the first page, then of the next page and so on. When it runs out of pages, it repeats the last page. So if there is just one page, it will use the same one repeatedly, but having two would let you have a front page and repeated continuation pages (for a letterhead, for example). (Possible future option: more control over groups of page repetition).

**contents** is a selector, set of elements, or jQuery collection identifying the contents to be flowed. Each element selected is treated as a separate piece of content which starts at the first box of the first page (except when **pagination** is *repeat* - see below - when it starts in the next available box, for things like many-up mailing labels).

**options** is an optional object which lets you change settings as follows:

| key | default  | meaning |
|:--- |:------- |:---------|
| box | ".box" | selector for boxes to flow content into
| pagination | "simplex" | Either **"simplex"**: start each new content on a new first page  following on from the previous page, typically for single side printing, or... **"duplex"**: start each content on a new  first page, but insert a blank page in between to make the first page be an odd-numbered page, typically for double-sided (duplex) printing, or... **"repeat"**: start each content in next available box, for example to produce mailing labels many-up on the same sheet |
| keeptogether | "keep-together" | Contents of elements with this class name are treated as a whole, moved to the next box if the whole element does not fit where it is. Beware that if no box can accommodate the element, you can get into an infinite loop |
| advancebefore | "advance-before" | Always move elements with this class to a new box (unless it is already at the start of a box) |
| pagenumber | "page-number" | The content of elements with this class (whether in the content or pages) is substituted with the page number. This will typically be a span, and no reflow happens because of this so you should put include dummy filler content (like '9' or '99'). |
| repeatfrom | "repeat-from" | When you run out of pages, repeat from the page with this class, rather than the last page (useful for alternating right and left pages, for example). |
| fillerpage | "filler-page" | Use the page labelled with this class name as the filler for duplex (so you can have background graphics on those as well if you want, or "this page intentionally left blank"). The filler page must be among the original pages but is not treated as one of the repeating pages (typically you'll put it at the end) |
| classprefix | "jf_" | various class names are added to elements; they all start with this prefix. You only need to change this if there is a clash with your own class names |

The **flow** method returns a jQuery collection comprising all the now populated pages. No content or pages remain in their original position in the document.

## typical use

See the example for more information.

Typically:

* your page elements will be divs defined with a physical size in css, e.g.

        width: 210mm;
        height: 297mm;

* your boxes within those pages will be div with class "box", position: absolute and overflow: hidden, with top, left, width and height positioning them within the page. Because absolutely positioned boxes don't collapse their vertically adjacent margins, you'll probably also need a css rule to avoid a space at the top of boxes:

        .box *:first-child { margin-top: 0; }

    you'll want a CSS rule to define printed pages, like this:

        @page {margin: 0; size: A4 portrait; }

    Don't use id's on your pages or descendants or contents or descendants, as they need to be duplicated.

* and then your html will be structured:

        <div class="page p1">
            <-- any background content -->
            <div class="box box1-1" style="top: ...mm, left: ...mm, width: ...mm, height: ...mmm"></div>
            <div class="box box1-2" style="top: ...mm, left: ...mm, width: ...mm, height: ...mmm"></div>
        </div>
        <div class="page p2">
            <-- any background content -->
            <div class="box box2-1" style="top: ...mm, left: ...mm, width: ...mm, height: ...mmm"></div>
            <div class="box box2-2" style="top: ...mm, left: ...mm, width: ...mm, height: ...mmm"></div>
        </div>
        <!-- etc... -->
        <div class="content letter1">
            <p>...</p>
            <!-- etc... -->
        </div>
        <div class="content letter2">
            <p>...</p>
            <!-- etc... -->
        </div>
        <!-- etc... -->

* and your jQuery (in head or after body):

        <script>
            $(function(){
                $(".page").flow(".content");
            });
        </script>

* Because jflow depends on measuring heights of things to see if they will fit in boxes, if you don't have images with pre-determined heights in the img tags, you'll need to wait for them to load before flowing:

        <script>
            $(window).on("load", function(){
                $(".page").flow(".content");
            });
        </script>
