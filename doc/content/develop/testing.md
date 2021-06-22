# Running the tests

To enable the regression tests one must configure the build with testing enabled. To check for any known failures you may check the [dashboard][]

```bash
cmake -DBUILD_TESTING=ON ...
```

To run the regression tests locally, from the build directory issue ctest command.

```bash
cd build_dir
ctest
```

To run tests based on the features it tests, most tests have labels. Labels are generally automatically assigned on test registration.

```bash
ctest -L PARALLEL|CATALYST
```

Label List:
 - `ADIOS1`
 - `ADIOS2`
 - `ASCENT`
 - `CATALYST`
 - `GENERAL`
 - `HDF5`
 - `PARALLEL`
 - `PYTHON`
 - `SHELL`
 - `VTK_IO`

# Adding regression tests using CTest

### senseiAddTest

Tests are added by calling the CMake function *senseiAddTest*. This function
encapsulates the common scenarios needed to compile, link, and run tests in
serial, and parallel; and configures CTest to flag absolute test failures
(those retrurning non-zero exit code) as well as tests that print the word
`ERROR` (case insensitive). Given that not all tests require a compile/link
step(eg Python based tests) arguments related to these steps are optional.

```CMake
function senseiAddTest(<test_name>
  COMMAND <test_command>...
  [PARALLEL <N>]
  [PARALLEL_SHELL <N>]
  [SOURCES  <source>...
   [EXEC_NAME <name>]
   [LIBS <libraries>...]]
  [FEATURES <feature>...]
  [REQ_SENSEI_DATA]
  [PROPERTIES <property>...])
```

Arguments to *senseiAddTest* are:

| Parameter          | Aruguments          | Description                                                                               |
| ------------------ | ------------------- | ----------------------------------------------------------------------------------------- |
|                    |    `<test_name>`    | *(required)* a unique test name                                                           |
| `COMMAND`          |  `<test_command>`   | *(required)* a command line including all arguments (except MPI) needed to run the test                |
| `PARALLEL`         |       `<N>`         | *(optional)* specify the test should use an MPI launch command to run `test_command` <br/> (ie. `mpiexec <mpi-pre-args> -np <N> <mpi-post-args> <test_command>`)|
| `PARALLEL_SHELL`   |       `<N>`         | *(optional)* Export environment variables for MPI with the specified number of processes (`MPI_LAUNCHER`/`MPI_PROCESSES`/`MPI_PROCESSES_HALF`) |
| `SOURCES`          |    `<source>...`    | *(optional)* a list of source code files needed to compile the test executable. This will trigger the compilation of the source and only needs to appear once per test executable. |
| `EXEC_NAME`        | `<executable_name>` | *(optional)* name to compile the test executable to. If not given then the test `<name>` is used. Only used when `SOURCES` is present.
| `LIBS`             |      `<lib>...`     | *(optional)* a list of additional libraries needed to link the test executable. Only used when `SOURCES` is present. |
| `FEATURES`         |    `<feature>...`   | *(optional)* a list of strings used to determine if the test should be enabled or not. For example calling *senseiAddTest* with `FEATURES PYTHON ADIOS2` will enable the test only when both `ENABLE_PYTHON` and `ENABLE_ADIOS2` 1 features are enabled. Note that this argument expects *strings* that have an `ENABLE_` prefix |
| `REQ_SENSEI_DATA`  |                     | *(optional)* a flag that indicates SENSEI's test data repo is needed to run the test. If the data repo is not found then the test will be disabled. |
| `PROPERTIES`       |   `<property>...`   | *(optional)* a list of [test  properties](https://cmake.org/cmake/help/v3.6/manual/cmake-properties.7.html\#test-properties) for this test. |

### examples
#### C++

##### Serial

Here is an example of a serial test that needs to be compiled in C++ and depends on the SENSEI test data repository for a baseline.

```CMake
senseiAddTest(testHistogramSerial
  COMMAND testHistogram EXEC_NAME testHistogram
  SOURCES testHistogram.cpp LIBS sensei)
```

##### Parallel

Here is a parallel version of the test. Note that it makes use of the above test's executable and therefore compiles nothing.

```CMake
senseiAddTest(testHistogramParallel
  COMMAND testHistogram
  PARALLEL ${TEST_NP})
```

#### Python

Here is an example of a Python test.

```CMake
senseiAddTest(testMeshMetadata
    COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/testMeshMetadata.py
  FEATURES PYTHON)
```

#### Shell Script

Here is an example of a scripted test that uses MPI.

```CMake
senseiAddTest(testReadWrite
    COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/testRWDriver.sh
            --src-dir ${CMAKE_CURRENT_SOURCE_DIR}
            --file TestFile
    PARALLEL_SHELL ${TEST_NP}
    PROPERTIES
      ENVIRONMENT
        PYTHON=${PYTHON_EXECUTABLE})
```

And in the script run the commands using the exported PARALLEL environment

```bash
# Parse the command line arguments....

# Launch the writer in the background
${MPI_LAUNCHER} ${PYTHON} ${srcdir}/testWrite.py \
  --file ${file} \
  ${MPIEXEC_POSTFLAGS} &

# Launch the reader while the writer is stil working
${MPI_LAUNCHER} ${PYTHON} ${srcdir}/testRead.py \
  --file ${file} \
  ${MPIEXEC_POSTFLAGS}
```

[dashboard]: https://cdash.nersc.gov/index.php?project=SENSEI
