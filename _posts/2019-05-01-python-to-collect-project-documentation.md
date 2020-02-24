---
layout: post
title: How I used Python to Collect Project Documentation
tags: [Python, Documentation]
comments: true
---

Documentation is always a challenge in the world of Software Engineering. Often times documentation is the last thing that gets done, and it’s hard to keep it updated. Some people make the argument that the code is the documentation, but you still need guides and other instructions for people to use your project.

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/05/1_o-cjc6h9ks7c3txd80g8_a.jpeg" />
</p>

One big challenge when you have a big project, is that even good documentation can be overwhelming. Often times people will create GitHub Wikis that are nothing more than several different markdown files linked together. This works, but makes it easy to get lost in the weeds. Moreover, it would be really helpful if there was a way to organize this documentation in a single place.

Recently, I wanted to gather a large number of markdown files from a project I was working on. Basically, I needed to __organize documentation__ into a single place. This place had to be easily readable, can be shared, and can be searched.

So I found [MkDocs](https://github.com/mkdocs/mkdocs/).

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/05/1_lymwvgr0wgf5flhy2nrwow-1.png" />
</p>

Image copied from the MkDocs page here [www.mkdocs.org](https://www.mkdocs.org/)

MkDocs is a site generator that was written in python and is also open source. The site generator basically creates a static site using markdown files. You can host your site in [GitHub Pages](https://pages.github.com/) or an [AWS S3 bucket static hosting](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html). You can do everything using the MkDocs CLI, and just requires you to have python installed on whatever machine you’re using.

MkDocs also uses themes, so you can pick how you can customize what you want your site’s look and feel to be. For any MkDocs project I’d recommend the [Material Theme](https://github.com/squidfunk/mkdocs-material) which is built using [Google’s Material Design Specification](https://material.io/design/).

MkDocs creates the site using a __yaml file__ to determine navigation, and a __docs folder__ for the markdown files you want in your site. The yaml file is basically instructions for MkDocs to create the site and usually looks similar to this:

```js
site_name: firstSite 
nav:     
  - Home: index.md     
  - About: about.md
```

This example shows a site named `first site` that has two markdown files in the docs folder to include __index.md__ and __about.md__. The generated site would enable lefthand navigation from the main homepage and look like the following:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/05/0_p0t33cqknw4aznfn.png" />
</p>

image copied from the MkDocs page here [www.mkdocs.org](https://www.mkdocs.org/)

Since MkDocs uses markdown files, this was perfect for what I was wanted to do. All I needed was to gather my projects README files in one place, and build in navigation where appropriate.

With my site generator picked out, I created a [python](https://www.python.org/) script that walked through the project and collected the files. Then I used [npm](https://www.npmjs.com/) to create scripts that automated this process, so whenever I wanted to update the documentation site, I just ran npm run deploy and pushed updated documentation to the repo’s GitHub pages site. With this process, I now had all the documentation in one location that was easily searchable and can be shared.

In the following sections, I’m going to walkthrough how you can create a documentation generator similar to what I did.

I’ve created a sample project [on GitHub](https://github.com/andrewevans02/docs-generator) that I’ll be using for my explanation. This sample project uses the [Angular Material GitHub project](https://github.com/angular/material2) as a source for gathering documentation. You can change this to be anything, I’m just using Angular Material here since it has a lot of README files and is a good example.

## Using My Sample Project

First, go to GitHub and create a fork of my [sample project](https://github.com/andrewevans02/docs-generator). Then do a Git Clone of the fork you created.

You should now have a project that has the following structure:

```bash
.
├── LICENSE
├── README.MD
├── generated-site
│   ├── base.yml
│   └── docs
│       └── index.md
├── package.json
└── site.py
```

* The `generated-site` folder has the main files and folders that MkDocs will use to generate the site. When our scripts run, this docs folder will also create a `site-files` folder to hold the markdown files that MkDocs will use.
* The `index.md` is sitting inside the docs folder for convenience, and to use as a home page for the generated site. You can just leave this file blank because it’s just as a landing page.
* The `base.yml` file will be used by the python script to create a custom yaml file for MkDocs to use when creating the site. As the python script runs, it will write values to this file. The file I’ve created includes the material theme, but you can customize this for your project if you wanted to.
* I used npm init to create the `package.json` at the project’s root. If you want to use this for your project, you’ll need to change the values for author, project, etc. to suit your needs. The `npm scripts` that I’ve included inside this file also allow us to build and deploy our project.
* The `site.py` file is the python script which I will cover in the next section.

## Creating the Python Script

The python script `site.py` in my sample project collects the markdown files from your GitHub project into a folder that MkDocs can read from. The script basically just walks through a local copy of your GitHub project, and then generates a staging area that MkDocs will use to create your documentation site. I’m going to walk through the script section by section, and show you were you can add customizations.

So let’s get started!

Here is the first part of the script:

```python
import os
import shutil

# global
md_files = []
project_folder = os.path.abspath(os.path.join(__file__ ,"../material2/"))
base_mkdocs = os.path.abspath(os.path.join(__file__ ,"../generated-site/base.yml"))
created_mkdocs = os.path.abspath(os.path.join(__file__ ,"../generated-site/mkdocs.yml"))
copied_folder = os.path.abspath(os.path.join(__file__ ,"../generated-site/docs/site-files/"))

# delete generated files and folders
def delete_files_and_folders():  
  shutil.rmtree(copied_folder, True)      
  shutil.rmtree(created_mkdocs, True)  
  print('deleted files and folders successfully')

# create initial files and folders
def create_files_and_folders():      
  os.makedirs(copied_folder)      
  os.makedirs(copied_folder + '/source')      
  os.makedirs(copied_folder + '/github')
  os.makedirs(copied_folder + '/guides')
  os.makedirs(copied_folder + '/project')
  shutil.copy(base_mkdocs, created_mkdocs)
  print('created files and folders successfully')  

# delete base yaml file to rebuild
def create_yaml():  
  shutil.copy(base_mkdocs, created_mkdocs)    
  print('created yaml file successfully') 
```

This first part is pretty straightforward. We are declaring the global values, and then we define cleanup functions that our script will use before it runs.

If you notice the `create_files_and_folders` function here is actually creating several subdirectories. This caters to the Angular Material repo, but you can change this to be whatever you want for your project.

Here is the second part of the script:

```python
def select_md():  
  os.chdir(project_folder)  
  for root, dirs, files in os.walk(project_folder):      
    for file in files:        
      if file.endswith(".md"):          
        file_path = os.path.join(root, file)              
        print(file_path)            
        md_files.append(file_path)  

def copy_md():  
  # in order for the script to work correctly
  # as the files are copied the yaml file is updated
  # since this relationship cannot be seperated here
  # we have multiple loops for each group of files
  # performance improvements and innovation welcome
  with open(created_mkdocs, "a") as myfile:        
    myfile.write('  - source' + ':' + '\r')      
    counter = 0      
    for file_found in md_files: 
      if('/src/lib' in file_found):
        file_split = file_found.split('/')          
        copy_name = ''          
        display_name = ''     
        for split in file_split:   
          if '.md' in split:              
            copy_name = str(counter) + '-' + split                  
            display_name = str(counter) + '-' + split.replace('.md', '')           
            break        
        copied_file = copied_folder + "/source/" + copy_name             
        shutil.copy(file_found, copied_file)         
        myfile.write('    - ' + display_name + ': site-files/source/' + copy_name + '\r')          
        counter = counter + 1

  # github files
  with open(created_mkdocs, "a") as myfile:        
    myfile.write('  - github' + ':' + '\r')      
    counter = 0      
    for file_found in md_files: 
      if('.github' in file_found):
        file_split = file_found.split('/')          
        copy_name = ''          
        display_name = ''     
        for split in file_split:   
          if '.md' in split:              
            copy_name = str(counter) + '-' + split                  
            display_name = str(counter) + '-' + split.replace('.md', '')           
            break        
        copied_file = copied_folder + "/github/" + copy_name             
        shutil.copy(file_found, copied_file)         
        myfile.write('    - ' + display_name + ': site-files/github/' + copy_name + '\r')          
        counter = counter + 1

  # guides files
  with open(created_mkdocs, "a") as myfile:        
    myfile.write('  - guides' + ':' + '\r')      
    counter = 0      
    for file_found in md_files: 
      if('guides' in file_found):
        file_split = file_found.split('/')          
        copy_name = ''          
        display_name = ''     
        for split in file_split:   
          if '.md' in split:              
            copy_name = str(counter) + '-' + split                  
            display_name = str(counter) + '-' + split.replace('.md', '')           
            break        
        copied_file = copied_folder + "/guides/" + copy_name             
        shutil.copy(file_found, copied_file)         
        myfile.write('    - ' + display_name + ': site-files/guides/' + copy_name + '\r')          
        counter = counter + 1

  # project files
  with open(created_mkdocs, "a") as myfile:        
    myfile.write('  - project' + ':' + '\r')      
    counter = 0      
    for file_found in md_files: 
      if('guides' in file_found) or ('/src/lib' in file_found) or ('.github' in file_found):
        continue
      else:
        file_split = file_found.split('/')          
        copy_name = ''          
        display_name = ''     
        for split in file_split:   
          if '.md' in split:              
            copy_name = str(counter) + '-' + split                  
            display_name = str(counter) + '-' + split.replace('.md', '')           
            break        
        copied_file = copied_folder + "/project/" + copy_name             
        shutil.copy(file_found, copied_file)         
        myfile.write('    - ' + display_name + ': site-files/project/' + copy_name + '\r')
```

This part is also pretty straightforward. The function `select_md()` walks the project directory to find the markdown files, and then the function `copy_md()` copies these files over to the site directory we created in the earlier section.

If you notice, the `copy_md` function is actually looking for files that have `/src/lib` and other specific values in them. The multiple loops look for these values, and when they see them they copy them into specific subdirectories. These subdirectories are then used (by MkDocs) at site creation to group the files into a collector on the lefthand navigation of the site generated. This makes it nice because you can easily group similar files in an area that is easily navigated.

Finally, here’s the main section of the script which executes the functions we’ve declared:

```python
# Start
delete_files_and_folders()
create_files_and_folders()
select_md()
copy_md()
```

In order, they run the following:

* `delete_files_and_folders()` — deletes any files and folders that were generated from the last run of the script
* `create_files_and_folders()` — creates any files and folders that are necessary for copying and building the structure for MkDocs
* `select_md` — selects the markdown files from the locally downloaded copy of your GitHub project
* `copy_md` — copies the markdown files and writes their values into the _markdown.yml_ file so that MkDocs can use the files and yaml to create the site

## Using the NPM Scripts

So with the python script above working the last step is to create some npm scripts that automate this whole process. Our goal is to get this working in a way that one terminal command can do the whole thing.

If you open the `package.json` file, look at the “scripts” section. This is where I’ve built custom commands that npm will recognize if you’re in the project’s root directory. To run any of these scripts just open a terminal at the root directory and prefix them with npm run. So for my example for the script called `build` it could be run with npm run build.

Here are the scripts I used:

```yaml
  "scripts": {
    "github-clone": "git clone https://github.com/angular/material2.git",
    "github-delete": "rm -rf material2",    
    "install-dependencies": "pip install mkdocs && pip install mkdocs-material",
    "build": "npm run github-delete && npm run github-clone && python site.py && cd generated-site && mkdocs serve",
    "deploy": "cd generated-site && mkdocs build && mkdocs gh-deploy"
  }
```

* `github-clone` — clones the GitHub project you want to gather documentation from (in this case its Angular Material)
* `github-delete` — deletes the local copy of your GitHub project
* `install-dependencies` — installs MkDocs and the material theme with python’s pip ([per the getting started guide](https://www.mkdocs.org/#getting-started))
* `build` — builds the project and serves it locally
* `deploy` — deploys the project to the repo’s GitHub pages site

So now, with all of this together, go to the root of the project and run the following:

npm run build and open your browser to http://localhost:8000 to see the following:


<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/05/first.png" />
</p>

If you use the side navigation you can see where our `copy_md()` function has grouped the files into subfolders like so:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/05/second.png" />
</p>

One of the best parts of MkDocs is that you can also use the search functionality to make finding documentation easy:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/05/third.png" />
</p>

Finally, stop the terminal and run npm run deploy to push the changes to the GitHub Page’s site for the repo. When it finishes deploying you should see the following:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/05/fourth.png" />
</p>

MkDocs will push the site to a gh-pages branch on your repo. This can be changed to a different branch and URL, but I’m just using the defaults here.

The GitHub Pages site where your MkDocs site is deploy to follows the convention of: `https://<github_username>.github.io/<repo_name>`


<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/05/fifth.png" />
</p>

## Closing Thoughts

So with this post I shared one way to use both MkDocs and Python to organize a project’s documentation. This can be shared, and creates a convenient way to showcase your documentation. The process I’ve setup here is still manual, but next steps would be to put this into a CICD pipeline. You could imagine running this whenever your project does a production release, or even on just whenever PRs get merged. If you use a Docker container with a CI system like Jenkins or CircleCI it makes all of this super easy. I hope you enjoyed learning about this, and can use this for some future projects.