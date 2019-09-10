unable to find utility "llvm-dsymutil"
编译出现“xcrun: error: unable to find utility "llvm-dsymutil", not a developer tool or in PATH”，

进入theos安装目录，一般是/opt/theos，$THEOS/makefiles/targets/目录下，以前是common，后来改为了_common，打开目录下的darwin_head.mk文件，将两处llvm-dsymutil修改为dsymutil。
--------------------- 
作者：some_do 
来源：CSDN 
原文：https://blog.csdn.net/some_do/article/details/90474974 
版权声明：本文为博主原创文章，转载请附上博文链接！


Can’t locate IO/Compress/Lzma.pm in @INC 
执行下面命令装一下就行了
sudo cpan -I IO::Compress::Lzma
http://iosre.com/t/tweak-make-package/10382/7