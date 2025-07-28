# LLM artifacts 

All artifacts are uploaded as binaries to the release tagged v0.0.1

- Scipy:  [en_core_sci_md-0.5.4.tar.gz](https://s3-us-west-2.amazonaws.com/ai2-s2-scispacy/releases/v0.5.4/en_core_sci_md-0.5.4.tar.gz)
- Open-MPI (For multi-GPU processing): [openmpi_4.1.6.orig.tar.xz](http://archive.ubuntu.com/ubuntu/pool/universe/o/openmpi/openmpi_${OPENMPI_VERSION}.orig.tar.xz)

# Llama-cpp-python custom pre-built wheels

- Current pre-built wheels do not support tesla v100 due to CMAKE args settings used within llama-cpp-python CUDA wheel release process. Uploaded artifact aims to  overrid the said CMAKE args and build CUDA compatible pre-built wheels. (ref: git@github.com:abetlen/llama-cpp-python.git)
- See more info on CMAKE ARGs being used here https://github.com/ggml-org/llama.cpp/blob/7f97599581fcf0c37432dd3b1f503b91bed97695/docs/build.md#unified-memory
- Also aims to fixe a musl linux bug in the llama-cpp-python CPU pre-built wheel as well.

# Instructions for a new CUDA release
- Run "git submodule update --remote --merge" to update to latest llama_cpp version and commit/push the change
- Update workflow to chose your preferred CUDA/Python/OS version
- Create new release with release description including the latest llama_cpp version for posterity
- Run CUDA workflow on the latest release tag


# GCC Compiler Compatibility Issues

If you come across compatibility issues for CPU release, recommend that you update the workflow to include repair for following shared files

This tends to happen when you build the image on ubuntu-latest (25.04/24.04) which has latest C compiler making it unable to be used on outdated linux machines E.g. Ubuntu 20. By default have ommited 32 bit build, it would require further shared files to be repaired. (vide: https://github.com/JohnnyTeutonic)

```
       - name: Build wheels
         uses: pypa/cibuildwheel@v2.21.1
         env:
           CIBW_SKIP: "*manylinux_i686* *musllinux* pp*"
           #CIBW_REPAIR_WHEEL_COMMAND_LINUX: "auditwheel repair --exclude libllama.so --exclude libggml.so --exclude libggml-base.so --exclude libggml-cpu.so {wheel} -w {dest_dir}"
           CIBW_REPAIR_WHEEL_COMMAND: ""
           CIBW_BUILD: "cp312-*"
           CIBW_BUILD_FRONTEND: "build[uv]"
         with:
           package-dir: ./vendor/llama-cpp-python
           output-dir: ./dist

      - uses: actions/upload-artifact@v4
        with:
          name: wheels
          path:  ./dist/*.whl
```
