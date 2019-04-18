# Git basic commands

## Table of Contents

- [Git basic commands](#git-basic-commands)
  - [Table of Contents](#table-of-contents)
    - [.gitignore](#gitignore)
    - [Pull Request](#pull-request)
    - [GitHub flow](#github-flow)
    - [CONTRIBUTING.md](#contributingmd)
      - [Developer workflow](#developer-workflow)
      - [Reviewer workflow](#reviewer-workflow)
      - [Naming convention](#naming-convention)
      - [Commit Message Format](#commit-message-format)

### .gitignore

Don't commit everything. Only sources files or assets needed to build the artifact.

Story telling on ours #failure-parties.

### Pull Request

Workshops partecipants:

| name       | surname | mail                          |
| ---------- | ------- | ----------------------------- |
| Alessandro | Poli    | alessandro.poli@mondora.com   |
| Manuel     | Serra   | manuel.serra@mondora.com      |
| Andrea     | Quirini | andrea.quirini@top-network.it |

Exercise: make your personal pull request and fill this table.

### GitHub flow

- Fork
- Develop
- Commit
- Push
- Merge

### CONTRIBUTING.md

Many flavours of workflows exist, make it yours. One example (from a `React/js` codebase).

#### Developer workflow

- checkout and pull the `master` branch
- make a feature branch for the feature you wish to develop (if possible, thename of the feature branch should match the key or the jira issue/storyassociated with it)
- commit your work on the feature branch and push it to the remote
- if needed, re-align your feature branch with `master`
- make a pull request to `master`
- ask a reviewer to review and merge the pull request

#### Reviewer workflow

- the feature works as expected (according the user story, or to otherrequirements)
- the code is decent
- there are unit tests for the feature
- all test pass
- linting passes

#### Naming convention

To avoid any problems with case sensitive/insensitive file system `kebab-case`
convention is used.

```bash
// Non valid
Components
├── MyComponent.jsx
└── superUtils.js

// Valid
components
├── my-component.jsx
└── super-utils.js
```

`kebab-case` has also the side effect to be homogeneous with npm dependencies imports.

#### Commit Message Format

Use conventional commits format.

See here for further documentation: https://www.conventionalcommits.org/
