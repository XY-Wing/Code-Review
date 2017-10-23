Code-Review
=====
   薛扬(Wing)       
---
### 1.What's code review?
- 代码评审是指在软件开发过程中，对源代码的系统性检查。通常的目的是查找系统缺陷，保证软件总体质量和提高开发者自身水平。 Code Review是轻量级代码评审，相对于正式代码评审，轻量级代码评审所需要的各种成本要明显低的多，如果流程正确，它可以起到更加积极的效果。正因如此，轻量级代码评审经常性得被引入到软件开发过程中。
### 2.Why we do Code Review?
- 提高质量</br>
- 及早发现潜在缺陷与BUG，降低事故成本。</br>
- 促进团队内部知识共享，提高团队整体水平。</br>
- 评审过程对于评审人员来说，也是一种思路重构的过程。帮助更多的人理解系统。</br>
### 3.My code review.
* 今天我要review的一个实现优化及功能提升的界面及其代码。
  * 案例：假勤审批（未优化）</br>
  ![image](https://github.com/XY-Wing/Code-Review/blob/master/GIF/holiday.gif)
  * 该界面存在的问题：点击进入（`假勤审批`），系统会直接一次行创建五个界面（`请假，出差，加班，外出，补签`）；并且，每个界面都有自己的网络请求，这些请求都是瞬间同时发出，存在一定的风险。（### `声明：产品设计如此` ###）。
  * 可优化方案：点击进入（`假勤审批`），首先只创建一个界面（请假），用户可自行点击其想要访问的剩余四个界面，点击后创建界面并发出请求，不点击，无操作。这样既节省系统开销，又节省用户流量，提升界面流畅度，提升性能。
# 应用：
![image](https://github.com/XY-Wing/Code-Review/blob/master/GIF/QueryDetailNone.gif)
![image](https://github.com/XY-Wing/Code-Review/blob/master/GIF/QueryDetail.gif)
* 大家可以仔细的看一下界面从`已签到`滑动到`未签到`时的异同。
### 实现方式：
* 进入时：
```ObjC
- (void)setupChildControllers
{
   [_controllers enumerateObjectsUsingBlock:^(UIViewController *  _Nonnull vc, NSUInteger idx, BOOL * _Nonnull stop) {
        if (idx == 0) [self invokeChildVCMethodWithIndex:idx];
    }];
   
}
```
* 滑动停止时
```ObjC
- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView
{
    CGFloat idx = (NSInteger)scrollView.contentOffset.x;
    [self invokeChildVCMethodWithIndex:idx];
}
```
* 点击文字按钮时
```ObjC
- (void)itemDidClicked:(UITapGestureRecognizer *)gesture
   //计算idx
   UILabel *item = (UILabel *)gesture.view;
   if (item == _items[_itemSelectedIndex]) return;
   idx = (int)[_items indexOfObject:item];
   [self invokeChildVCMethodWithIndex:idx];
}
```
* 主方法
```ObjC
- (void)invokeChildVCMethodWithIndex:(NSInteger)index
{
    UIViewController *vc = _controllers[index];
    objc_msgSend(vc, NSSelectorFromString(kXYChildVCNecessaryMothed));
}
```
