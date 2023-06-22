# Install `mkdocs`

Assuming you have Python installed.

```
$(PIP) install mkdocs mkdocs-material
```

where `PIP` is either `pip` or `pip3` depending on your system.

# Compile and View Locally

```
mkdocs serve
```

# Compile and Publish on Github

A Github Action has been setup to publish every `git push` automatically.

but if there is a reason to publish manually, run
```
mkdocs gh-deploy
```
