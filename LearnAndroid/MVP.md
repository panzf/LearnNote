## MVP模式##
### 为什么要有MVP ###

 只使用简单的View-Modle结构使得结构不清晰到项目中后期你会发现所有事物都联系在一起了(View的逻辑与Modle的逻辑)而且中后期更多的是与View打交道修改起来困难使用MVP就能够很好的细分解耦和更容易测试更少BUG

### 优点与缺点 

####### 优点： ####


- 降低耦合度，实现了Model和View真正的完全分离，可以修改View而不影响Modle
- 模块职责划分明显，层次清晰，隐藏数据
- View可以进行组件化，高度复用View
- 代码灵活性更高

###### 缺点： ######
- Persenter中除去应用逻辑外还有大量的View-Modle Modle-View的手动同步逻辑不易于维护
- 由于对视图控制渲染都放在Persenter中，所以View与Persenter交互频繁与视图过于紧密联系一旦View变动Persenter也要跟着变动
- 额外的学习成本高
### MVP模式的4要素 ###
- View :负责绘制UI元素、与用户进行交互(在Android中体现为Activity）
- View interface :需要View实现的接口，View通过View interface与Presenter进行交互，降低耦合，方便进行单元测试
- Model :负责存储、检索、操纵数据(有时也实现一个Model interface用来降低耦合）
- Presenter :作为View与Model交互的中间纽带，处理与用户交互的负责逻辑