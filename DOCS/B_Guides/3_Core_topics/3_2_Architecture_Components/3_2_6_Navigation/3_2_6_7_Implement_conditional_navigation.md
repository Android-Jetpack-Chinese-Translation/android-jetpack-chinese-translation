# 实现有条件的导航
> 原文链接：[Implement conditional navigation  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/navigation/navigation-conditional)

您的应用可能有一系列的只能在特定条件下（例如，当用户需要登录时）使用的目的地，即：**有条件的目的地（conditional destination）**。这些目的地应当被创建为由其他目的地发起的独立目的地或嵌套导航图。

图一展示了这样的情形：某个用户导航到个人主页目的地，而后者判定用户并没有登录，因此要求用户导航到登录目的地；接着，等用户完成登陆后，该登录目的地再把用户带回个人主页的目的地。

![图一：有条件的导航](https://developer.android.google.cn/images/topic/libraries/architecture/navigation-conditional-graph.png?hl=zh-cn)
**图一：**有条件的导航

该登录目的地应在用户回到个人主页目的地后，将自己从导航栈中出栈。请在导航回到原先的目的地时调用 [`popBackStack()`](https://developer.android.google.cn/reference/androidx/navigation/NavController.html?hl=zh-cn#popBackStack())，那么原先的目的地就会被顶到栈顶，因而处于活跃状态。

