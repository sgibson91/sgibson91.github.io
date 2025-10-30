# sgibson91.github.io

## Repo Overview

* Blog posts are hosted in `content/blog/YYYY/`

## Building the website locally

You need to have [Hugo](https://gohugo.io/) installed.

### Installing Hugo with Homebrew

```bash
brew install hugo
```

### Creating new content

Make sure this command is run from the root of the directory.

```bash
hugo new path/to/new-content.md
```

### Locally rendering changes

```bash
hugo serve --enableGitInfo -DF
```

## Updating the theme

To pull in any upstream changes to the theme being used as a git submodule, run the following.

```bash
git submodule update --remote --merge
```

## Contributing

Please read the [Contributing Guidelines](./CONTRIBUTING.md) and [Code of Conduct](./CODE_OF_CONDUCT.md) to get started making a contribution.
