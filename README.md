# vendored
Storage of vendored packages for use within our deployment pipeline


# How to build deadsnakes packages on a Mac (for arm64 packages)
In order to build deadsnakes for focal (the current version of Ubuntu we're using in development )

```
mkdir deadsnakes
cd deadsnakes
git clone https://github.com/deadsnakes/runbooks
git clone https://github.com/deadsnakes/py3.12
git clone https://github.com/deadsnakes/py3.11

# Versions immediately before the removal of Focal
git -C runbooks/ checkout 11ff34425a759^

git -C py3.11 checkout d42462279826a56bcc747ea51210bebf31cc535a^
git -C py3.12 checkout 76bc0e16d6b88^

# Build image for arm64
cd ../runbooks/dockerfiles
docker build . -f Dockerfile.tools -t local:tools

# Remove amd64 from build-essential in Dockerfile.focal
# TODO: turn this into a sed script or something.
runbooks/dockerfiles/Dockerfile.focal

# Build the focal image for arm64.
docker build . -f Dockerfile.focal -t local:deadsnakes-focal

# Edit tools file to use the local docker image built for arm64.
# TODO: turn this into a sed script or something.
runbooks/tools
ghcr.io/deadsnakes/tools > local:tools

# Edit build file to use local docker image.
# TODO: turn this into a sed script or something.
runbooks/build
ghcr.io/deadsnakes/{dist} > local:deadnsakes-{dist}

# A gpg key is needed to build the images.
# If you don't have one, create one using this command.
gpg --gen-key

# Build the Python 3.11 packages.
cd py3.11
export DEBFULLNAME="..."
export DEBEMAIL="..."
../runbooks/meta-gbp materialize focal

cd ../../py3.11

../runbooks/meta-gbp build

# Deb files should be in the dist/ directory.

# Repeat the above instructions for py3.12
```
