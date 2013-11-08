This is for a blog or blog-like sites (ie. documentation). It's intended to have just the right amount of default styling. Most blog boilerplates are either too prescriptive or too unstructured in their layouts. Both extremes make you do unneccessary front-end work to get up and running. This is intended to be a happy medium:

![Feed](http://f.cl.ly/items/1Q0d0b3u3l1W0L3V1r3v/Screen%20Shot%202013-11-08%20at%2012.36.25.png)
![Post](http://f.cl.ly/items/13042w2p070r2428403Z/Screen%20Shot%202013-11-08%20at%2013.00.07.png)

## What does it come with?
Structure:
- Simple blog
- Pagination
- XML feed
- Tags
- Simple fluid layout with sidebar
- Minimal styling to be easily customizeable ([screenshot](http://cl.ly/image/460610072B2y))

Defaults:
- HAML
- SCSS
- Compass
- Susy for grids

## Installation:

Middleman supports [project templates](http://middlemanapp.com/getting-started/#toc_6). Project templates are awesome because you can keep all the boilerplate crap in one place on your box. However, the installation process is a little involved. To install middleman-blog-project-template as a template available to your user:

1. Install Middleman (requires Ruby and Rubygems): ``gem install middleman``
2. Clone the Git repository into ``~/.middleman``, like so ``git clone https://github.com/rainforestapp/middleman-blog-project-template.git ~/.middleman/blog-boilerplate```
3. Create your project folder and ``cd`` into it.
4. Use the new template argument for the ```middleman init``` command within your project folder: ``middleman init my_new_project --template=blog-boilerplate``

## Usage:

1. Ensure you have Ruby + bundler installed
2. Run ``bundle install`` from within your project directory
3. Run ``bundle exec middleman server`` and go to http://0.0.0.0:4567/
