# Custom Markdownlint rules

Our current setup makes it hard to add rules via NPM.
Therefore we copied the rule into our Git Repository.
This way they are working within IDEs and our makefile docker images.

## Rules

### Max one sentence per line

A custom markdownlint rule to enforce a maximum of one sentence per line.

- [GitHub](https://github.com/aepfli/markdownlint-rule-max-one-sentence-per-line)
- [NPM.js Registry](https://www.npmjs.com/package/markdownlint-rule-max-one-sentence-per-line)
- File: [max-one-sentence-per-line.js](max-one-sentence-per-line.js)
- Version: 0.0.2

### GitHub Admonitions

Ensure proper GitHub admonition.

- [GitHub](https://github.com/aepfli/markdownlint-rule-github-admonition)
- [NPM.js Registry](https://www.npmjs.com/package/markdownlint-rule-github-admonition)
- File: [admonition.js](admonition.js)
- Version: 0.0.1
