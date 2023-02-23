---
title: Data Layers, and Finding Ways to Reduce Repetition
category: Rovani in C♯
date: 2014-04-02
---

![Data Workflow](/images/data-layers.png "Data Workflow")

This should be a footnote, but I have always wanted an excuse to use the stencils from the Visio Work Flow Diagram.  They just seem like a fun way to represent something.

What does it take, if almost everything is created by hand, to do something as simple as add a new field to a table in the database:

1. UPDATE TABLE script
1. Edit DBML xml file (let Visual Studio regenerate the designer file)
1. Edit the POCO class and any mapping methods
1. Edit the JSON serialization method
1. Edit the Knockout.js Model or ViewModel object
1. Edit the HTML template

This is just for displaying the content.  Additional work is required to allow for editing values and pushing them all the way back to the database.  This lead me down the road of finding how I can eliminate steps.  I reasoned that there had to be solutions already created that would took care of any least one of the various steps.  Thankfully, I was able to find solutions for most of it.

Entity Framework Code First (ef-cf) has been the most promising tool for me to use to cut out the step 1 and roll it into a replacement for step 2.  AutoMapper took care of replacing mapping methods in step 3 (though I still have to update the POCO class).  Json.Net has been my library of choice for serialization.  Knockoutjs.Mapping plugin too care of Step 5.

At this point, updating the data model looks like this:

1. Edit EF model.
1. Edit POCO class.
1. Edit HTML template.

This is clearly more manageable.  I am putting together a tutorial all about taking a project from scratch and implementing all of the pieces needed to get this configured.  If I stopped having so many false starts, I am sure the post (and the project) would be finished by now.