# About

conan package for cling https://root.cern.ch/root/htmldoc/guides/users-guide/Cling.html

```bash
# Uses:
    llvm_repo_url = "http://root.cern.ch/git/llvm.git"
    cling_repo_url = "http://root.cern.ch/git/cling.git"
    clang_repo_url = "http://root.cern.ch/git/clang.git"
```

## LICENSE

MIT for conan package. Packaged source uses own license, see https://releases.llvm.org/2.8/LICENSE.TXT and https://github.com/root-project/root/blob/master/LICENSE

## Local build

```bash
export CC=clang-6.0
export CXX=clang++-6.0

$CC --version
$CXX --version

conan remote add conan-center https://api.bintray.com/conan/conan/conan-center False

export PKG_NAME=cling_conan/master@conan/stable

(CONAN_REVISIONS_ENABLED=1 \
    conan remove --force $PKG_NAME || true)

CONAN_REVISIONS_ENABLED=1 \
    CONAN_VERBOSE_TRACEBACK=1 \
    CONAN_PRINT_RUN_COMMANDS=1 \
    CONAN_LOGGING_LEVEL=10 \
    GIT_SSL_NO_VERIFY=true \
    conan create . conan/stable -s build_type=Debug --profile clang --build missing -o openssl:shared=True

CONAN_REVISIONS_ENABLED=1 \
    CONAN_VERBOSE_TRACEBACK=1 \
    CONAN_PRINT_RUN_COMMANDS=1 \
    CONAN_LOGGING_LEVEL=10 \
    conan upload $PKG_NAME --all -r=conan-local -c --retry 3 --retry-wait 10 --force
```

## Docker build with `--no-cache`

```bash
export MY_IP=$(ip route get 8.8.8.8 | sed -n '/src/{s/.*src *\([^ ]*\).*/\1/p;q}')
sudo -E docker build \
    --build-arg PKG_NAME=cling_conan \
    --build-arg PKG_CHANNEL=conan/stable \
    --build-arg PKG_UPLOAD_NAME=cling_conan/master@conan/stable \
    --build-arg CONAN_EXTRA_REPOS="conan-local http://$MY_IP:8081/artifactory/api/conan/conan False" \
    --build-arg CONAN_EXTRA_REPOS_USER="user -p password1 -r conan-local admin" \
    --build-arg CONAN_UPLOAD="conan upload --all -r=conan-local -c --retry 3 --retry-wait 10 --force" \
    --build-arg BUILD_TYPE=Debug \
    -f cling_conan_source.Dockerfile --tag cling_conan_repoadd_source_install . --no-cache

sudo -E docker build \
    --build-arg PKG_NAME=cling_conan \
    --build-arg PKG_CHANNEL=conan/stable \
    --build-arg PKG_UPLOAD_NAME=cling_conan/master@conan/stable \
    --build-arg CONAN_EXTRA_REPOS="conan-local http://$MY_IP:8081/artifactory/api/conan/conan False" \
    --build-arg CONAN_EXTRA_REPOS_USER="user -p password1 -r conan-local admin" \
    --build-arg CONAN_UPLOAD="conan upload --all -r=conan-local -c --retry 3 --retry-wait 10 --force" \
    --build-arg BUILD_TYPE=Debug \
    -f cling_conan_build.Dockerfile --tag cling_conan_build_package_export_test_upload . --no-cache

# OPTIONAL: clear unused data
sudo -E docker rmi cling_conan_*
```

## How to run single command in container using bash with gdb support

```bash
# about gdb support https://stackoverflow.com/a/46676907
docker run --cap-add=SYS_PTRACE --security-opt seccomp=unconfined --rm --entrypoint="/bin/bash" -v "$PWD":/home/u/project_copy -w /home/u/project_copy -p 50051:50051 --name DEV_cling_conan cling_conan -c pwd
```
