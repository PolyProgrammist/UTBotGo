from ubuntu:20.04
# Downloading necessary packages
run apt-get update; \
    apt-get install -y tzdata
run apt-get update; \
    apt-get install -y \
        git cmake gcc g++ m4 ninja-build python python3 python3-pip wget unzip \
        libsqlite3-dev zlib1g zlib1g-dev
# Downloading Z3
run wget -q -O z3.zip \
        https://github.com/Z3Prover/z3/archive/df8f9d7dcb8b9f9b3de1072017b7c2b7f63f0af8.zip; \
    unzip -q z3.zip; \
    mv z3-df8f9d7dcb8b9f9b3de1072017b7c2b7f63f0af8 z3
# Building and installing Z3
run cd z3; \
    python scripts/mk_make.py; \
    cd build; \
    make; \
    make install
# Downloading llvm (only necessary commit)
run wget -q -O llvm-project.zip \
        https://github.com/llvm/llvm-project/archive/2c5590adfe09283d0c400507be61198b63ce5063.zip; \
    unzip -q llvm-project.zip; \
    mv llvm-project-2c5590adfe09283d0c400507be61198b63ce5063 llvm-project
# Downloading gollvm
run cd llvm-project/llvm/tools; \
    git clone https://go.googlesource.com/gollvm; \
    cd gollvm; \
    git checkout f17ba8c7708356ef447525e05cd6f2770845c7d7; \
    git clone https://go.googlesource.com/gofrontend; \
    cd gofrontend; \
    git checkout 3e9f4ee16683883ccfb8661d99318c74bb7a4bef; \
    cd ../libgo; \
    git clone https://github.com/libffi/libffi.git; \
    cd libffi; \
    git checkout b60d4fc7bb4419021bf566e09de55ed1d9b041db; \
    cd ..; \
    git clone https://github.com/ianlancetaylor/libbacktrace.git; \
    cd libbacktrace; \
    git checkout d0f5e95a87a4d3e0a1ed6c069b5dae7cbab3ed2a
# Building llvm and gollvm
run cd llvm-project; \
    mkdir build; \
    cmake \
        -S llvm \
        -B build \
        -G Ninja \
        -DLLVM_ENABLE_PROJECTS="clang" \
        -DCMAKE_BUILD_TYPE=Release \
        -DLLVM_ENABLE_ASSERTIONS=On; \
    cd build; \
    ninja
# Downloading klee
run wget -q -O klee.zip \
        https://github.com/klee/klee/archive/dfec2ced4e90e745631331ee087f19224df864a0.zip; \
    unzip -q klee.zip; \
    mv klee-dfec2ced4e90e745631331ee087f19224df864a0 klee
copy patches/ktest-tool.patch patches/ktest-tool.patch
run patch /klee/tools/ktest-tool/ktest-tool < patches/ktest-tool.patch
# Building klee
run mkdir klee/build; \
    cd klee/build; \
    cmake \
        -DCMAKE_BUILD_TYPE=Release \
        -DENABLE_KLEE_ASSERTS=ON \
        -DENABLE_SOLVER_Z3=ON \
        -DLLVM_CONFIG_BINARY=/llvm-project/build/bin/llvm-config \
        -DLLVMCC=/llvm-project/build/bin/clang \
        -DLLVMCXX=/llvm-project/build/bin/clang++ \
        -DENABLE_TCMALLOC=OFF \
        -DENABLE_DOXYGEN=OFF \
        ..; \
    make
# Installing llvm, gollvm and klee
env PATH="$PATH:/llvm-project/build/bin:/klee/build/bin"
# Updating sources of gollvm and gofrontend
copy patches/gollvm.patch patches/gollvm.patch
copy patches/gofrontend.patch patches/gofrontend.patch
run cd llvm-project/llvm/tools/gollvm; \
    git apply /patches/gollvm.patch
run cd llvm-project/llvm/tools/gollvm/gofrontend; \
    git apply /patches/gofrontend.patch
# Building and installing go
run cd llvm-project; \
    cmake \
        -S llvm \
        -B build \
        -G Ninja \
        -DCMAKE_INSTALL_PREFIX=/go \
        -DCMAKE_BUILD_TYPE=Release \
        -DLLVM_USE_LINKER=gold \
        -DGOLLVM_BUILD_LLVM_BC_LIBRARY=On; \
    cd build; \
    ninja install-gollvm
env LD_LIBRARY_PATH=/go/lib64
env PATH="/go/bin:$PATH"
# Installing python libraries
run pip3 install pyyaml
# Updating make
run wget https://ftp.gnu.org/gnu/make/make-4.3.tar.gz; \
    tar xf make-4.3.tar.gz; \
    cd make-4.3; \
    ./configure; \
    make; \
    make install
