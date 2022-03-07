+++
title = "Deploying to PyPI with sr.ht"
date = 2019-04-10
tags = ["python"]
draft = false
+++

I recently started to use [builds.sr.ht](https://builds.sr.ht) (part of the [sourcehut.org](https://sourcehut.org)
stack) to run continuous integration for a small python
project. The project eventually reached a releasable state, and I
wanted to automate that task. I had never deployed a project to
[PyPI](https://pypi.org/), but after learning more about the builds.sr.ht CI system
(specifically the ability to use secrets) I decided to give it a
shot. Running simple unit tests with builds.sr.ht was super easy,
so I hoped adding PyPI deployment would be pretty simple -- it
definitely is.


## Setting up your secret PyPI credentials {#setting-up-your-secret-pypi-credentials}

First create a temporary file (that will be our `pypirc` file,
[read more here](https://packaging.python.org/guides/distributing-packages-using-setuptools/#uploading-your-project-to-pypi) if this doesn't sound familiar) with the following
contents:

```toml
[pypi]
username = your_username
password = your_password
```

Travel to <https://builds.sr.ht/secrets> and add it. Just give it a
name, select the File type, make the path `~/.pypirc`, make the
permission mode `600`, and upload it (get rid of the copy on your
local file system if you don't want to keep a local `~/.pypirc`).


## The build manifest {#the-build-manifest}

In the `tasks` section of the build manifest we're just going to
add a `deploy` step. In the `build` step, where I setup my python
environment, I make sure to install `twine` (necessary for
uploading to PyPI).

```yaml
image: ...
packages:
  - ...
sources:
  - ...
secrets:
  - abcdefgh-ijkl-lmno-pqrx-tuvwxyz12345
tasks:
  - build: |
      python -m venv cienv
      source cienv/bin/activate
      pip install pytest twine setuptools wheel
      cd myproject
      pip install .
  - test: |
      ...
  - deploy: |
      source cienv/bin/activate
      cd myproject
      python setup.py sdist bdist_wheel
      python .ci-scripts/srht-pypi.py
```

For this example I'm building both a source distribution (`sdist`)
and a wheel (`bdist_wheel`) for the toy project[^fn:1]. In the
repository I have a directory called `.ci-scripts` with a script
to handle the PyPI upload. The script ensures that I only upload
to PyPI if the repository git hash is on a tag, and the name of
the tag is the same as the version of the python project (the
versions and tags are formatted `X.Y.Z`). Here are the contents of
that script:

```python
import subprocess
import myproject.version
import sys

def main():
    res = subprocess.run(["git", "describe"], stdout=subprocess.PIPE)
    describe_out = res.stdout.decode("utf-8").split("-")
    print(describe_out)
    if len(describe_out) > 1:
        return 0
    elif myproject.version.version == describe_out[0].strip():
        res = subprocess.run("twine upload dist/*", shell=True)
        return res.returncode
    else:
        return 0;

if __name__ == "__main__":
    main()
```

[^fn:1]: to compile extension modules, builds.sr.ht jobs might not be
    the best choice for wheels. The [`cibuildwheel`](https://github.com/joerick/cibuildwheel) package is worth
    reading about. It is possible to spin up docker containers in a
    sr.ht build, but I don't have a strong handle on that procedure.
