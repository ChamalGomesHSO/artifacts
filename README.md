## Artifacts

This repo is created to easily build binaries related to common LLM/NLP libraries. Before doing a release for any package, refer to following key steps, 
1. Run `git submodule update --remote --merge` to update all submodules.
2. Update the respective workflow to choose your preferred CUDA/Python/OS version.
3. Create a new release, including a description with the latest source package version.
4. Run the desired workflow file.

## Llama-cpp-python

Provide custom-built CUDA-compatible wheels that override the default CMAKE args for your preferred choice of GPU.

**References:**
- Source: [llama-cpp-python](https://github.com/abetlen/llama-cpp-python)
- CMAKE documentation: [llama.cpp build guide](https://github.com/ggml-org/llama.cpp/blob/master/docs/build.md#unified-memory)
- Pre-built wheels are generated using the `llama-cpu.yaml` or `llama-cuda.yaml` workflows
- CUDA Support & Architecture Compatibility for different GPUs:
       ```bash
       -DCMAKE_CUDA_ARCHITECTURE=70;75;80
       ```

| Compute Capability | CUDA Architecture | GPUs                        | Supported CUDA Versions                | Azure Support |
|--------------------|------------------|-----------------------------|-----------------------------------------|---------------|
| **sm_50**          | Maxwell          | GTX 750, Tesla M40          | ≤ 11.x *(deprecated)*                   | ❌            |
| **sm_60 / sm_61**  | Pascal           | GTX 1080, Tesla P100        | ≤ 12.2 *(deprecated in 12.3+)*          | ❌            |
| **sm_70 / sm_72**  | Volta            | Tesla V100, Jetson AGX Xavier| ✅ **12.8.x** *(deprecated in 13.x)*   | ❌            |
| **sm_75**          | Turing           | RTX 2080, T4                | ✅ **12.8.x**                           | ✅            |
| **sm_80**          | Ampere           | A100                        | ✅ **12.8.x**                           | ✅            |
| **sm_86**          | Ampere           | RTX 3090, A100              | ✅ **12.8.x**                           | ✅            |
| **sm_89 / sm_90**  | Ada / Hopper     | RTX 4090, H100              | ✅ 12.8.x and 13.x                      | ✅            |


## Negspacy 

Negspacy is a spaCy pipeline component for negation detection. Wheels are built for easy use, as the latest GitHub release is not available on PyPI at time of writing.

**References:**
- Source: [negspacy repository](https://github.com/jenojp/negspacy)
- Pre-built wheels are generated using the `negspacy-build.yaml` workflow
- The negspacy repository is included as a git submodule in the vendor folder

## Sundry NLP artifacts 

All artifacts are uploaded as binaries to the release tagged v0.0.1

- Scipy:  [en_core_sci_md-0.5.4.tar.gz](https://s3-us-west-2.amazonaws.com/ai2-s2-scispacy/releases/v0.5.4/en_core_sci_md-0.5.4.tar.gz)
- Open-MPI (For multi-GPU processing): [openmpi_4.1.6.orig.tar.xz](http://archive.ubuntu.com/ubuntu/pool/universe/o/openmpi/openmpi_${OPENMPI_VERSION}.orig.tar.xz)


## GCC Compiler Compatibility Issues

If you come across compatibility issues for llama_cpp_python CPU release, recommend that you update the workflow to include repair for following shared files.

This tends to happen when you build the image on ubuntu-latest (25.04/24.04) which has the latest C compiler, making it unable to be used on outdated Linux machines (e.g., Ubuntu 20). By default, we've omitted 32-bit builds as they would require additional shared files to be repaired.

```yaml
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
