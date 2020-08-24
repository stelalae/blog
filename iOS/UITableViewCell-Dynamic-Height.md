# UITableViewCell动态高度及高度缓存

优秀的第三方库：[UITableViewDynamicLayoutCacheHeight](https://github.com/liangdahong/UITableViewDynamicLayoutCacheHeight)，下面大致说下自己实现思路：

UITableView的cell自适应高度其实很简单，但最重要的是一定要给cell自上而下加好约束，如果写出来的的cell没有自适应高度，那基本上都是cell的约束没有写好的原因。**所以，自上而下，上下左右的约束都要写好，才能达到自适应高度的效果。**

设置tableView自适应属性和预估高度。理论上这个是可以随便写的，并不影响显示结果，但是越接近真实高度越好。

```objective-c
- (CGFloat)tableView:(UITableView *)tableView estimatedHeightForRowAtIndexPath:(NSIndexPath *)indexPath {
    return 80;
}

//  或者（推荐下面的）

self.tableView.rowHeight = UITableViewAutomaticDimension;
self.tableView.estimatedRowHeight = 80;
```

- UITableView在进行渲染的时候会先走estimatedHeightForRowAtIndexPath这个方法，会先给它一个预估的高度值，然后在根据Cell的具体内容去计算真实高度值；
- 在Cell里面写UI的时候，self或者self.contentView必须要统一，不能有的加在self上有的加在self.contentView上，否则影响了Cell的 自适应高度。各个控件上下全都约束好，除了内容不固定的地方，其他地方的高度尽量要给一个准确的值！
- 一般Cell需要自适应高度，是因为有一个需要用UILabel显示的地方的内容和高度不固定，所以才需要自适应高度，针对这个`[self.contentLabel setNumberOfLines:0]` ，即numberoflines这个属性一定要设置成0， 约束上不用设置高度，但是上下约束一定要约束好。
- 在写Cell上的控件的约束的时候，一定要注意，自上而下，第一个控件距离顶部的距离，最后一个控件距离底部的距离都要写上，尤其不能忘了最后一个控件距离底部的约束一定要写上。



现在你已经实现了高度自适应，但是系统每次在渲染时还是去计算自适应。

为了提高性能，可将第一次显示后拿到的高度缓存下来，然后在需要高度时直接返回即可。下面是改版方案：

```objective-c
- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath {
  NSNumber *height = @(cell.frame.size.height); // 拿到高度
  self.arrMsg[indexPath.row].height = height; // 缓存高度
}

-(CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
  if(self.arrMsg[indexPath.row].height) {
    return self.arrMsg[indexPath.row].height; // 返回缓存的高度
  }
  return UITableViewAutomaticDimension; // 告诉系统使用动态高度
}
```



另外，UILabel在TableCell中的高度自适应请看[systemLayoutSizeFittingSize 高度自适应](https://www.jianshu.com/p/fa0288a18ede)。



---

参考资料：

- [UITableView自动计算cell高度](https://blog.csdn.net/FISHBALL1/article/details/78971469)
- [systemLayoutSizeFittingSize 高度自适应](https://www.jianshu.com/p/fa0288a18ede)

