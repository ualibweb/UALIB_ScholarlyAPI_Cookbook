---
title: Technical Details
---

::: sectionauthor
Vincent F. Scalfani \<<vfscalfani@ua.edu>\>
:::

# Technology and Software Used

1.  Python tutorial and Mathematica content ([with Wolfram
    Kernel](https://github.com/WolframResearch/WolframLanguageForJupyter))
    is written in [Jupyter Notebooks](https://jupyter.org/)
2.  All other content is written in
    [reStructuredText](https://www.sphinx-doc.org/en/master/usage/restructuredtext/index.html).
3.  Code testing is done locally.
4.  [Jupyter Book](https://jupyterbook.org/intro.html) is used to
    compile and create the HTML files locally.
5.  The HTML content is then hosted with [GitHub
    Actions](https://docs.github.com/en/actions). This HTML content is
    pushed manually into the docs folder.
