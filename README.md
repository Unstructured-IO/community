## Community

Welcome to the Unstructured Community!

We are building an ecosystem of preprepocessing pipeline tools for Data Scientists
and Data Engineers so they may quickly work through the challenge of extracting
structured data from unstructured raw documents.

## Getting Started

Unstructured's open-source packages currently target Python 3.8. If you are using or contributing
to Unstructured code, we encourage you to work with Python 3.8 in a virtual environment. You can
use the following instructions to get up and running with a Python 3.8 virtual environemtn
with `pyenv-virtualenv`:

#### Mac / Homebrew

1. Install `pyenv` with `brew install pyenv`.
2. Install `pyenv-virtualenv` with `brew install pyenv-virtualenv`
3. Next, use the following commands to add the `pyenv-virtualenv` startup code to `~/.bash_profile`

```
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
```

4. Install Python 3.8 by running `pyenv install 3.8.14`.
5. Create and activate a virtual environment by running:

```
pyenv virtualenv 3.8.14 unstructured
pyenv activate unstructured
```

You can changed the name of the virtual environment from `unstructured` to another name if you're
creating a virtual environment for a pipeline. For example, if you're a creating a virtual
environment for the SEC preprocessing, you can run `pyenv virtualenv 3.8.14 sec`.

#### Linux

1. Run `git clone https://github.com/pyenv/pyenv.git ~/.pyenv` to install `pyenv`
2. Run `git clone https://github.com/pyenv/pyenv-virtualenv.git ~/.pyenv/plugins/pyenv-virtualenv`
   to install `pyenv-virtualenv` as a `pyenv` plugin.
3. Use the following commands to add the `pyenv-virtualenv` startup code to `~/.bashrc`:

```
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
```

4. Follow steps 4 and 5 from the Mac/Homebrew instructions.

## Contributions

We welcome contributions! This repo's [Issues](https://github.com/Unstructured-IO/community-tasks/issues)
tracks bugs, features, and enhancement requests for [Unstructured's open source repositories](https://github.com/Unstructured-IO/).

## Key Concepts

### Bricks

Bricks are the "blocks" or Python functions from which preprocessing pipelines are made, and are organized
in the [Unstructured](https://github.com/Unstructured-IO/unstructured) library. These collectively form
the Swiss Army knife that Python developers can use to extract structured data from raw documents into
the format that they want. They may be used independently of any other Unstructured repos  under the
terms of its license. `pip install unstructured` and you are good to go.

### Preprocessing pipeline APIs

A preprocessing pipeline API (or just "pipeline API") is a notebook that includes a Python function
capable of transforming a raw document to structured data. By following the [documented conventions](Pipelines-and-APIs.md),
FastAPI APIs may be auto-generated from a pipeline notebook.

See [pipeline-sec-filings](https://github.com/Unstructured-IO/pipeline-sec-filings/) for an example repo
includes a preprocessing pipeline API and auto-generated FastAPI.

### Developer tools for generating FastAPIs

The [unstructured-api-tools](https://github.com/Unstructured-IO/unstructured-api-tools) library includes the
tooling required to create FastAPIs from pipeline notebooks.
