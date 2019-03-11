# 界面组件和注意事项
> 原文链接：[Paging library UI components and considerations  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/paging/ui)

本节教程是基于 [Paging 库概览](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_7_Paging_library/3_2_7_1_Overview.md)的，介绍了您应如何在应用 UI 中把列表的信息展示给用户，尤其是当这些信息变动时。

## 把 UI 连接到 ViewModel

您可以把一个 [`LiveData<PagedList>`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData?hl=zh-cn) 实例连接到 [`PagedListAdapter`](https://developer.android.google.cn/reference/android/arch/paging/PagedListAdapter?hl=zh-cn)，如下所示：

```java
public class ConcertActivity extends AppCompatActivity {
    private ConcertAdapter mAdapter;
    private ConcertViewModel mViewModel;

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mViewModel = ViewModelProviders.of(this).get(ConcertViewModel.class);
        mViewModel.concertList.observe(this, mAdapter::submitList);
    }
}
```

当数据源提供新的 `PagedList` 实例时，activity 会把这些对象发送给适配器。[`PagedListAdapter`](https://developer.android.google.cn/reference/android/arch/paging/PagedListAdapter?hl=zh-cn) 的实现定义了这些更新将如何被计算，而列表的分页和版本比较（diffing）是自动完成的。因此，您的 [`ViewHolder`](https://developer.android.google.cn/reference/android/support/v7/widget/RecyclerView.ViewHolder?hl=zh-cn) 只需绑定到一个已提供的特定项目：

```java
public class ConcertAdapter
        extends PagedListAdapter<Concert, ConcertViewHolder> {
    protected ConcertAdapter() {
        super(DIFF_CALLBACK);
    }

    @Override
    public void onBindViewHolder(@NonNull ConcertViewHolder holder,
            int position) {
        Concert concert = getItem(position);

        // 注意，当 “concert“ 是占位符时它可能是 null。
        holder.bindTo(concert);
    }

    private static DiffUtil.ItemCallback<Concert> DIFF_CALLBACK =
            new DiffUtil.ItemCallback<Concert>() {
        // 当 ID 属性相同时，我们就认为两个项目是相同的。
        @Override
        public boolean areItemsTheSame(Concert oldItem, Concert newItem) {
            return oldItem.getId() == newItem.getId();
        }

        // 使用 Object.equals() 来得知项目的内容是否产生了变化。
        // 您可以实现 equals() 方法，或者编写您自己定义的数据比较逻辑。
        @Override
        public boolean areContentsTheSame(Concert oldItem, Concert newItem) {
            return oldItem.equals(newItem);
        }
    };
}
```

`PagedListAdapter` 使用一个 [`PagedList.Callback`](https://developer.android.google.cn/reference/android/arch/paging/PagedList.Callback?hl=zh-cn) 对象来处理页面重载。当用户滚动列表时，`PagedListAdapter` 调用 [`PagedList.loadAround()`](https://developer.android.google.cn/reference/android/arch/paging/PagedList?hl=zh-cn#loadaround) 来提示其底层的 `PagedList` 应当从 [`DataSource`](https://developer.android.google.cn/reference/android/arch/paging/DataSource?hl=zh-cn) 获取哪些数据。

> **注意**：`PagedList` 的内容是不可更改的。这代表着，即使新的内容可以被读入一个 PagedList 实例，项目只要已经完成了加载就不能变更。这样一来，当一个 `PagedList` 的内容更新时，`PagedListAdapter` 对象就会收到一个全新的包含了新内容的 `PagedList`。

## 实现版本比较的回调

签名的例子展示了 [`areContentsTheSame()`](https://developer.android.google.cn/reference/android/support/v7/util/DiffUtil.ItemCallback?hl=zh-cn#arecontentsthesame) 的一个手动实现，它使用对象的相关字段来进行比较。你也可以在 Java 代码中使用 `Object.equals()` 方法、或者 Kotlin 代码中使用 `==` 运算符来比较两个内容，但请确保实现 `equals()` 方法或者使用 Kotlin 的类。

## 使用一个不同的适配器类型来进行版本比较

如果您选择不继承自 `PagedListAdapter`（例如，当您使用了一个已经提供了适配器的类库）您仍能通过 [`AsyncPagedListDiffer`](https://developer.android.google.cn/reference/android/arch/paging/AsyncPagedListDiffer?hl=zh-cn) 对象来使用分页库适配器的版本比较功能

## 为您的 UI 提供占位符

如果您想让 UI 在应用加载完数据前就展示列表的情况，那么您可以给用户展示占位符项目。`RecyclerView` 能通过把项目临时设置成 `null` 来处理这种情况。

> **注意**：默认情况下，分页库是启用该占位符功能的。

占位符有以下的好处：
- **支持滚动条**：`PagedList` 把列表中项目的数量提供给 `PagedListAdapter`，这样一来适配器就能绘制一个滚动条，用于传达整个列表的长度的信息。当新的页面加载时，由于您的列表大小并未改变，滚动条就不会跳跃。
- **无须加载进度条**：由于列表的大小是已知的，您无须再警告用户正在加载更多的项目，因为占位符本身就能传达这个信息。

然而，在添加占位符的支持之间，请您谨记这些先决条件：
- **要求有一个数量可数的数据集合**：[Room 数据持久化库](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_8_Room_Persistence_Library.md) 中的 `DataSource` 实例能高效地数清楚其项目的数量。然而，如果您使用的是一个自定义的本地存储解决方案、或是一个[只从网络加载数据的架构](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_7_Paging_library/3_2_7_1_Overview.md)，那么想要确定数据库中一共有多少个项目就可能会成本很高甚至不可能做到。
- **要求适配器能处理未加载的数据项目**：您使用的用来把列表准备给 UI 填充的适配器或展示机制需要处理数据项目是 `null` 的情况。例如，当您绑定一个数据项目到 `ViewHolder` 时，您需要提供表示未加载数据的缺省值。
- **要求数据项目的视图有着相同的尺寸**：如果列表项目的大小会根据其内容而变动，例如社交网络的更新，那么这些项目彼此淡入淡出的效果就有点尴尬了。我们强烈建议在这种情况下禁用占位符功能。

## 提交反馈

通过以下途径分享您的反馈和想法：

[Issue tracker](https://issuetracker.google.com/issues/new?component=413106&template=1096385)
![bug](https://developer.android.google.cn/topic/libraries/architecture/images/bug.png)

报告问题以便我们修复。

## 其他资源

欲了解更多有关分页库的信息，请参阅以下资源：

### 示例
- [Android Architecture Components Paging sample](https://github.com/googlesamples/android-architecture-components/tree/master/PagingSample) （分页库示例代码）
- [Paging With Network Sample](https://github.com/googlesamples/android-architecture-components/tree/master/PagingWithNetworkSample) （带网络数据的分页库示例）

### Codelabs
- [Android Paging codelab](https://codelabs.developers.google.com/codelabs/android-paging/index.html?index=..%2F..%2Findex#0) （分页库代码实验室）

### 视频
- [Android Jetpack: manage infinite lists with RecyclerView and Paging (Google I/O '18)](https://www.youtube.com/watch?v=BE5bsyGGLf4) （用分页库管理无穷列表控件）
- [Android Jetpack: Paging](https://www.youtube.com/watch?v=QVMqCRs0BNA) （分页库视频简介）
