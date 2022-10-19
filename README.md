# Obsidian to HTML converter

This is a short Python script to convert an [Obsidian](https://obsidian.md/) vault into a vault of HTML files, with the goal of publishing them as static files. It is heavily dependent on the excellent [markdown2](https://github.com/trentm/python-markdown2) by [trentm](https://github.com/trentm), but also deals with some parsing and file handling that makes it compatible with Obsidian's flavor of Markdown.

## Installation

Install `obsidian-html` by running:

    pip install git+https://github.com/vincentscode/obsidian-html.git

## Usage

`obsidian-html` will by default convert all the Markdown documents in the folder you're running it in, and place the HTML files in a directory called `html`. You might not want to run it directly in your vault or place the converted files in another directory. This is specified by this syntax:

    obsidian-html <path to vault> -o <path to html files>

The script will only convert the files located directly in the directory specified and never work recursively. To specify subfolders, these must be supplied to the `-d` flag, like in this example:

    obsidian-html <vault> -d "Daily notes" "Zettels"

### Templates

The output is not very exiting from the get-go. It needs some style and structure. This is done by using a HTML template. A template must have the formatters `{title}` and `{content}` present. Their value should be obvious. The template file is supplied to `obsidian-html` by the `-t` flag, like this:

    obsidian-html <vault> -t template.html

### TeX support via KaTeX

By loading KaTeX in the HTML template and initializing it with `$` and `$$` as delimiters, you will have TeX support on the exported documents.

<details>
<summary>Add this to the bottom of you template's body</summary>
<code>
<!-- KaTeX -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.11.1/dist/katex.min.css"
  integrity="sha384-zB1R0rpPzHqg7Kpt0Aljp8JPLqbXI3bhnPWROx27a9N0Ll6ZP/+DiW/UqRcLbRjq" crossorigin="anonymous">

<!-- The loading of KaTeX is deferred to speed up page rendering -->
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.11.1/dist/katex.min.js"
  integrity="sha384-y23I5Q6l+B6vatafAwxRu/0oK/79VlbSz7Q9aiSZUvyWYIYsd+qj+o24G5ZU2zJz"
  crossorigin="anonymous"></script>

<!-- To automatically render math in text elements, include the auto-render extension: -->
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.11.1/dist/contrib/auto-render.min.js"
  integrity="sha384-kWPLUVMOks5AQFrykwIup5lo0m3iMkkHrD0uJ4H5cjeGihAutqP0yW0J6dpFiVkI"
  crossorigin="anonymous"></script>

<!-- Parsing single dollar signs -->
<script>
  document.addEventListener("DOMContentLoaded", function () {{
      renderMathInElement(document.body, {{
        delimiters: [
          {{left: "$$", right: "$$", display: true}},
        {{left: "\\[", right: "\\]", display: true}},
    {{left: "$", right: "$", display: false}},
    {{left: "\\(", right: "\\)", display: false}}
      ]
  }});
  }});
</script>
</code>
</details>

### Syntax highlighting of code blocks

Using [highlight.js](https://highlightjs.org/), syntax highlighting is easily achieved.


<details>
<summary>Just add this to the bottom of you template's body</summary>
<code>
<!-- Syntax highlighting through highlight.js -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/highlight.js@10.1.2/styles/github-gist.css">
<script src="//cdn.jsdelivr.net/gh/highlightjs/cdn-release@10.1.2/build/highlight.min.js"></script>

<script>hljs.initHighlightingOnLoad();</script>
</code>
</details>

## Deploying vault with GitHub Actions

Make a GitHub Actions workflow using the YAML below, and your vault will be published to GitHub Pages every time you push to the repository.

1. Make sure you have GitHub Pages set up in the vault.
2. Modify the following YAML job to match your repository.

    ```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ master ]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true


jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2

    - name: Set up Python 3.8
      uses: actions/setup-python@v2
      with:
        python-version: 3.8

    - name: Install obsidian-html
      run: |
        python -m pip install --upgrade pip
        pip install git+https://github.com/vincentscode/obsidian-html.git
      
    - name: Generate HTML through obsidian-html
      run: obsidian-html /home/runner/work/obsidian-vault/obsidian-vault -o ./_site

    - name: Upload artifact
      uses: actions/upload-pages-artifact@v1
 
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v1 

    ```

## To do

- [ ] Support sub-dirs and recursing
- [ ] Support local attachments
- [ ] Support the `![[]]` embedding syntax (perhaps using iframe or some similar method)
- [ ] Support extra features added by the user through YAML metadata

## Known issues

- Links in headers lead to weird header ids, and thus malfunctioning header links from other pages.
