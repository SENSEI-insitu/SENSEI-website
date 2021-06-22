# Continuous Integration (CI)

A System has been devised for testing and continuous integration using [GitHub Workflows][workflows]. By leveraging these tools, the SENSEI project is able to ensure the quality of the software when it is integrated into the ECP software stack.

[//]: # "To view the results of these runs navigate your web browser to the [NERSC CDash dashboard](http://cdash.nersc.gov/index.php?project=SENSEI.)"

# Setting up a new CI job

An example using the GitHub Workflows can be found at [os-linux][].

The three main steps of the CI workflow are:

 * [Configure](#Configure)
 * [Build](#Build)
 * [Test](#Test)
 * [Upload Artifacts](#Upload-Artifacts)

## Configure

This step sets the stage for the rest of the downstream steps. Here we configure the build and tests for the OS and features the CI will test. For this, SENSEI uses a collection of modular CMake scripts that build the CMakeCache.txt. The name of the configuration is generally the same as the name of the job. In the example [os-linux](yaml) the job `linux-ecp` using the configuration `linux_ecp`.

The configuration is controlled by two environment variables

 * `CMAKE_CONFIGURATION`: This variable is *required* to be set. An example is `linux_ecp` which is used for the `linux-ecp` [job][os-linux]
 * `CMAKE_GENERATOR`: This sets the generator cmake will use to build, the default generator is "Ninja".
 * `CMAKE_BUILD_TYPE`: This sets the build type, the default build type is "Debug".


```yaml
    - name: Configure
      run: ${{ env.LAUNCHER }} ctest -VV -S .github/ci/ctest_configure.cmake
```

## Build

The build step is where SENSEI is built using all of the features specified by the configuration.

There are some environment variables that may be set for the CI job to modify how the build step performs.

 * `CTEST_MAX_PARALLELISM`: This limits the number of consequtive compilations tasks, for GitHub Actions this currently defaults to "2".

```yaml
   - name: Build
     run: ${{ env.LAUNCHER }} ctest -VV -S .github/ci/ctest_build.cmake
```

## Test

The testing step is where the SENSEI tests are run. This includes the tests that use the miniapps and standalone drivers that test specific bridges. Depending on the features enabled during configuration, certain tests will be enable or disabled accordingly.

A note about GitHub Actions:

Due to that fact that GitHub Actions only provides two(2) cores for running CI it was decided to disable all tests that run with more than two(2) cores. Most of the tests marked `PARALLEL` run with 4 cores, so the current method of disabling tests that use more than two(2) cores is to disable all tests labeld `PARALLEL` if the environment variable `CTEST_MAX_PARALLELISM` < four(4).

```yaml
   - name: Test
     run: ${{ env.LAUNCHER }} ctest -VV -S .github/ci/ctest_test.cmake
```

## Upload Artifacts

This last step is to store the result of the CI. This step is optional because it may not be desirable or needed to keep artifacts of the CI job around.

```yaml
   - name: "Upload Artifacts"
     uses: actions/upload-artifact@v2
     with:
       # Call the compressed artifact the same name as the configuration with the job ID
       # This will help avoid conflicts between CI artifacts
       name: ${{ env.CMAKE_CONFIGURATION }}_${{ env.GITHUB_JOB }}

       path: build          # Save the entire build directory
       retention-days: 1    # Limit the number of days the artifact persists to one day
```

## Create New Workflow

For an example of creating a new CI configuration that uses HDF5 and ADIOS2 and tests on a Linux OS, let's call it `linux_hdf5_adios2`. To get started we need a CMake configure file called `configure_linux_hdf5_adios2.cmake`. This file will be located in the directory `<sensei-source>/.github/ci/`.

The contents of this file will be pulling in the configurations for HDF5, ADIOS2, and Linux. Note the OS configuration always comes last as it will include additional configurations that may customize some settings based on the features that are configured. One example of this is VTK rendering.


`<sensei-source>/.github/ci/configure_hdf5_adios2.cmake`
```CMake
include(${CMAKE_CURRENT_LIST_DIR}/configure_hdf5.cmake)
include(${CMAKE_CURRENT_LIST_DIR}/configure_adios2.cmake)
include(${CMAKE_CURRENT_LIST_DIR}/configure_linux.cmake)
```

Now that the configure file is setup, it is time to add a new job. Because this is a linux job, we will use the `sensei-ci:latest` docker image. This image contains all of the possible dependencies for SENSEI that may be tested in CI. As more features are made available to CI additional jobs will be introduced to build and test them.

 * ADIOS2
 * Catalyst
 * HDF5
 * Python 3
 * VTK

In the os-linux.yml a new job can be added under the `jobs` section of the yaml.

```yaml
  linux-hdf5-adios2:
    runs-on: ubuntu-latest
    container:
      # By using the latest container the CI will automatically update dependencies
      image: ryankrattiger/sensei-ci:latest
      options: --user=root
    env:
      # This launcher is required when using the sensei-ci container since it uses spack as a backend.
      # Eventually this requirement will be removed
      LAUNCHER: .github/ci/docker/spack/launcher.sh

      CMAKE_CONFIGURATION: linux_hdf5_adios2
   steps:
   # GitHub Action to clone sensei and checkout the triggering branch
   - uses: action/checkout@v2

   # Run the Configure/Build/Test steps in order
   - name: Configure
     run: ${{ env.LAUNCHER }} ctest -VV -S .github/ci/ctest_configure.cmake

   - name: Build
     run: ${{ env.LAUNCHER }} ctest -VV -S .github/ci/ctest_build.cmake

   - name: Test
     run: ${{ env.LAUNCHER }} ctest -VV -S .github/ci/ctest_test.cmake

   # (Optional) Upload the CI artifacts
   - name: "Upload Artifacts"
     uses: actions/upload-artifact@v2
     with:
       name: ${{ env.CMAKE_CONFIGURATION }}_${{ env.GITHUB_JOB }}
       path: build
       retention-days: 1
```

[os-linux]: https://github.com/kwryankrattiger/sensei/blob/ryan-ci-test/.github/workflows/os-linux.yml
[workflows]: https://github.com/kwryankrattiger/sensei/tree/ryan-ci-test/.github/workflows
