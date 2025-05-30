# Contributing to sgibson91.github.io

:space_invader: :tada: First off, thank you for taking the time to contribute! :tada: :space_invader:

The following is a set of guidelines for contributing to [sgibson91.github.io](https://sgibson91.github.io) on GitHub.
These are mostly guidelines, not rules.
Use your best judgement and feel free to propose changes to this document in a Pull Request.

**Table of Contents**

- [Code of Conduct :purple_heart:](#code-of-conduct-purple_heart)
- [What should I know before I get started? :question:](#what-should-i-know-before-i-get-started-question)
  - [What is it conceptually? 🤔](#what-is-it-conceptually-)
  - [What is it actually? :mag:](#what-is-it-actually-mag)
- [How can I contribute? :gift:](#how-can-i-contribute-gift)
  - [Reporting Bugs :bug:](#reporting-bugs-bug)
  - [Suggesting Blog Posts :memo:](#suggesting-blog-posts-memo)
  - [Making a Pull Request :arrow_right:](#making-a-pull-request-arrow_right)
- [Style Guides :art:](#style-guides-art)
  - [Markdown style guide :pencil2:](#markdown-style-guide-pencil2)
  - [Git Commit Messages :envelope:](#git-commit-messages-envelope)
- [Additional Notes :notebook:](#additional-notes-notebook)
  - [Issue and Pull Request Labels :label:](#issue-and-pull-request-labels-label)

---

## Code of Conduct :purple_heart:

This project and everyone participating in it is expected to abide by and uphold the [Code of Conduct](./CODE_OF_CONDUCT.md).
Please report unacceptable behaviour to [drsarahlgibson@gmail.com](mailto:drsarahlgibson@gmail.com).

## What should I know before I get started? :question:

### What is it conceptually? 🤔

[sgibson91.github.io](https://sgibson91.github.io) is the personal, professional website of Dr Sarah Gibson.
She uses this website to advertise her skills and projects she is involved in (where they are open), blog about her experiences as an RSE, and document any public speaking events she has taken part in.
The opinions she expresses here are her own and not representative of any of the communities she participates in.

As a result, Sarah reserves the right to have final say on any content that will be added to this repo in order to make sure that it aligns with her beliefs and the professional manner with which she wishes to conduct herself.

### What is it actually? :mag:

This is a personal webpage built by [Hugo](https://gohugo.io/), hosted by [GitHub Pages](https://pages.github.com/) and implements the [Anatole](https://themes.gohugo.io/anatole/) Hugo theme for styling.

## How can I contribute? :gift:

### Reporting Bugs :bug:

Please open a bug report issue if you discover a problem with the website.
This could be something like:

- a spelling mistake in a post,
- a dead link,
- a page or element that doesn't render properly.

The repository has a [bug report template](./.github/ISSUE_TEMPLATE/bug_report.md) that should help you describe the issue you're seeing.
Please fill it in to the best of your ability and as descriptively as possible as this will help Sarah squash that bug! :bug:

### Suggesting Blog Posts :memo:

If there's a particular topic you'd like to hear Sarah's opinion on in blog form, please open a blog issue.
Sarah is always looking for new ideas for topics of her blog posts and would love to know what you want to hear about!
And yes, this does include co-writing blogs with her as well!

The repository has a [blog post issue](./.github/ISSUE_TEMPLATE/blog-post-template.md) that should help outline ideas and collect sources to build the blog post on.
There is also a [blog post template](./_posts/blog/template.md) to begin a new blog post.

### Making a Pull Request :arrow_right:

The following guidelines should help you make a successful Pull Request to this repository.

1. [Fork the repo](https://help.github.com/en/github/getting-started-with-github/fork-a-repo)
2. [Create a new branch in your fork](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-and-deleting-branches-within-your-repository#creating-a-branch)
   1. When naming the branch, please follow the convention `<type>/<issue-number>/<short-descriptor>`.
      For example, if you are creating a branch to work on a blog post about Continuous Integration outlined in issue #11, your branch name would look as follows: `blog/11/ci`.
3. Edit the files you need to or add new ones!
4. [Open up your Pull Request](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request-from-a-fork)
   1. The repository has a [pull request template](./.github/PULL_REQUEST_TEMPLATE.md) that should help you describe the work you have done.

Congratulations!
You've opened your Pull Request! :tada:

Sarah will then review your contribution and, providing tests pass, will merge it!

## Style Guides :art:

### Markdown style guide :pencil2:

The webpages are written in Markdown syntax and are automatically transformed into HTML by Jekyll.

When writing content in Markdown, it is recommended to begin each new sentence on a new line.
(Check out the raw version of this file to see an example!)
Each sentence will be rendered as a single paragraph until a blank line is found, this will create a new paragraph.
This is the recommended way to write since, if/when changes are suggested during the review process, the GitHub User Interface will only highlight the sentence the changes have been requested for - not the whole paragraph.
This makes reviews much easier to read!

If you are not familiar with Markdown, checkout [GitHub's Guide to Mastering Markdown](https://guides.github.com/features/mastering-markdown/).

### Git Commit Messages :envelope:

You should feel free to spice up your git commit messages with some emojis! :tada:

Here is a list of some recommended emoji types and their meanings:

- :fire: for removing files
- :bug: for fixing bugs
- :sparkles: for adding new project pages
- :pencil: for writing blogs
- :lipstick: for updating the style files
- :construction: for work in progress
- :green_heart: for fixing the CI build
- :pencil2: for fixing typos
- :ok_hand: for commits made during review
- :speaker: for adding a new talk

You can find a full list of emojis suggested by the [gitmoji project](https://gitmoji.carloscuesta.me/).

## Additional Notes :notebook:

### Issue and Pull Request Labels :label:

This repo uses [GitHub labels](https://help.github.com/en/github/managing-your-work-on-github/about-labels) to track and manage issues and pull requests since GitHub can easily be [searched by labels](https://help.github.com/en/github/searching-for-information-on-github/searching-issues-and-pull-requests#search-by-label).
Here is a list of the most commonly used labels in this repo.

| Name | Description |
| :--- | :--- |
| `blog` | Relating to the writing of blog posts |
| `bug` | Something isn't working |
| `comms` | Relating to Sarah's contact info or adding press releases to the "Media" page |
| `project` | Adding a new project page |
| `speaking` | Adding info about a speaking engagement to the "Speaking" page |
| `website` | General updates or changes to the website |
| `work-in-progress` | Work in progress |
