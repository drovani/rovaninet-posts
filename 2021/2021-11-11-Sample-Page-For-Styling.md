---
title: Sample Page For Styling
category: Rovani in C♯
date: 2021-11-11
tags:
- writing
- sample
- experiment
---

This page is serving as a sample of all the various layouts and formattings that I put into a markdown file. The initial draft was of situations that I could think of off the top of my head, then additional scenarios were added as I found them and need to spot-check out they are formatted. Now that this opening paragraph has enough words in it to wrap a few times, let's get onto the other stuff.

## Only The Title Is An H1; This Is a Long H2 Sub Heading

"h2 + p" - This is the first paragraph under the first subheading. There should be space between the h2 title and this paragraph. This paragraph has enough text to wrap a few lines, just to make sure there is some content visibile.

### This H3 is a section header

"h3 + p" - A section like this is typically going to have two or more paragraphs or a list of items. It doesn't make sense to have real heading below this since the text would already start being fairly fragmented at this point.

#### An H4 should be inline.

In order to have the first line of a paragraph really stand out, and where it makes sense to have it be something pulled into an outline, then an H4 would be appropriate. It would need to follow the rules of a title in that it is short and preps the reader for the content; but follows capitalization rules for a sentence, in that only the first word, proper names, and such are in capitals; and it should end with a period.

Only sometimes does a paragraph with an H4 header then have a sibling paragraph, but when it does, I want to make sure th e

## H2 Sub Heading Act As a Hard Break In Subject

Now that all of the headings have been explored, let's take a look at lists. There is usually a text introduction to what the list is going to be filled with, here is the first unordered list.

- "ul + li" for this first item, or I guess it could be "ul li:first-child"
- "ul li" will target this second, and all items in the unordered list
- "ul li:last-child" could specifically target this line item

When I want to have longer lists with two levels, this is what it would look like.

- "ul + li" for this first item, or I guess it could be "ul li:first-child"
- "ul li" will target this second, and all items in the unordered list
  - Sometimes, a list will have a second level of items
  - This is what a nested unordered list would look like.
- "ul li:last-child" could specifically target this line item

Usually, lists have some text between them, instead of just attempting to put a visual break between lists with empty space. This is what an ordered list looks like.

1. "ol li:first-child" targets this first item.
1. "ol li" for all of the other list items.
1. "ol li:last-child" targets this last item.

## Commonly Used Blocks

Sometimes, I'll use `code breaks` to indicate objects that should be `monospace formatted` to indicate that it represents a different piece of work.

```typescript {9-10}
export default class SampleCodeBlock {
    public Line2: string;
    public Line3: int;

    constructor(public Line5: string,
        private line6: int[]
    ){
        "This string is on Line 8";
        const line9 = "should be highlighted".
        var line10 = new ShouldBeHighlighted();
    }
}
```

### Blockquotes

> There once was a man from Nantucket  
> Who kept all his cash in a bucket.  
>   But his daughter, named Nan,  
>   Ran away with a man  
> And as for the bucket, Nantucket.

<figcaption>Prof. Dayton Voorhees. (1902) <cite>Princeton Tiger</cite></figcaption>

For copy/paste purposes, here is an `&mdash;` &mdash;, an `&ndash;` &ndash;, and hyphen -. The above citation is manually assembled, using a `<figcaption>` element and the name of the publication surrounded with a `<cite>` element. It would be nice if there was a markdown standard punctuation for that, but there isn't. Side-note, did you know there is a `<q>` element for inline quotes? I just found out about that.

## Images Typically Have Two Locations

The first one is where it's a large image that should be styled as a block, where the text is above and below it.

![Draw an Owl](/images/draw-an-owl.png)

There there are images I would like inline, like this one ![this one](/images/azure-storage-account-icon.png) and this one ![another one](/images/azure-resource-group-icon.png) because they are small icons that add to the paragraph text.

- I even use inline images ![this one](/images/azure-storage-account-icon.png) in lists
- So I need to have something to look for them ![another one](/images/azure-resource-group-icon.png)