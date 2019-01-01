# Compile Tensorflow on Mac Mojave (non-CUDA)

The easiest and most straight-forward way of using TensorFlow Serving is with Docker images. However, I compiled tf on my Mac client in order to use it in an anaconda dev environment.

Default Tensorflow of anconda for Mac was compiled with mkl Support only, and produces the follwing warnings:
tensorflow/core/platform/cpu_feature_guard.cc:141] Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA.

Instructions to compile Tensorflow are described below.

## Build Environment

### clang

clang --version
Apple LLVM version 10.0.0 (clang-1000.10.44.4)
Target: x86_64-apple-darwin18.2.0
Thread model: posix
InstalledDir: /Library/Developer/CommandLineTools/usr/bin

Building with just -c opt --copt=-march=native should be at least as good as
--copt=-mavx --copt=-mavx2 --copt=-mfma --copt=-msse4.1 --copt=-msse4.2.
But -march=native might make even faster code by tuning specifically for the CPU
on the system you're building on.

Using [clang](http://llvm.org/): export CC=/usr/bin/clang

### Python

My local environment is miniconda env with Python 3.6.6.

### Bazel

* [Bazel versions](https://github.com/bazelbuild/bazel/releases) - my preference
* [Bazel homebrew](https://blog.bazel.build/2018/08/22/bazel-homebrew.html)

In order to build objc, Bazel requires that you specify an xcode version - this is usually done automatically by xcode_configure. If that's not working, you can manually specify the xcode version on the command line using the --xcode_version flag or alternatively start Xcode - go to preferences - tab local and take care that you desired 
command line tool is selected (in my case Xcode 10.1)

Note, If already ran in a problem, you'll need to run:
`bazel clean --expunge`

with '--expunge' the entire working tree will be removed and the server stopped.

### TF dependencies

Install the TensorFlow pip package dependencies (in my virtual environment, I omit the --user argument):

```bash
pip install -U pip six numpy wheel mock
pip install -U keras_applications==1.0.6 --no-deps
pip install -U keras_preprocessing==1.0.5 --no-deps
```

## Build an install

git clone https://github.com/tensorflow/tensorflow.git

cd tensorflow

./configure

bazel build --config=opt //tensorflow/tools/pip_package:build_pip_package

./bazel-bin/tensorflow/tools/pip_package/build_pip_package &lt;target folder for wheel package&gt;

Replace existing conda package:

pip install --upgrade &lt;target folder for wheel package&gt;/tensorflow-version-tags.whl

You may download the ready to run whl package from this repo.

build takes a while:

´´´
INFO: Elapsed time: 7041,361s, Critical Path: 318,16s
INFO: 9289 processes: 9289 local.
INFO: Build completed successfully, 9904 total actions
´´´

## Train Performance

I made a simple performance comparison with a common and reproducible lerarning task.
Use `utorials/image/cifar10´`form [tensorflow/models](https://github.com/tensorflow/models.git).

Standard conda tf package performance, by `python cifar10_train.py`:

* ~220 examples/sec; 0.55-0.6 sec/batch)

CPU optimized, compiled on Mac:

* ~780 examples/sec; ~0.15 sec/batch)

Amazing: at minimum 3.5 times faster

Relation to GPU performance:

* ~ 1190 examples/sec, ~0,1 sec/batch for an old-timer (940MX)
* ~ 6000~7000 examples/sec for GTX1070, which is around 9 times increase over recent 6 core i7 of MacBook pro

Conclusion: For real deep learning scenarios you need to add GPU performance somehow from cloud or a workstaion with sufficient GPU power. However, for simple local scenarios it's worth to compile your own CPU optimized version