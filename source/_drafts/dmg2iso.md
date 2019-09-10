在macOS上可以在Terminal里完成dmg到iso的转换。

1、dmg-cdr
convert /path/imagefile.dmg -format UDTO -o /path/convertedimage.cdr
2、cdr-iso
makehybrid /path/convertedimage.cdr -iso -joliet -o /path/convertedimage.iso

反向操作 iso到dmg：
hdiutil convert /path/imagefile.iso -format UDRW -o /path/convertedimage.dmg

ref:https://www.jianshu.com/p/96ef03754c59