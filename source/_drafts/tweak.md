unable to find utility "llvm-dsymutil"
编译出现“xcrun: error: unable to find utility "llvm-dsymutil", not a developer tool or in PATH”，

进入theos安装目录，一般是/opt/theos，$THEOS/makefiles/targets/目录下，以前是common，后来改为了_common，打开目录下的darwin_head.mk文件，将两处llvm-dsymutil修改为dsymutil。
