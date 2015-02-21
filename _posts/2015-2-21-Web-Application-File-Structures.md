---
layout: post
title: Web Application File Structures
---

This is a topic where most people give you wrong advice. Unless you have seen a lot of applications grow from scratch, it is easy to follow the pattern of your last projects, or just take advice from someone else. The reason why it is very easy to go wrong is that the right solution depends on the nature of your product.

First, we will focus on the motivation behind creating a folder structure that makes sense. We will then explore the different methods for grouping files. As I have recent experience in developing Backbone+Marionette applications, I will describe two possible folder structures, including reasons why one of the structures would have massively backfired in our main product. This article will be concluded with special emphasis on the dynamic nature of code reorganization and refactoring will be encouraged for the purpose of improving the quality of your product. 

### Why do you need a solid file structure?

In the world of efficient text editors and integrated development environments, it is very easy to locate the resource you need, as long as you know what you are searching for. As soon as you go beyond locating a resource you already know, problems arise. The way how you structure your files should also give you a clear overview. When you have to think hard about where your features are, something is definitely wrong.

Many people accept the structure they read in a book or the structure suggested by their favorite code generator. Have you ever tried out a boilerplate generator written by someone else just to conclude that the structure is not flexible enough for your needs? 

Different problems have different solutions. You can create a throw-away prototype, a small web-app in a week, or a huge corporate software with features requiring developer hours in the ten thousands range. In the first two cases, this article does not matter much to you. In the last case, please go the extra mile and think about a structure that makes sense to you instead of accepting someone else's work.

> I just checked a Backbone+Marionette application I regularly contributed to. It contains more than a thousand files. How many files does the tutorial you follow have? If you went beyond a hundred, I would be surprised. This is why third party advice should be filtered by common sense.

### Syntactic and Semantic Grouping

When you create folders such as `js`, `css`, `templates`, you group your files in a syntactic way. In other words, files with different syntax are created in different folders. When naming your folders after the name of your pages and components, the grouping you apply is semantic. In most applications, both syntactic and semantic grouping are used.

> The original terminology refers to "Package by Layer" versus "Package by Feature". For a full description of these terms, check out <a href="http://www.javapractices.com/topic/TopicAction.do?Id=205" target="_blank">this link</a>.

Many applications start grouping the files based on their format. Javascript, CSS and HTML files are often in different folders. Third party dependencies are also separated from the code of the application. Semantic grouping is only applied after different types of files have been separated. For instance, consider the following folder structure:

```
├── css
│   └── style.css
├── js
│   ├── lib
│   │   ├── backbone.min.js
│   │   ├── jquery.min.js
│   │   └── underscore.min.js
│   ├── models
│   │   ├── account.model.js
│   │   ├── dashboard.model.js
│   │   └── report.model.js
│   ├── presenters
│   │   ├── header
│   │   │   ├── menu.presenter.js
│   │   │   └── search.presenter.js
│   │   └── contents
│   │       ├── dashboard.presenter.js
│   │       └── reports.presenter.js
│   ├── app.js
│   └── main.js  
├── sass
│   ├── pages
│   │   ├── header
│   │   │   ├── menu.sass
│   │   │   └── search.sass
│   │   └── contents
│   │       ├── dashboard.sass
│   │       └── reports.sass
│   └── style.sass
├── templates
│   ├── header
│   │   ├── menu.template
│   │   └── search.template
│   └── contents
│       ├── dashboard.template
│       └── reports.template
└── index.php
```

It is worth noting that the exact same semantic structure of the regions of your page (header, contents) and their corresponding files are replicated three times. The only reason why a fourth occurrence was prevented in the models folder is that this structure aims at differentiating data objects from their presentation. Would you like to develop software structured like this? 

Some of you may say yes, and the answer would be fully justified. For instance, in your Sublime Text editor, pressing Command+P or CTRL+P helps you locate each dashboard file if you search for them. Compass can easily watch for any changes in the sass files inside the sass folder. If you have ten or even twenty views and templates, you can get away with this structure quite well.

Imagine that the size of your software grows beyond a point when this structure stays maintainable. For instance, the number of your templates can grow to a hundred or more very easily, within a matter of months. Syntactic grouping of the files will start hurting you more and more. Suppose that one Backbone View and one template belongs to your Todo list component, and you would like to refactor it such that you want to create a template for the container of the list and a different template for a list item. In addition, you would like to represent your todo list with a composite view and a list of item views. The corresponding stylesheets have to be refactored as well. Fortunately, you can keep your models and your collections intact, therefore this refactor will only require you to make changes in three different places of the folder structure. Given that you were supposed to modify files belonging to a component, it is quite hard to accept that the same change in file names has to be replicated three times in the exact same way. 

Another problem of the top level syntactic grouping is that the relations between the files is not possible to overview at a glance. Suppose that you have a hundred models. How do you know if a model is used by three different presenters? How do you know which templates are nested in each other? 

Once your application grows, semantic grouping of your components comes to the rescue. Let's re-group the same files of the original example:

```
├── css
│   └── style.css
├── lib
│   ├── backbone.min.js
│   ├── jquery.min.js
│   └── underscore.min.js
├── models
│   ├── account.model.js
│   ├── dashboard.model.js
│   └── report.model.js
├── regions
│   ├── header
│   │   ├── menu
│   │   │   ├── menu.presenter.js
│   │   │   ├── menu.scss
│   │   │   └── menu.template
│   │   └── search
│   │       ├── search.presenter.js
│   │       ├── search.scss
│   │       └── search.template
│   └── contents
│       ├── dashboard
│       │   ├── dashboard.presenter.js
│       │   ├── dashboard.scss
│       │   └── dashboard.template
│       └── reports
│           ├── reports.presenter.js
│           ├── reports.scss
│           └── reports.template 
├── sass
│   └── style.sass
├── app.js
├── main.js 
└── index.php
```

Menu, Search, Dashboard and Reports just became almost fully self-contained components. The only missing object required to operate a component is the model referenced by the presenter file. We keep them in a separate folder for now. It may become beneficial in your case to include the models inside the components, especially if a model exclusively belongs to one component and especially if the model does not post Ajax requests. Keeping the models in one place is closer to the package by layer approach, while moving the models inside the components they are used in encourages packaging by feature.

Notice that a component may have subcomponents or a component may consist of multiple presenters, templates or even stylesheets. Once the structure of a component becomes more complex, it makes sense to separate the presenters, templates, stylesheets, models and collections into different folders inside each component. 

### Reorganization guided by refactoring

"Don't cross the bridge until you come to it" is a common self-help advice that applies to your application structure as well. If you would like to build a throw-away prototype in 4 hours, any structure will do fine. In some cases however, some applications tend to become larger than originally intended. In this case, the folder structure that helped you at the beginning would end up causing trouble. The only appropriate answer is to change the structure.

Although mixing Javascript files with templates and stylesheets does not sound intuitive at first, in practice this is exactly what you need to have an overview of a component, including internal state, template and presentation. 

In some cases, common components may cause trouble. For instance, suppose that one of your forms contains an input field where you can enter time, bundled with a dropdown list to select the corresponding units. If the units are in seconds and you enter 60, changing seconds to minutes would automatically replace the contents of the textfield with 1. Furthermore, suppose that you would like to use the exact same field in another form. The first step is to extract all the functionality belonging to the time field and the unit selector, create its own template, presenter, model and the two way data binding. Select the region where you would like to use this component and place it in its `components` folder:

```
   └── contents
   	   ├── components
   	   |   └──+ timelength 
       ├──+ dashboard
       ├──+ scheduler
       ...
       └──+ reports
```

In this example, the `timelength` component was indicated to be used inside the `contents` region. Does it matter if one day this component will be reused in the header? No. Cross the bridge once you get there. If you ever found out that your component is needed in another region as well, take the component out to a `components` folder common to all regions. When developing a big application, it is always a worse decision to keep using a legacy or a generalized structure instead of moving your components exactly where they belong. The cost of the move is very small, a junior developer can finish it without errors in a matter of 10 to 15 minutes, including the update of your CommonJs module config if needed.

It is worth mentioning that in case your company is using Git and you have a multiple teams maintaining multiple repositories, you may end up in a situation where your application uses multiple submodules. The Git submodule system is quite convenient in general, but make sure you always satisfy the following conditions:
- make sure the build is fully automatic. Make sure that your task runner successfully executes all the tasks, including the update of your submodules, even if your submodule folders are not clean
- make sure you never include the same submodule twice, not even recursively

### Semantic grouping for better maintainability

We have compared the syntactic and the semantic grouping techniques and found out that the more code you are responsible for, the more you need to create reusable components and organize your code such that your components contain everything you need for maintenance, including templates, stylesheets, models and views.

When you are in doubt whether your file structure is correct, ask yourself what you need to have a full overview of a page you are looking at. Alternatively, when trying to locate a feature, ask yourself what files you need for that feature. If those files are far away or their location is not obvious to you, it makes sense to create a better structure.
