+++
title = "Eljira Announcement"
author = ["Chris Quinn"]
date = 2025-05-02T08:37:00-06:00
tags = ["emacs", "software"]
draft = false
+++

[Eljira](https://github.com/sawwheet/eljira) is an Emacs interface into Jira.

I had three main motiviations in creating this project:

1.  I hate Jira, and none of the existing interfaces fit my needs (mainly custom field related)
    1.  For something so central to my work, it is super inefficient to navigate.
2.  I wanted to dig my teeth into lisp
3.  I want the trolls to tear apart an open source project of mine

Unfortunately I cannot give too much of a visual demo, becaus the only Jira instance I have access
to is my employers. Redacting info, or creating a new personal Jira instance is work I don't want to
do. With that said, here is a general overview of the project:


## Queries {#queries}

In my Jira worfklow, I have to switch between different projects, and different contexts within a
given project. For example: in my main Jira project I will want to look at my issues, issues I have recently
closed, newly created issues, and so on. Then there are other projects where I want do the same
thing. Thus the heart of this project is born, in the form of `eljira-queries`.

The gist of the idea is that you create a 'query', and when this query is executed you get all the
issues returned in a nice table format.

In the `eljira-queries` variable, I define the different 'views' of issues I look at on a regular basis:

```emacs-lisp
(setq eljira-queries '((:name "My Issues"
                         :jql "project = \"SRE\" AND assignee = currentUser() AND resolution = Unresolved ORDER BY created DESC"
                         :fields ((:name "Key" :formatter (lambda (value)
                                                            (propertize value 'face 'bold)))
                                  (:name "Type" :field "issuetype")
                                  (:name "Status" :field "status")
                                  (:name "Summary" :field "summary")
                                  (:name "Reporter" :field "reporter")
                                  (:name "Points" :field "customfield_10000" :min-width 7
                                         :formatter (lambda (value)
                                                      (number-to-string (if value (round value) 0))))
                                  (:name "Comments" :field "comment" :hide t)
                                  (:name "Parent" :field "parent" :getter (lambda (issue table)
                                                                            (let-alist (eljira-issue-fields issue)
                                                                              .parent.key)))
                                  (:name "Description" :field "description" :hide t)))
                  (:name "My Issues - In Progress"
                         :jql "project = \"SRE\" AND assignee = currentUser() AND status = \"In Progress\" ORDER BY type DESC"
                         :fields ((:name "Key" :formatter (lambda (value)
                                                            (propertize value 'face 'bold)))
                                  (:name "Type" :field "issuetype")
                                  (:name "Status" :field "status")
                                  (:name "Summary" :field "summary")
                                  (:name "Reporter" :field "reporter")
                                  (:name "Points" :field "customfield_10000" :min-width 7
                                         :formatter (lambda (value)
                                                      (number-to-string (if value (round value) 0))))
                                  (:name "Comments" :field "comment" :hide t)
                                  (:name "Parent" :field "parent" :getter (lambda (issue table)
                                                                            (let-alist (eljira-issue-fields issue)
                                                                              .parent.key)))
                                  (:name "Description" :field "description" :hide t)))
                  (:name "Issues Completed Past Week"
                         :jql "assignee = currentUser() AND status = Done AND updated >= -1w"
                         :fields ((:name "Key" :formatter (lambda (value)
                                                            (propertize value 'face 'bold)))
                                  (:name "Type" :field "issuetype")
                                  (:name "Status" :field "status")
                                  (:name "Summary" :field "summary")
                                  (:name "Reporter" :field "reporter")
                                  (:name "Assignee" :field "assignee")
                                  (:name "Points" :field "customfield_10000" :min-width 7
                                         :formatter (lambda (value)
                                                      (number-to-string (if value (round value) 0))))
                                  (:name "Comments" :field "comment" :hide t)
                                  (:name "Description" :field "description" :hide t)))))
```

Given that this is my first lisp project, I was quite excited when working through this data
structure. The realization that my data and configuration can replace so much code in the main
project was a big "ah-ha" lisp moment! Putting custom formatter and getter functions in the
configuration is great. No more trying to define custom formats for the end user to utilize; Just
create your own!

With this variable defined I run `M-x eljira` and am prompted to choose one of these queries. I am
then presented with an interface like so:

{{< figure src="/images/UI.png" >}}

The top window is the `query` window, and the bottom window is the `context` window.


## Contexts {#contexts}

One of my biggest gripes with Jira is just how much crap there is. In reality I need to see very
little of the information they cram in to an issue. Part of this is a Jira issue, and part of it is
an organization issue; Regardless, I wanted less. This is where the `contexts` concept comes in to
play.

The bottom window will automatically update to display the currently set `context` of the issue at
point. So as you scroll through the query window, you will automatically see the information you
want.

I have two contexts that I use: `Description` and `Comments`. When the context is set to
'Description', I will see the description of the issue. I will let you guess what I see when it is
set to 'Comments'.

There is also a nice header that you can configure to show simple key value fields.


## Custom Fields {#custom-fields}

Perhaps my number 1 issue with existing Jira interfaces, is the custom field support they
provide. In my company, each issue requires a set of custom fields at creation time. I also am
constantly editing these custom fields. While the UX around my implementation is sub-par at best, I
can actually handle all my needs right from Emacs. Something I have been unable to accomplish until
now.

What is cool is that each custom field I add to the `eljira-customfields` variable automatically gets
put in a transient menu for me. There are some caveats to this depending on the type of custom
field.

Here I set my customfields:

```emacs-lisp
(setq eljira-customfields '((:name "Points" :key "p" :field "customfield_10000" :type integer)
                            (:name "Need by date" :key "d" :field "customfield_10001" :type date)))
```

Now when I edit an issue, those fields show up in my transient menu:
{{< figure src="/images/edit-transient.png" >}}

Each custom field defined in here will be configurable when editing, and creating an issue.

The caveat is if the customfield is not of type integer, date, or number. For example: cascading
field. For this, there is a static variable that can be configured: `eljira-static-customfields`. As
the name suggests, the values in here are static and applied to an issue upon creation. As I
mentioned, my company requires some fields when creating an issue. Luckily I can just hard code it
as it as this static variable is aplicable 99% of the time. You cannot modify this field when
editing an issue. I never said this project was startup worthy!


## Actions {#actions}

As mentioned at the beginning of this post, Jira is super inefficient to navigate for something that
is so central to my workflow; Especially when I am used to the power of emacs. There are a growing
number of actions I want to take on a given issue, and I want to be able to do so without searching
for, and clicking on multiple buttons.

This might have been my favorite part to code as I wanted to use a macro. As of writing this, the
macro is ugly; both in how I use it, and how it is written. But whatever, it gets the job done for
me. This macro is called `eljira-defaction`.

As the name suggests, it defines actions that can be taken on any issue. There are plenty of builtin
actions:

-   Edit assignee
-   Move status
-   Edit summary/description
-   etc

My favorite that I included is `Move status continuously`. I often have to progress an issue from 1
status to another, and it includes multiple hops. For example: New -&gt; To Do -&gt; In Progress -&gt; Done.
In Jira, I would have to click the dropdown menu 3 times to go from New to Done. With this custom
action, I will be continuously prompted for the status I want to move to until I push `C-g` to quit.

For any other actions I want that are specific to my workflow I can utilize `eljira-defaction` in my
personal config. They will be added to the transient menu automatically.
