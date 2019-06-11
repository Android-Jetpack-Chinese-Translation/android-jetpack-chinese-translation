# 数据组件和注意事项
> 原文链接：[Paging library data components and considerations  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/paging/data)

本节教程是基于 [Paging 库概览](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_6_Paging_library/3_2_6_1_Overview.md)的，介绍了您应如何根据您应用的架构来自定义数据加载的解决方案。

## 构建一个可观测的列表

通常情况下，您的 UI 代码观察 `ViewModel` 中的一个 `LiveData<PagedList>` 对象（或者，如果您用的是 RxJava2 的话，`Flowable<PagedList>` 或 `Observable<PagedList>` 对象）。该对象把您应用的列表数据内容连接到展示层。

欲创建这样的一个 `PagedList` 对象，把 [`DataSource.Factory`](https://developer.android.google.cn/reference/android/arch/paging/DataSource.Factory?hl=zh-cn) 的实例传入 [`LivePagedListBuilder`](https://developer.android.google.cn/reference/android/arch/paging/LivePagedListBuilder?hl=zh-cn) 或 [`RxPagedListBuilder`](https://developer.android.google.cn/reference/android/arch/paging/RxPagedListBuilder?hl=zh-cn) 对象。一个 `DataSource` 对象负责把页面加载到单个 `PagedList` 中，而其工厂类根据内容的更新来创建 `PagedList` 的新实例。[Room 数据持久化库](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_7_Room_Persistence_Library.md)提供了 `DataSource.Factory`，但你也可以自己来定制。

如下的代码展示如何使用 Room 的 [`DataSource.Factory`](https://developer.android.google.cn/reference/android/arch/paging/DataSource.Factory?hl=zh-cn) 的构造能力，来在您应用的 `ViewModel` 中创建 `LiveData<PagedList>` 实例：

```java
ConcertDao:

@Dao
public interface ConcertDao {
    // 整型参数告诉 Room 应当使用 PositionalDataSource 对象，
    // 并由 Room 加载基于位置的数据。
    @Query("SELECT * FROM concerts ORDER BY date DESC")
    DataSource.Factory<Integer, Concert> concertsByDate();
}

ConcertViewModel:

// 整型参数所对应的是一个 PositionalDataSource 对象。
DataSource.Factory<Integer, Concert> myConcertDataSource =
       concertDao.concertsByDate();

LiveData<PagedList<Concert>> concertList =
        LivePagedListBuilder(myConcertDataSource, /* 分页大小 */ 20).build()
```

## 定义您自己的分页配置

欲配置 `LiveData<PagedList>` 来处理更复杂的情形，您还能定义自己的分页配置。特别地，您可以定义如下属性：
- **[分页大小](https://developer.android.google.cn/reference/android/arch/paging/PagedList.Config.Builder#setPageSize%28int%29)**：每一页数据项目的数量。
- **[预加载的阈值](https://developer.android.google.cn/reference/android/arch/paging/PagedList.Config.Builder#setPrefetchDistance%28int%29)**：分页库在距离应用 UI 中的最后一个可见的数据项该阈值个项目时开始预加载。该阈值应当是分页大小的若干倍。
- **[占位符是否存在](https://developer.android.google.cn/reference/android/arch/paging/PagedList.Config.Builder#setEnablePlaceholders%28boolean%29)**：决定 UI 是否应为还未加载完毕的列表项目展示占位符。欲了解使用占位符的优缺点讨论，请参阅 [为您的 UI 提供占位符](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_6_Paging_library/3_2_6_2_UI_Components_and_Considerations.md)。

如果您想在分页库从数据库中加载一个列表时获得更多掌控，请将一个自定义的 [`Executor`](https://developer.android.google.cn/reference/java/util/concurrent/Executor?hl=zh-cn) 对象传入 [`LivePagedListBuilder`](https://developer.android.google.cn/reference/android/arch/paging/LivePagedListBuilder?hl=zh-cn)，如下所示：

```java
ConcertViewModel:

PagedList.Config myPagingConfig = new PagedList.Config.Builder()
        .setPageSize(50)
        .setPrefetchDistance(150)
        .setEnablePlaceholders(true)
        .build();

// 整型参数所对应的是一个 PositionalDataSource 对象。
DataSource.Factory<Integer, Concert> myConcertDataSource =
        concertDao.concertsByDate();

LiveData<PagedList<Concert>> concertList =
        new LivePagedListBuilder<>(myConcertDataSource, myPagingConfig)
            .setFetchExecutor(myExecutor)
            .build();
```

## 选择合适的数据来源类型

选择最合适处理您源数据结构的数据来源：
- 使用 [`PageKeyedDataSource`](https://developer.android.google.cn/reference/android/arch/paging/PageKeyedDataSource?hl=zh-cn) 如果您加载的页面嵌入了上一个/下一个键。例如，如果您从网络加载社交网络的状态，您可能需要传入一个 `nextPage` 的口令来加载下一波数据。
- 使用 [`ItemKeyedDataSource`](https://developer.android.google.cn/reference/android/arch/paging/ItemKeyedDataSource?hl=zh-cn) 如果您需要使用第 N 个数据来加载第 N+1 个数据。例如，如果您在讨论组应用中加载帖子中的评论，您可能需要把最后一个评论的 ID 作为参数来加载下一个评论。
- 使用 [`PositionalDataSource`](https://developer.android.google.cn/reference/android/arch/paging/PositionalDataSource?hl=zh-cn) 如果您需要从数据源的任意位置加载分页的数据。该类支持从您选择的任意位置开始加载数据，例如，您可以从第 1200 个数据开始加载一批 20 个数据项目。

## 在数据无效时发起通知

使用分页库的时候，**数据层**负责通过调用 `DataSource` 类中的 [`invalidate`](https://developer.android.google.cn/reference/android/arch/paging/DataSource?hl=zh-cn#invalidate) 方法，在一个数据表（table）或行（row）过期时通知您应用的其他层。

> **注意**：你引用的 UI 可以通过[下拉刷新](https://developer.android.google.cn/training/swipe/?hl=zh-cn)来触发这种数据作废功能。

## 构建您自己的数据源

如果您使用了一个自定义的本地数据解决方案，或者直接从网络加载数据，您可以实现 [`DataSource`](https://developer.android.google.cn/reference/android/arch/paging/DataSource?hl=zh-cn) 的一个子类。如下的代码展示了一个以演唱会开始时间作为主要输入的数据源：

```java
public class ConcertTimeDataSource
        extends ItemKeyedDataSource<Date, Concert> {
    @NonNull
    @Override
    public Date getKey(@NonNull Concert item) {
        return item.getStartTime();
    }

    @Override
    public void loadInitial(@NonNull LoadInitialParams<Date> params,
            @NonNull LoadInitialCallback<Concert> callback) {
        List<Concert> items =
            fetchItems(params.key, params.requestedLoadSize);
        callback.onResult(items);
    }

    @Override
    public void loadAfter(@NonNull LoadParams<Date> params,
            @NonNull LoadCallback<Concert> callback) {
        List<Concert> items =
            fetchItemsAfter(params.key, params.requestedLoadSize);
        callback.onResult(items);
    }
}
```

接着，您就可以通过实现一个 `DataSource.Factory` 子类的方式，把这些定制的数据加载到 `PagedList` 对象。如下的代码展示如何生成上述自定义数据源的新实例：

```java
public class ConcertTimeDataSourceFactory
        extends DataSource.Factory<Date, Concert> {
    private MutableLiveData<ConcertTimeDataSource> mSourceLiveData =
            new MutableLiveData<>();

    @Override
    public DataSource<Date, Concert> create() {
        ConcertTimeDataSource source =
            new ConcertTimeDataSource();
        mSourceLiveData.postValue(source);
        return source;
    }
}
```

## 考虑内容更新是如何工作的

当您构建可观察的 `PagedList` 对象时，考虑内容更新是如何工作的。如果您是直接从一个 [Room 数据持久化库](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_7_Room_Persistence_Library.md) 加载数据，那么内容更新就会被自动推送到您应用的 UI。

如果您使用的是网络 API，那么一般都会使用”下拉刷新“这样的用户操作，来充当作废当前 `DataSource` 并请求新数据的信号。如下的代码展示了这种行为：

```java
public class ConcertActivity extends AppCompatActivity {
    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        // ...
        mViewModel.getRefreshState()
                .observe(this, new Observer<NetworkState>() {
            // 本例展示了触发刷新操作的一种可能的做法。
            @Override
            public void onChanged(@Nullable MyNetworkState networkState) {
                swipeRefreshLayout.isRefreshing =
                        networkState == MyNetworkState.LOADING;
            }
        });

        swipeRefreshLayout.setOnRefreshListener(new SwipeRefreshListener() {
            @Override
            public void onRefresh() {
                mViewModel.invalidateDataSource();
            }
        });
    }
}

public class ConcertTimeViewModel extends ViewModel {
    private LiveData<PagedList<Concert>> mConcertList;
    private DataSource<Date, Concert> mConcertTimeDataSource;

    public ConcertTimeViewModel(Date firstConcertStartTime) {
        ConcertTimeDataSourceFactory dataSourceFactory =
                new ConcertTimeDataSourceFactory(firstConcertStartTime);
        mConcertTimeDataSource = dataSourceFactory.create();
        mConcertList = new LivePagedListBuilder<>(dataSourceFactory, 20)
                .setFetchExecutor(myExecutor)
                .build();
    }

    public void invalidateDataSource() {
        mConcertTimeDataSource.invalidate();
    }
}
```

## 提供数据表示之间的映射

分页库支持针对 `DataSource` 加载的数据的基于项目和基于分页的数据转换。

下列代码展示了把演唱会名称和日期的组合映射到含有名称和日期的单一字符串的映射：

```java
public class ConcertViewModel extends ViewModel {
    private LiveData<PagedList<String>> mConcertDescriptions;

    public ConcertViewModel(MyDatabase database) {
        DataSource.Factory<Integer, Concert> factory =
                database.allConcertsFactory().map(concert ->
                    concert.getName() + "-" + concert.getDate());
        mConcertDescriptions = new LivePagedListBuilder<>(
            factory, /* 分页大小 */ 30).build();
    }
}
```

当您想把加载好的数据包装、转换或者准备时，该功能就会非常有用。由于这些工作都是由加载器执行的，您可以借此完成高成本的任务，例如从硬盘读取或者查询另一个数据库。

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
