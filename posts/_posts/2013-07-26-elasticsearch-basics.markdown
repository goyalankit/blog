---
layout: post
title: "Elasticsearch basics - Analyzers"
date: 2013-07-24 14:18
comments: true
published: true
categories: elasticsearch tutorials analyzers
---

Elasticsearch is a powerful open source search engine build over Apache
Lucene. You can do all kind of customized searches on huge amount of
data by creating customized indexes. This post gives an overview of
Analysis module of elasticsearch.

Analyzers basically helps you in analyzing your data.`:o` You need to analyze data while creating indexes and while searching. You could analyze your analyzers using [Analyze Api](//www.elasticsearch.org/guide/reference/api/admin-indices-analyze.html) provided by elasticsearch.

Creating indexes mainly involves three steps: 

* **Pre-processing of raw text** using [char filters]( //www.elasticsearch.org/guide/reference/index-modules/analysis/mapping-charfilter/). This may be used to strip html tags, or you may define your custom mapping.  (*Couldn't find a way to test this using analyse api. Please put it in comments if you know some way to test these through Analyze Api*) 


Example: You could use a **char-filter** of type [`html_strip`](//www.elasticsearch.org/guide/reference/index-modules/analysis/htmlstrip-charfilter/) to strip out html tags.  

A text like this:

{% highlight html %}
<p> Learn Something New Today! which is <b>always</b> fun </p>
{% endhighlight %}

would get converted to:


{% highlight html %}
Learn Something New Today! which is always fun
{% endhighlight %}
<!-- more -->

* **Tokenization of the pre-processed text** using tokenizers. Tokenizers breaks the pre-processed text into tokens. There are different kind of tokenizers available and each of them breaks the text into words differently. By default elasticsearch uses [standard tokenizer](//www.elasticsearch.org/guide/reference/index-modules/analysis/standard-tokenizer/). 


standard tokenizer normalizes the data. Note that it removes `!` from `Today!`

A pre-processed text like this:

`Learn Something New Today! which is always fun`

gets broken as

`Learn` `Something` `New` `Today` `which` `is` `always` `fun`

You could check for yourself using Analyze Api mentioned above.

{% highlight sh %}
curl -XGET 'localhost:9200/_analyze?tokenizer=standard' \
    -d 'Learn Something New Today! which is always fun'
{% endhighlight %}

{% highlight javascript %}
{
    "tokens": [
        {
            "end_offset": 5,
            "position": 1,
            "start_offset": 0,
            "token": "Learn",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 15,
            "position": 2,
            "start_offset": 6,
            "token": "Something",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 19,
            "position": 3,
            "start_offset": 16,
            "token": "New",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 25,
            "position": 4,
            "start_offset": 20,
            "token": "Today",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 32,
            "position": 5,
            "start_offset": 27,
            "token": "which",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 35,
            "position": 6,
            "start_offset": 33,
            "token": "is",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 42,
            "position": 7,
            "start_offset": 36,
            "token": "always",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 46,
            "position": 8,
            "start_offset": 43,
            "token": "fun",
            "type": "<ALPHANUM>"
        }
    ]
}
{% endhighlight %}

* After the tokenization, **token filters** performs further operations on the processed text like converting it to [lowercase](//www.elasticsearch.org/guide/reference/index-modules/analysis/lowercase-tokenfilter.html) or [reversing](//www.elasticsearch.org/guide/reference/index-modules/analysis/reverse-tokenfilter/) of tokens.

By default [standard tokenfilter](//www.elasticsearch.org/guide/reference/index-modules/analysis/standard-tokenfilter/) is used which normalizes the tokens. After the application of lowercase tokenfilter.

A processed text like this:

`Learn` `Something` `New` `Today` `which` `is` `always` `fun`

gets broken as

`learn` `something` `new` `today` `which` `is` `always` `fun`


{% highlight sh %}
## Analyze Api
curl -XGET 'localhost:9200/_analyze?tokenizer=standard&filters=lowercase' \
    -d 'Learn Something New Today! which is always fun'
{% endhighlight %}


{% highlight javascript %}
{
    "tokens": [
        {
            "end_offset": 5,
            "position": 1,
            "start_offset": 0,
            "token": "learn",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 15,
            "position": 2,
            "start_offset": 6,
            "token": "something",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 19,
            "position": 3,
            "start_offset": 16,
            "token": "new",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 25,
            "position": 4,
            "start_offset": 20,
            "token": "today",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 32,
            "position": 5,
            "start_offset": 27,
            "token": "which",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 35,
            "position": 6,
            "start_offset": 33,
            "token": "is",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 42,
            "position": 7,
            "start_offset": 36,
            "token": "always",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 46,
            "position": 8,
            "start_offset": 43,
            "token": "fun",
            "type": "<ALPHANUM>"
        }
    ]
}
{% endhighlight %}

Thus analyzer is composed of char-filters, tokenizers and tokenfilters. Analyzers defines what kind of search you can preform on your data.

You can have multiple indexes on a field and create your own custom char-filters, tokenizers and tokenfilters. You can have different analyzers for different indexes.

## Let's see it in action

Example below creates an index with **char-filter** as `html_strip`, **tokenizer** as `standard` and tokenfilter i.e, **filter** as `lowercase` and `standard`

{% highlight sh %}
curl -XPUT http://localhost:9200/test -d \
    {'
        "settings":{
            "analysis":{
                "analyzer":{
                    "default":{
                         "type":"custom",
                         "tokenizer":"standard",
                         "filter":["standard", "lowercase"],
                         "char_filter" : ["html_strip"]
                     }
                }
            }
        }
  }'
{% endhighlight %}

You can analyze the text using:
{% highlight sh %}
curl 'http://localhost:9200/test/_analyze' -d \
    '<p> Learn Something New Today! which is <b>always</b> fun </p>'
{% endhighlight %}

{% highlight javascript %}
{
    "tokens": [
        {
            "end_offset": 9,
            "position": 1,
            "start_offset": 4,
            "token": "learn",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 19,
            "position": 2,
            "start_offset": 10,
            "token": "something",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 23,
            "position": 3,
            "start_offset": 20,
            "token": "new",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 29,
            "position": 4,
            "start_offset": 24,
            "token": "today",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 36,
            "position": 5,
            "start_offset": 31,
            "token": "which",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 39,
            "position": 6,
            "start_offset": 37,
            "token": "is",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 53,
            "position": 7,
            "start_offset": 43,
            "token": "always",
            "type": "<ALPHANUM>"
        },
        {
            "end_offset": 57,
            "position": 8,
            "start_offset": 54,
            "token": "fun",
            "type": "<ALPHANUM>"
        }
    ]
}
{% endhighlight %}

Above results shows that the while creating index it first stripped off the html tags and broke the text into words. And then converted them to lowercase.

Following the same procedure you can analyze different kind of
analyzers. Explore different kind of tokenizers, tokenfilters at //www.elasticsearch.org/guide/reference/index-modules/analysis/

In future posts I will discuss more about how to make custom analyzers and features of elasticsearch like filters and facets.
