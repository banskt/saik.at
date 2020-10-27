+++
date = "2020-10-27T04:05:25-06:00"
draft = false
title = "Blogging Research: Jupyter Notebooks with fastpages"
+++

I keep track of my daily work using Jupyter Notebook.
I use [fastpages](https://github.com/fastai/fastpages), developed by [fast.ai](https://www.fast.ai/)
to simplify the process of creating blog posts from my research.
A simple push to Github makes my research public (see my [research notebooks](https://banskt.github.io/))
and helps to share and update contents daily.

For the setup, I created a [Github Pages](https://pages.github.com/) website using [Jekyll](https://jekyllrb.com/).
The source code is [here](https://github.com/banskt/banskt.github.io).

## Creating new projects
I follow the [setup instructions](https://github.com/fastai/fastpages#setup-instructions) of fastpages
to set up a new Github repository.
And, I link the new repository in the frontpage of my Github Pages (filename `index.markdown`). 
Here are two example entries:
```
- [Empirical Bayes Multiple Regression](/iridge)
- [GLM with adaptive shrinkage prior](/glm-ash-notes)
```
Push the edited file to my Github Pages, and the new project is ready.

## Personal configuration
I modify the following files in the newly created clone of fastpages to personalize the look and feel.
```
_config.yml
_includes/post_list.html
_includes/footer.html
_sass/minima/custom-styles.scss
images/favicon.ico
index.html
```
To reduce effort, I created them once and I just copy them over to the newly created projects. 

## Creating new posts
Daily workflow involves working on new Jupyter Notebooks,
and pushing them at the end of the day.
Again, the documentation at fastpages is very helpful for formatting the web pages.

And, that is it!
