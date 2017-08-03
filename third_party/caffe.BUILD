package(default_visibility = ["//visibility:public"])

# gets the filepath to protobuf
genquery(
    name = "protobuf-root",
    expression = "@com_github_google_protobuf//:protobuf",
    scope = ["@com_github_google_protobuf//:protobuf"],
    opts = ["--output=location"]
)

genrule(
    name = "configure",
    message = "Building Caffe (this may take a while)",
    srcs = [
        ":protobuf-root",
        ":CMakeLists.txt",
        "@com_github_google_protobuf//:protoc",
        "@com_github_google_protobuf//:protobuf",
    ],
    outs = [
        "lib/libcaffe.so",
        "lib/libproto.a",
        "include/caffe/proto/caffe.pb.h",
    ],
    cmd =
        '''
        srcdir=$$(pwd);
        workdir=$$(mktemp -d -t tmp.XXXXXXXXXX);
        outdir=$$srcdir/$(@D);

        protobuf_incl=$$(grep -oP "^/\\\S*(?=/)" $(location :protobuf-root))/src;
        protoc=$$srcdir/$(location @com_github_google_protobuf//:protoc);
        protolib=$$srcdir/$$(echo "$(locations @com_github_google_protobuf//:protobuf)" | grep -o "\\\S*/libprotobuf.so"); ''' +

        # configure cmake.
        '''
        pushd $$workdir;
        cmake $$srcdir/$$(dirname $(location :CMakeLists.txt)) \
            -DCMAKE_INSTALL_PREFIX=$$srcdir/$(@D) \
            -DCMAKE_BUILD_TYPE=Release            \
            -DCPU_ONLY=ON                         \
            -DBUILD_python=OFF             \
            -DBUILD_python_layer=OFF       \
            -DUSE_OPENCV=ON                      \
            -DBUILD_SHARED_LIBS=ON               \
            -DUSE_LEVELDB=ON                     \
            -DUSE_LMDB=ON                        \
            -DPROTOBUF_INCLUDE_DIR=$$protobuf_incl\
            -DPROTOBUF_PROTOC_EXECUTABLE=$$protoc \
            -DPROTOBUF_LIBRARY=$$protolib; ''' +

        '''
        cmake --build . --target caffe -- -j 8
        cp -r ./lib $$outdir
        cp -r ./include $$outdir; ''' +

        '''
        # clean up
        popd;
        # rm -rf $$workdir;
        '''
)

cc_library(
    name = "lib",
    includes = ["include/"],
    srcs = [
        "lib/libcaffe.so",
        "lib/libproto.a"
    ],
    hdrs = glob(["include/**"])
           + ["include/caffe/proto/caffe.pb.h"],
    deps = [
        "@com_github_google_protobuf//:protobuf",
    ],
    defines = ["CPU_ONLY"],
    linkopts = [
        "-L/usr/lib/x86_64-linux-gnu/hdf5/serial/lib",
        "-Wl,-rpath,/usr/local/lib:/usr/lib/x86_64-linux-gnu/hdf5/serial/lib",
        "-lboost_system",
        "-lboost_thread",
        "-lboost_filesystem",
        "-lpthread",
        "-lblas",
        "-lcblas",
        "-lhdf5_hl",
        "-lhdf5",
        "-lz",
        "-ldl",
        "-lm",
    ],
    visibility = ["//visibility:public"],
    linkstatic = 0,
)