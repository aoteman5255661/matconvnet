# Requirements

You will need a sufficiently recent MATLAB version (R2015b or newer) and a compiler with C++11 support (Visual Studio 2015, GCC 4.8, Xcode 7.3.1 or higher). For GPU computation, you will need at least CUDA 7.5  and, optionally, CuDNN v4 or newer. Depending on the specific  MATLAB version, further constraints may apply, some of which are discussed below and in the [FAQ](faq).

# Installing and compiling the library

In order to install the library, follows these steps:

1.  Download and unpack the library source code into a directory of
    your choice. Call the path to this directory `<MatConvNet>`.
2.  [Compile](#compiling) the library.
3.  Start MATLAB and type:

        > run <MatConvNet>/matlab/vl_setupnn

    in order to add MatConvNet to MATLAB's search path.

At this point the library is ready to use. You can test it by using
the command (using MATLAB R2015b or later):

    > vl_testnn

To test GPU support (if you have [compiled it](#gpu)) use instead:

    > vl_testnn('gpu', true)

Note that the second tests runs slower than the CPU version; do not
worry, this is an artifact of the test procedure.

<a name='compiling'></a>
## Compiling

MatConvNet compiles under Linux, Mac, and Windows. This page discusses
compiling MatConvNet using the MATLAB function
[`vl_compilenn`](mfiles/vl_compilenn). While this is the easiest
method,
[the command line or an IDE can be used as well](install-alt.md).

<a name='cpu'></a>
### Compiling for CPU

If this is the first time you compile MatConvNet, consider trying
first the CPU version. In order to do this, use the
[`vl_compilenn`](mfiles/vl_compilenn) command supplied with the
library:

1.  Make sure that MATLAB is
    [configured to use your compiler](http://www.mathworks.com/help/matlab/matlab_external/changing-default-compiler.html).
    In particular, before running `vl_compilenn` do not forget to setup
    `mex` (doing so nce is sufficient) as follows:

    ```
    mex -setup
    mex -setup C++
    ```

    The prerequisites are as follows:

    * For **macOS**, make sure you have Xcode installed. A copy can be obtained for
      free from the Mac App Store. If `mex -setup` returns errors, see
      [troubleshooting](faq#macos).

    * For **Linux**, make sure you have GCC 4.8 and LibJPEG are installed. To
      install LibJPEG in and Ubuntu/Debian-like distributions use:
      ```
      sudo apt-get install build-essential libjpeg-turbo8-dev
      ```
      For Fedora/Centos/RedHat-like distributions use instead:
      ```
      sudo yum install gcc gcc-c++ libjpeg-turbo-devel
      ```
      Older versions of GCC (e.g. 4.7) are not compatible with the C++ code in MatConvNet.

    * For **Windows**, make sure you have Visual Studio 2015 or greater installed.

2.  Open MATLAB and issue the commands:

        > cd <MatConvNet>
        > addpath matlab
        > vl_compilenn

At this point MatConvNet should start compiling. If all goes well, you
are ready to use the library. If not, you can try debugging the
problem by running the compilation script again in verbose mode:

    > vl_compilenn('verbose', 1)

Increase the verbosity level to 2 to get even more information.

> **Remark:** The 'vl_imreadjpeg' tool uses an external image library
> to load images. In macOS and Windows, the default is to use the
> system libraries (Quartz and GDI+ respectively), so this dependency
> is immaterial. In Linux, this tool requires the LibJPEG
> library and the corresponding development files to be installed in
> the system. If needed, the `ImageLibraryCompileFlags` and
> `ImageLibraryLinkFlags` options can be used to adjust the
> compiler and linker flags to match a specific library installation.
> It is also possible to use the `EnableImreadJpeg` option of `vl_compilenn` to
> turn off this feature.

<a name='gpu'></a>
### Compiling the GPU support

To use the GPU-accelerated version of the library, you will need a
NVIDA GPU card with compute capability 2.0 or greater and a copy of
the NVIDIA CUDA toolkit. Officially, MATLAB supports the following
CUDA versions:

| MATLAB    | CUDA toolkit      |
|-----------|-------------------|
| R2017a    | 8.0               |
| R2016b    | 7.5               |
| R2016a    | 7.5               |
| R2015b    | 7.0               |

You can also use the `gpuDevice` MATLAB command to find out MATLAB's
version of the CUDA toolkit. Nevertheless, it is also possible and
often useful to use a more recent version of CUDA than the one
officially supported by MATLAB; this is [explained later](#nvcc).

> **macOS**. CUDA is typically one or two step behind the latest
> Xcode. For example, CUDA 8.0 requires Xcode 7.3.1 instead of more
> recent versions.
>
> Using an older version of Xcode is explained [here](faq#macos).
>
> It can be helpful to consult the
> [CUDA Installation Guide for Mac](http://docs.nvidia.com/cuda/pdf/CUDA_Installation_Guide_Mac.pdf).

Assuming that there is only a single copy of the CUDA toolkit
installed in your system and that it matches MATLAB's version, compile
the library with:

    > vl_compilenn('enableGpu', true)

If you have multiple versions of the CUDA toolkit, or if the script
cannot find the toolkit for any reason, specify the path to the CUDA
toolkit explicitly. For example, on macOS this may look like:

    > vl_compilenn('enableGpu', true, 'cudaRoot', '/Developer/NVIDIA/CUDA-8.0')

Once more, you can use the `verbose` option to obtain more information
if needed.

<a name='nvcc'></a>
### Using an unsupported CUDA toolkit version

MatConvNet can be compiled to use a more recent version of the CUDA
toolkit than the one officially supported by MATLAB. While this may
cause unforeseen issues (although none is known so far), it is
necessary to use [recent libraries such as cuDNN](#cudnn).

Compiling with a newer version of CUDA requires using the
`cudaMethod,nvcc` option. For example, on macOS this may look like:

    > vl_compilenn('enableGpu', true, ...
                   'cudaRoot', '/Developer/NVIDIA/CUDA-8.0', ...
                   'cudaMethod', 'nvcc')

Note that at this point MatConvNet MEX files are linked *against the
specified CUDA libraries* instead of the one distributed with
MATLAB. Hence, in order to use MatConvNet it is now necessary to allow
MATLAB accessing these libraries. On Linux one way to do so
is to start MATLAB from the command line (terminal) specifying the
`LD_LIBRARY_PATH` option. For instance, on Linux this may look like:

    $ LD_LIBRARY_PATH=/Developer/NVIDIA/CUDA-8.0/lib64 matlab

On Windows, chances are that the CUDA libraries are already visible to
MATLAB so that nothing else needs to be done.

#### Windows with CUDA 12.8

CUDA 12.8 builds must use the `nvcc` method and a 64-bit Visual Studio
2019 or Visual Studio 2022 C++ compiler. Configure the compiler first
with `mex -setup C++`, then compile with:

    > vl_compilenn('enableGpu', true, ...
                   'cudaRoot', 'C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.8', ...
                   'cudaMethod', 'nvcc', ...
                   'verbose', 1)

If `CUDA_PATH` points to CUDA 12.8, the `cudaRoot` option may be omitted.
The build script queries `nvcc` for its supported GPU architectures, so
it does not pass the `sm_20` or `sm_30` targets removed from recent CUDA
toolkits. To target a specific GPU explicitly, pass a `cudaArch` value
such as:

    -gencode=arch=compute_89,code=sm_89 -gencode=arch=compute_89,code=compute_89

On macOS, this step should not be necessary as the library paths are
hardcoded during compilation.

<a name='cudnn'></a>
### Compiling the cuDNN support

MatConvNet supports the NVIDIA <a
href='https://developer.nvidia.com/cuDNN'>cuDNN library</a> for deep
learning (and in particular their fast convolution code). The current
code supports the cuDNN 9.8 legacy API as well as older cuDNN releases.
Make sure that the CUDA toolkit matches the cuDNN package. When using
CUDA 12.8 and cuDNN 9.8, use the [`nvcc` compilation method](#nvcc).

On Windows, unpack cuDNN so that the selected `<Cudnn>` root contains:

     <Cudnn>/
       bin/
         cudnn*.dll
       include/
         cudnn.h
         cudnn_version.h
       lib/
         x64/
           cudnn.lib

Compile CUDA 12.8 and cuDNN 9.8 support with:

    > vl_compilenn('enableGpu', true, ...
                   'cudaRoot', 'C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.8', ...
                   'cudaMethod', 'nvcc', ...
                   'enableCudnn', true, ...
                   'cudnnRoot', 'C:\local\cudnn-9.8', ...
                   'verbose', 1)

All cuDNN DLLs in `<Cudnn>/bin` are copied to `matlab/mex`
automatically.

Unpack the cuDNN library binaries and header files in a place
`<Cudnn>` of you choice. In the rest of this example, it will be
assumed that this cuDNN RC4 has been unpacked in `local/cudnn-rc4` in
the `<MatConvNet>` root directory,
(i.e. `<Cudnn>`=`<MatConvNet>/local/cudnn-rc4`). For example, the
directory structure on macOS should look like:

     COPYING
     Makefile
     Makefile.mex
     ...
     examples/
     local/
       cudnn-rc4/
         include/
           cudnn.h
         lib/
           libcudnn.7.5.dylib
           libcudnn.dylib
         ...

Use `vl_compilenn` with the `enableCudnn,true` option to compile the
library; do not forget to use `cudaMethod,nvcc` as, at it is likely,
the CUDA toolkit version is newer than MATLAB's CUDA toolkit. For
example, on macOS this may look like:

    > vl_compilenn('enableGpu', true, ...
                   'cudaRoot', '/Developer/NVIDIA/CUDA-7.5', ...
                   'cudaMethod', 'nvcc', ...
                   'enableCudnn', true, ...
                   'cudnnRoot', 'local/cudnn-rc4') ;

MatConvNet is now compiled with cuDNN support. When starting MATLAB,
however, do not forget to point it to the paths of both the CUDA and
cuDNN libraries. On a Linux terminal, this may look like:

    $ cd <MatConvNet>
    $ LD_LIBRARY_PATH=/Developer/NVIDIA/CUDA-7.5/lib64:local matlab

On Windows, the build script copies the cuDNN DLL files into the
`<MatConvNet>/matlab/mex` directory.

On macOS, this step should not be necessary as the library paths are
hardcoded during compilation.

## Further examples

To compile all the features in MatConvNet on macOS and MATLAB 2014b,
CUDA toolkit 6.5 and cuDNN Release Candidate 2, use:

    > vl_compilenn('enableGpu', true, 'cudaMethod', 'nvcc', ...
                   'cudaRoot', '/Developer/NVIDIA/CUDA-6.5', ...
                   'enableCudnn', true, 'cudnnRoot', 'local/cudnn-rc2') ;

The equivalent command on Ubuntu Linux would look like:

    > vl_compilenn('enableGpu', true, 'cudaMethod', 'nvcc', ...
                   'cudaRoot', '/opt/local/cuda-6.5', ...
                   'enableCudnn', true, 'cudnnRoot', 'local/cudnn-rc2') ;

Using MATLAB 2015b, CUDA 7.5, and cuDNN R4:

    > vl_compilenn('enableGpu', true, ...
                   'cudaMethod', 'nvcc', ...
                   'cudaRoot', '/opt/local/cuda-7.5', ...
                   'enableCudnn', true, ...
                   'cudnnRoot', 'local/cudnn-rc4') ;
