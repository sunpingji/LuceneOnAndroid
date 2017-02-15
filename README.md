# Lucene On Android 填坑

之前本科毕设的时候做过Lucene到Android上使用的毕设。那时候做的比较简单，所以最近想要看看最新的lucene在Android上使用的效果。于是就填了不少坑。
##1.Lucene版本的选择
一开始肯定选择最新的版本，也就是6.4.1版本。然后lucene 6以上都是使用java 8 编译的。编译的时候就会报需要选择java8的错误

![image_1b908e8tt9cc142o8141jpd18kr9.png-14.2kB][1]
build中改成Java8 又要求使用jack。
![image_1b908hfai44i3k517tl8e4hh7m.png-11.2kB][2]
使用jack中又发现lucene使用的lambda有问题
![image_1b908kru0a7117n2om9150kcu413.png-28.9kB][3]
于是放弃治疗。放弃lucene 6系列。
那么再尝试下lucene 5系列。lucene 5是java 7 编译的，所以build脚本不需要改动，当我以为以为没有万事ok的时候，发现运行时出现了问题。
Lucene中StringHelper这个类 有东西是通过java 7的 java.nio.file这个库实现的。而Android里并没有完整的集成java 7 所以导致会报错。![image_1b909b545qm3kaq1bgb1o8qirr1g.png-9.8kB][4]
http://stackoverflow.com/questions/24869323/android-import-java-nio-file-files-cannot-be-resolved
那么怎么办呢。就引入源代码而不是library,来修改源代码。结果又出问题。第一个问题就是运行错误。会报类似
![image_1b909jfkh1k8jlfn1ugkilboll1t.png-21.6kB][5]
这个问题。这个问题可能有两方面，第一就是可能没有引入源码的Meta_inf文件，第二就算引入了也会报错，因为Android打包的时候不会把工程下的Meta_inf给打进去，所以加了也白加。
那么怎么办呢，两个方法，第一个就是改SipHelper中的Loader，改变加载位置，第二个就是打包结束后改Meta_inf中的内容。我采用的是第一种，参考http://stackoverflow.com/questions/21951049/lucene-4-on-android-error/22886880#22886880。改完以后应该没问题了吧？还是有，Lucene 5系列除了用了files 这个Android上的 java 7 没有的类以外 还有 classValue 和 methodHandler 等等。我改了几个以后实在不想改了。放弃治疗。于是放弃java 5了。
那么 java 4呢 我看了下grepCode中lucece的源码发现lucene 4 都是用codecs处理过的，也就是都会用Meta_inf的问题要处理。于是算了 就用lucene 3吧。果然没啥问题 。build的时候

      compile group: 'org.apache.lucene', name: 'lucene-core', version: '3.6.2'
      compile group: 'org.apache.lucene', name: 'lucene-queries', version: '3.6.2'
       compile group: 'org.apache.lucene', name: 'lucene-queryparser', version: '3.6.2'

把这几个都加入就行了，好吧，填坑结束。最终在lucene 6已经存在的情况下，用Lucene 3系列。
  [1]: http://static.zybuluo.com/sunpj/1uw4c0wahx4kesj44nia8qk7/image_1b908e8tt9cc142o8141jpd18kr9.png
  [2]: http://static.zybuluo.com/sunpj/oovhylvco95h33pf3cghos52/image_1b908hfai44i3k517tl8e4hh7m.png
  [3]: http://static.zybuluo.com/sunpj/nm67dhvng7s3hu9ghl6waba3/image_1b908kru0a7117n2om9150kcu413.png
  [4]: http://static.zybuluo.com/sunpj/baazhru5cdnk5kjs3x296hr9/image_1b909b545qm3kaq1bgb1o8qirr1g.png
  [5]: http://static.zybuluo.com/sunpj/dyn6v9wd9y6opw6c4pbarn6h/image_1b909jfkh1k8jlfn1ugkilboll1t.png