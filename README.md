# gatsby-transformer-remark-rehype

Parses Markdown files using [Remark](http://remark.js.org/) & Rehype.

This package is a mod of [gatsby-transformer-remark](https://www.npmjs.com/package/gatsby-transformer-remark)

## Install

`npm install gatsby-transformer-remark`

## How to use

```javascript
// In your gatsby-config.js
plugins: [
  {
    resolve: `gatsby-transformer-remark`,
    options: {
      // CommonMark mode (default: true)
      commonmark: true,
      // Footnotes mode (default: true)
      footnotes: true,
      // Pedantic mode (default: true)
      pedantic: true,
      // GitHub Flavored Markdown mode (default: true)
      gfm: true,
      // Gatsby flavor plugins configs
      plugins: [],
      // Remark plugins
      remarkPlugins: [],
      // Rehype plugins
      rehypePlugins: []
    },
  },
],
```

The following parts of `options` are passed down to Remark as options:

- `options.commonmark`
- `options.footnotes`
- `options.pedantic`
- `options.gfm`

The details of the Remark options above could be found in [`remark-parse`'s documentation](https://github.com/remarkjs/remark/tree/main/packages/remark-parse#processoruseparse-options)

A full explanation of how to use markdown in Gatsby can be found here:
[Creating a Blog with Gatsby](https://www.gatsbyjs.org/blog/2017-07-19-creating-a-blog-with-gatsby/)

There are many Gatsby Remark plugins which you can install to customize how Markdown is processed. Many of them are demoed at https://using-remark.gatsbyjs.org/. See also the [source code for using-remark](https://github.com/gatsbyjs/gatsby/tree/master/examples/using-remark).

## Parsing algorithm

It recognizes files with the following extensions as Markdown:

- md
- markdown

Each Markdown file is parsed into a node of type `MarkdownRemark`.

All frontmatter fields are converted into GraphQL fields. TODO link to docs on
auto-inferring types/fields.

This plugin adds additional fields to the `MarkdownRemark` GraphQL type
including `html`, `excerpt`, `headings`, etc. Other Gatsby plugins can also add
additional fields.

## How to query

A sample GraphQL query to get MarkdownRemark nodes:

```graphql
{
  allMarkdownRemark {
    edges {
      node {
        html
        headings {
          depth
          value
        }
        frontmatter {
          # Assumes you're using title in your frontmatter.
          title
        }
      }
    }
  }
}
```

### Getting table of contents

// todo
### Excerpts

#### Length

By default, excerpts have a maximum length of 140 characters. You can change the default using the `pruneLength` argument. For example, if you need 500 characters, you can specify:

```graphql
{
  allMarkdownRemark {
    edges {
      node {
        html
        excerpt(pruneLength: 500)
      }
    }
  }
}
```

#### Format

By default, Gatsby will return excerpts as plain text. This might be useful for populating [opengraph](https://en.wikipedia.org/wiki/Facebook_Platform#Open_Graph_protocol) HTML tags for SEO reasons. You can also explicitly specify a `PLAIN` format like so:

```graphql
{
  allMarkdownRemark {
    edges {
      node {
        excerpt(format: PLAIN)
      }
    }
  }
}
```

It's also possible to ask Gatsby to return excerpts formatted as HTML. You might use this if you have a blog post whose excerpt contains markdown content--e.g. header, link, etc.--and you want these links to render as HTML.

```graphql
{
  allMarkdownRemark {
    edges {
      node {
        excerpt(format: HTML)
      }
    }
  }
}
```

You can also get excerpts in Markdown format.

```graphql
{
  allMarkdownRemark {
    edges {
      node {
        excerpt(format: MARKDOWN)
      }
    }
  }
}
```

## gray-matter options

`gatsby-transformer-remark` uses [gray-matter](https://github.com/jonschlinkert/gray-matter) to parse markdown frontmatter, so you can specify any of the options mentioned [here](https://github.com/jonschlinkert/gray-matter#options) in the `gatsby-config.js` file.

### Example: Excerpts

If you don't want to use `pruneLength` for excerpts but a custom separator, you can specify an `excerpt_separator` in the `gatsby-config.js` file:

```javascript
{
  "resolve": `gatsby-transformer-remark`,
  "options": {
    "excerpt_separator": `<!-- end -->`
  }
}
```

Any file that does not have the given `excerpt_separator` will fall back to the default pruning method.

## Troubleshooting

### Excerpts for non-latin languages

By default, `excerpt` uses `underscore.string/prune` which doesn't handle non-latin characters ([https://github.com/epeli/underscore.string/issues/418](https://github.com/epeli/underscore.string/issues/418)).

If that is the case, you can set `truncate` option on `excerpt` field, like:

```graphql
{
  markdownRemark {
    excerpt(truncate: true)
  }
}
```

### Excerpts for HTML embedded in Markdown files

If your Markdown file contains HTML, `excerpt` will not return a value.

In that case, you can set an `excerpt_separator` in the `gatsby-config.js` file:

```javascript
{
  "resolve": `gatsby-transformer-remark`,
  "options": {
    "excerpt_separator": `<!-- endexcerpt -->`
  }
}
```

Edit your Markdown files to include that HTML tag after the text you'd like to appear in the excerpt:

```markdown
---
title: "my little pony"
date: "2017-09-18T23:19:51.246Z"
---

<p>Where oh where is that pony?</p>
<!-- endexcerpt -->
<p>Is he in the stable or down by the stream?</p>
```

Then specify `MARKDOWN` as the format in your GraphQL query:

```graphql
{
  markdownRemark {
    excerpt(format: MARKDOWN)
  }
}
```
