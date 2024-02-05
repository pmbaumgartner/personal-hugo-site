---
title: "Reporting With Quarto"
date: 2024-02-05T07:51:34-05:00
draft: false
---

<meta property="og:title" content="Reporting With Quarto" />
<meta property="og:description" content="Effective Evaluation and Reporting with Quarto in Python" />
<meta property="og:type" content="website" />
<meta property="og:url" content="https://peterbaumgartner.com/blog/reporting-with-quarto/" />
<meta property="og:image" content="https://i.postimg.cc/XvnnBCny/DALL-E-2024-02-05-07-55-58-Visualize-a-cartoonish-friendly-robot-seated-at-a-colorful-desk-surro.webp" />


Recently I've been using Quarto to generate HTML reports to share with stakeholders and I'd like to share a workflow and configuration that has worked for me.

## Background
Most of my programming I do in VSCode. A typical workflow involves  using an IPython REPL for exploratory coding, organizing commonly used code into a package, and creating CLI functions with `typer` as an abstraction for common tasks. This approach solves 90% of my problems on data science projects and gives me a replicable workflow that's easily modified. However, when it comes to reporting for things like model evaluation, I didn't have a satisfying workflow (until now). Reporting usually involves lots of tables, visualizations, examples, and narrative that need to be connected in some way. Previously I've tried to shoehorn these various components into my workflow typically by creating single-use scripts for intermediate data processing and outputting images of visualizations, then attempting some hacky solution for putting them in a markdown document. It wasn't really working for me, but then I discovered Quarto, and it has become my go-to solution for reporting evaluations.

## Enter Quarto
[Quarto](https://quarto.org/) is *an open source technical publishing system for creating beautiful articles, websites, blogs, books, slides, and more.* 

With my workflow I use Quarto (and the [VSCode Extension](https://marketplace.visualstudio.com/items?itemName=quarto.quarto)) to render notebooks I have authored within VSCode to HTML. The notebook workflow gives me a place to draft code I need to get the data in the right structure for any reporting as well as explore various visualization options. In my reporting notebooks, I use notebook cells to organize code into three buckets: narrative (in markdown), data processing, and tables/visualization. The tables/visualizations containing rich output gets displayed within the quarto document along with the narrative. 

I typically spend a fair amount of time exploring and developing code for the tables and visualizations, which is why having this in a notebook environment is useful. I can create temporary cells to inspect intermediate data, I can compare various visualization options, and I can quickly debug any issues that exist in getting data from raw to rendered. Additionally, I am often rendering and re-executing the notebook to the final HTML to catch any issues in how I have the information structured or presented. This step is key to developing a "final" document. This also helps prevent against the typical cell execution order pitfalls of notebooks be re-executing the full notebook from time-to-time to verify the output. Executing the notebook upon rendering is not a Quarto default, so be sure you render with the [`--execute` flag](https://quarto.org/docs/projects/code-execution.html#notebooks) or change the relevant option in the front-matter.

Speaking of [front-matter](https://quarto.org/docs/computations/execution-options.html), all you need to do to enable Quarto to run on a notebook is include some YAML as the first cell and declare the cell type as `Raw`. The front-matter is where I change the default settings for a Quarto document in a few ways. The most important thing is to set the `include` option to `false` so that nothing is output to the rendered document from any cell. This makes it so that without explicitly changing these settings in any specific cell, we are in a state where rendering a notebook as a document will not output anything from cells with code in them. I do this because most of my cells are data processing and typically irrelevant to a reader of the document - only the narratives and visuals are relevant. A typical [front-matter](https://quarto.org/docs/computations/execution-options.html) for me looks like this:

```yaml
title: "Model Evaluation"

abstract: |
  We evaluated 3 different models using the [Palmer penguins](https://allisonhorst.github.io/palmerpenguins/articles/intro.html) dataset.

author:
  - name: Peter Baumgartner

format:
  html:
    toc: true
    embed-resources: true
    minimal: true
theme: default
include: false
```

Some other relevant [options](https://quarto.org/docs/output-formats/html-basics.html) in the `format.html` section:
- `toc: true` - This displays a table of contents which is automatically generated from your markdown headers. Outside of making it easy to navigate for a reader, I find including this also helps me think about how to better structure my document.
- `embed-resources: true` - I will often send the rendered HTML document to stakeholders - this ensures that any CSS and JS resources Quarto needs are included in that HTML file.
- `minimal: true` - Since I'm often sending the entire HTML file I only use the minimal set of features so that I'm not sending a giant HTML file.

With this front-matter structure, I write any data processing or calculation cells as normal. Then, for any tabular output cells, I include the following cell metadata:

```txt
# | include: true
# | echo: false
# | label: tbl-results
# | tbl-cap: Model Evaluation Results
```

And if a visualization or figure:

```txt
# | include: true
# | echo: false
# | label: fig-results-comparision
# | fig-cap: Model Evaluation Comparision
```

In common across both cell types is setting `include: true` and `echo: false`[^1]. The `include` setting overrides my front-matter setting and displays the cell. Without `echo: false`, the code used to generate the visualization would also be displayed. Since I am delivering reports to a lay audience, and the relevant code is only for the visualization, I do not want to display the code to the reader.

Also in common is the `label`. With Quarto, if you prefix a [tabular](https://quarto.org/docs/authoring/tables.html#cross-references) output with `tbl-` or a [figure](https://quarto.org/docs/authoring/figures.html) with `fig-`, it will do three important things: automatically number that table, allow you to cross reference that table in markdown, and apply table or figure specific styling. 

Note that you can still display cell output even if it's not a table or figure, these are simply the most common for my workflow. Occasionally I will output computed markdown or any `print` statements I want visible as well.

## Project Integration

I really like this workflow for reporting by itself, but there is some great synergy if you are writing the rest of your code as a python package. Because the rest of my code code is written as a package, I can import commonly referenced project-specific data, objects, and functions into my notebook and use them. A simple example with supervised learning be a mapping of label ID to description (`{0: "yes", 1: "no"}`) that can be used for table or figure labels. If I have a few examples I want to run through a predictive model and generate predictions to display within the report, I'm using the same inference API in my notebook as I would be "in production" - I don't need to rewrite any functions to load models or do inference. 

Sometimes this reporting workflow generates new feature ideas to include in my API as well. Recently I had a multiclass supervised learning problem where we wanted to start reporting how many predictions were above a certain threshold - for example `>0.9`. Originally our model was outputting only the class label, but now we had a need to update the API from our model to output the probability as well. This is a feature that is useful outside of this specific reporting, so we included it with our model API, added some tests, and now it is part of our inference API with these models.

## More Complex Quarto Reporting

Rendering single notebooks works great for single documents I want to create and distribute. However, I often have a more complex use case where I want to repeat a set of experiments or evaluations across datasets in a modular way and including all this information in one file would be too long. For this use case, I create a [Quarto Website](https://quarto.org/docs/websites/). I use the website to organize information by experiment or dataset with the navigation elements. 

The configuration for a website is managed in a `_quarto.yml` file within a website folder. Within this file you define how you want the navigation to work by linking to the source notebooks or markdown documents you want to render and include in the site. Often when creating a site, I will have pages that don't have computational elements and don't need to be a notebook - in this case, they can simply be Quarto Markdown (`.qmd`) files. 

Creating a website means that I have to find a place to host the pages, since I now have more than one report I want to distribute. I will typically do this with GitHub Pages, but any hosting platform will work. In most cases, I integrate the rendering process with CI/CD at this point using something like GitHub Actions so that I know the website always contains the most recent updates.

## Conclusion
In summary, adopting Quarto for HTML report generation has significantly streamlined the reporting for me across projects. By integrating Quarto into my existing VSCode and Python package-based environment, I've been able to address challenges I had around developing reporting and evaluation narrative and code. I also appreciate that I get a clear and accessible formatted HTML document I can deliver to stakeholders. If you're facing the same issues in your projects, I encourage you to give Quarto a try.


[^1]: I could have also included `echo: false` in my front-matter so that I'm not re-writing it every cell, but there are sometimes occasions I do want to display the code so I am usually explicit in declaring this per-cell.
