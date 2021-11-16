# Release Instructions

Before creating a new release carefully consider the version number of the new
release. We are following [Semantic Versioning](https://semver.org/) and
[PEP440](https://www.python.org/dev/peps/pep-0440/).

## Preparing the Required Python Packages

* Install twine for pypi package uploads

  ```sh
  python3 -m pip install --user --upgrade twine
  ```

## Configuring the Access to the Python Package Index (PyPI)

*Note:* This is only necessary for users performing the release process for the
first time.

* Create an account at [Test PyPI](https://packaging.python.org/guides/using-testpypi/).

* Create an account at [PyPI](https://pypi.org/).

* Create a pypi configuration file `~/.pypirc` with the following content (Note:
  `<username>` must be replaced):

  ```ini
  [distutils]
  index-servers =
      pypi
      testpypi

  [pypi]
  username = <username>

  [testpypi]
  repository = https://test.pypi.org/legacy/
  username = <username>
  ```

## Prepare testing the Release

* Fetch upstream changes and create a branch:

  ```sh
  git fetch upstream
  git checkout -b create-new-release upstream/main
  ```

* Get the current version number

  ```sh
  poetry run python -m pontos.version show
  ```

* Update the version number to some alpha version e.g.

  ```sh
  poetry run python -m pontos.version update 1.2.3a1
  ```

## Uploading to the PyPI Test Instance

* Create a source and wheel distribution:

  ```sh
  rm -rf dist build ospd.egg-info
  poetry build
  ```

* Upload the archives in `dist` to [Test PyPI](https://test.pypi.org/):

  ```sh
  twine upload -r testpypi dist/*
  ```

* Check if the package is available at <https://test.pypi.org/project/ospd>.

## Testing the Uploaded Package

* Create a test directory:

  ```sh
  mkdir ospd-install-test
  cd ospd-install-test
  python3 -m venv test-env
  source test-env/bin/activate
  pip install -U pip  # ensure the environment uses a recent version of pip
  pip install --pre -I --extra-index-url https://test.pypi.org/simple/ ospd
  ```

* Check install version with a Python script:

  ```sh
  python3 -c "from ospd import __version__; print(__version__)"
  ```

* Remove test environment:

  ```sh
  deactivate
  cd ..
  rm -rf ospd-install-test
  ```

## Prepare the Release

* Determine new release version number

  If the output is something like  `1.2.3.dev1` or `1.2.3a1`, the new version
  should be `1.2.3`.

* Update to new version number (`<new-version>` must be replaced by the version
  from the last step)

  ```sh
  cd path/to/git/clone/of/ospd
  poetry run python -m pontos.version update <new-version>
  ```

* Update the `CHANGELOG.md` file:
  * Change `[unreleased]` to new release version.
  * Add a release date.
  * Update reference to Github diff.
  * Remove empty sub sections like *Deprecated*.

* Create a git commit:

  ```sh
  git add .
  git commit -m "Prepare release <version>"
  ```

## Performing the Release on GitHub

* Create a pull request (PR) for the earlier commit:

  ```sh
  git push origin
  ```
  Open GitHub and create a PR against <https://github.com/greenbone/ospd>.

* Ask another developer/maintainer to review and merge the PR.

* Once the PR is merged, update the local `main` branch:

  ```sh
  git fetch upstream
  git rebase upstream/main main
  ```

* Create a git tag:

  ```sh
  git tag v<version>
  ```

  Or even a tag signed with a personal GPG key:

  ```sh
  git tag --sign --message "Tagging the <version> release" v<version>
  ```

* Push changes and tag to Github:

  ```sh
  git push --tags upstream
  ```

## Uploading to the 'real' PyPI

* Uploading to PyPI is done automatically by pushing a git tag via CircleCI

* Check if new version is available at <https://pypi.org/project/ospd>.

## Bumping `main` Branch to the Next Version


* Update to a Development Version

  The next version should contain an incremented minor version and a dev suffix
  e.g. 2.3.0.dev1

  ```sh
  poetry run python -m pontos.version update <next-dev-version>
  ```

* Create a commit for the version bump:

  ```sh
  git add .
  git commit -m "Update version after <version> release"
  git push upstream
  ```

## Announcing the Release

* Create a Github release:

  See https://help.github.com/articles/creating-releases/
