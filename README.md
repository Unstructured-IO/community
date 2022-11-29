<h3 align="center">
  <img
    src="https://raw.githubusercontent.com/Unstructured-IO/unstructured/main/img/unstructured_logo.png"
    height="200"
  >
</h3>

<div align="center">

  <a href="https://github.com/Unstructured-IO/unstructured/blob/main/LICENSE.md">![https://pypi.python.org/pypi/unstructured/](https://img.shields.io/pypi/l/unstructured.svg)</a>
  <a href="https://pypi.python.org/pypi/unstructured/">![https://pypi.python.org/pypi/unstructured/](https://img.shields.io/pypi/pyversions/unstructured.svg)</a>
  <a href="https://GitHub.com/unstructured-io/unstructured/graphs/contributors">![https://GitHub.com/unstructured-io/unstructured.js/graphs/contributors](https://img.shields.io/github/contributors/unstructured-io/unstructured)</a>
  <a href="https://github.com/Unstructured-IO/unstructured/blob/main/CODE_OF_CONDUCT.md">![code_of_conduct.md](https://img.shields.io/badge/Contributor%20Covenant-2.1-4baaaa.svg) </a>
  <a href="https://GitHub.com/unstructured-io/unstructured/releases">![https://GitHub.com/unstructured-io/unstructured.js/releases](https://img.shields.io/github/release/unstructured-io/unstructured)</a>
  <a href="https://pypi.python.org/pypi/unstructured/">![https://github.com/Naereen/badges/](https://badgen.net/badge/Open%20Source%20%3F/Yes%21/blue?icon=github)</a>
</div>

<h3 align="center">
  Open-Source Pre-Processing Tools for Unstructured Data
</h3>

Welcome to the **Unstructured Community!** :blush:

We are building an ecosystem of preprocessing pipeline tools for Data Scientists
and Data Engineers, so they may quickly work through the challenge of extracting
structured data from unstructured raw documents.

## :coffee: Getting Started

Unstructured's open-source packages currently target Python 3.8. If you are using or contributing
to Unstructured code, we encourage you to work with Python 3.8 in a virtual environment. You can
use the following instructions to get up and running with a Python 3.8 virtual environment
with `pyenv-virtualenv`:

#### Mac / Homebrew

1. Install `pyenv` with `brew install pyenv`.
2. Install `pyenv-virtualenv` with `brew install pyenv-virtualenv`
3. Follow the instructions [here](https://github.com/pyenv/pyenv#user-content-set-up-your-shell-environment-for-pyenv)
   to add the `pyenv-virtualenv` startup code to your terminal profile.
4. Install Python 3.8 by running `pyenv install 3.8.15`.
5. Create and activate a virtual environment by running:

```
pyenv virtualenv 3.8.15 unstructured
pyenv activate unstructured
```

You can changed the name of the virtual environment from `unstructured` to another name if you're
creating a virtual environment for a pipeline. For example, if you're a creating a virtual
environment for the SEC preprocessing, you can run `pyenv virtualenv 3.8.15 sec`.

#### Linux

1. Run `git clone https://github.com/pyenv/pyenv.git ~/.pyenv` to install `pyenv`
2. Run `git clone https://github.com/pyenv/pyenv-virtualenv.git ~/.pyenv/plugins/pyenv-virtualenv`
   to install `pyenv-virtualenv` as a `pyenv` plugin.
4. Follow steps 3-5 from the Mac/Homebrew instructions.

## :open_hands: Contributions

We welcome contributions! This repo's [Issues](https://github.com/Unstructured-IO/community-tasks/issues)
tracks bugs, features, and enhancement requests for [Unstructured's open source repositories](https://github.com/Unstructured-IO/).

When contributing, please follow our [Contributing to Unstructured](CONTRIBUTING.md) guidelines. Thank you!

## :green_book: Key Concepts

### :bricks: Bricks

Bricks are the "blocks" or Python functions from which preprocessing pipelines are made, and are organized
in the [Unstructured](https://github.com/Unstructured-IO/unstructured) library. These collectively form
the Swiss Army knife that Python developers can use to extract structured data from raw documents into
the format that they want. They may be used independently of any other Unstructured repos  under the
terms of its license. `pip install unstructured` and you are good to go.

### :small_blue_diamond: Preprocessing pipeline APIs

A preprocessing pipeline API (or just "pipeline API") is a notebook that includes a Python function
capable of transforming a raw document to structured data. By following the [documented conventions](Pipelines-and-APIs.md),
FastAPI APIs may be auto-generated from a pipeline notebook.

See [pipeline-sec-filings](https://github.com/Unstructured-IO/pipeline-sec-filings/) for an example repo
includes a preprocessing pipeline API and auto-generated FastAPI.

### :nut_and_bolt: Developer tools for generating FastAPIs

The [unstructured-api-tools](https://github.com/Unstructured-IO/unstructured-api-tools) library includes the
tooling required to create FastAPIs from pipeline notebooks.
