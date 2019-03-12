# 分页（Paging）库概览
原文链接：[Paging library overview  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/paging/)

分页库可帮助您一次加载和显示少量数据。 按需加载部分数据可减少网络带宽和系统资源的使用。

本指南提供了库的几个概念性示例，以讲解它如何工作。 要查看此库如何运行的完整示例，请尝试使用 codelab 或者从 [其他资源](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_7_Paging_library/3_2_7_1_Overview.md#其他资源) 中获取示例。

## 分页库架构

本节介绍并展示分页库的主要组件。

- PagedList

分页库的关键组件是 [`PagedList`](https://developer.android.google.cn/reference/androidx/paging/PagedList) 类，它分块加载应用程序的数据或页面。当需要更多数据时，会被加载到已有的 `PagedList` 对象当中。 如果任何加载的数据发生更改，则会从 [`LiveData`](https://developer.android.google.cn/reference/androidx/lifecycle/LiveData) 或基于 RxJava2 的对象向可观察数据持有者发射新的 `PagedList` 实例。[`PagedList`](https://developer.android.google.cn/reference/androidx/paging/PagedList) 对象一旦生成，应用的 UI 就会展示内容，同时遵循 UI 控制器的 [生命周期](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_4_Handling_Lifecycles.md)。

以下代码片段展示了如何使用 PagedList 对象的 LiveData 持有者来配置应用程序的视图模型（view model），以便加载和显示数据：

```java
public class ConcertViewModel extends ViewModel {
    private ConcertDao concertDao;
    public final LiveData<PagedList<Concert>> concertList;

    // 创建一个每页 50 条记录的 PageList 对象
    public ConcertViewModel(ConcertDao concertDao) {
        this.concertDao = concertDao;
        concertList = new LivePagedListBuilder<>(
                concertDao.concertsByDate(), 50).build();
    }
}
```

- 数据

每个 [`PagedList`](https://developer.android.google.cn/reference/android/arch/paging/PagedList) 实例都从相应的 [`DataSource`](https://developer.android.google.cn/reference/android/arch/paging/DataSource) 对象中加载应用数据的最新快照。数据从应用的后端服务器或数据库流向 `PagedList` 对象。

以下示例使用 [Room 数据持久化库](https://developer.android.google.cn/training/data-storage/room) 来组织应用程序的数据，但如果您想使用其他方法存储数据，还可以提供自己的 [数据源工厂](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_7_Paging_library/3_2_7_3_Data_Components_and_Considerations.md#构建您自己的数据源)。

```java
@Dao
public interface ConcertDao {
    // 整型参数告诉 Room 应当使用
    // PositionalDataSource 对象
    @Query("SELECT * FROM concerts ORDER BY date DESC")
    DataSource.Factory<Integer, Concert> concertsByDate();
}
```

欲了解更多有关加载数据到 `PagedList` 对象的信息，请参阅[数据组件和注意事项](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_7_Paging_library/3_2_7_3_Data_Components_and_Considerations.md)。

- UI

`PagedList` 使用 `PagedListAdapter` 把数据条目（item）加载到 `RecyclerView` 中。这些类齐心协力地加载数据和展示结果，以及预读取视图内容，并负责绘制内容变化时的动画。

欲了解更多信息，请参阅[界面组件和注意事项](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_7_Paging_library/3_2_7_2_UI_Components_and_Considerations.md)。

## 支持不同的数据架构

分页库支持下列数据架构：
- 仅从后端网络服务器提供。
- 仅存储在设备上的数据库中。
- 使用设备上数据库作为缓存并与其他数据源组合的形式。

图1显示了每种架构方案中数据的流动方式。 对于仅限网络或仅限数据库的解决方案，数据直接流向应用程序的 UI 模型（model）。 如果您使用的是组合方法，则数据会先从后端服务器流入设备上的数据库，再流入应用程序的 UI 模型。 每隔一段时间，每个数据流末端就会耗尽要加载的数据，此时它会从提供数据的组件中请求更多数据。 例如，当用完设备上数据库的数据后，它会从服务器请求更多数据。

![paging-library-data-flow](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/IMGS_BACKUP/paging-library-data-flow.png)

**图1** 数据如何流经分页库支持的每个体系结构

### 只有网络

欲展示从后端服务器获得的数据，请使用同步版本的 [Retrofit API](http://square.github.io/retrofit/) 来把信息加载到 [您自定义的 `DataSource` 对象](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_7_Paging_library/3_2_7_3_Data_Components_and_Considerations.md) 中。

> **注意**：分页库的 [`DataSource`](https://developer.android.google.cn/reference/android/arch/paging/DataSource) 对象并不提供任何错误处理的功能，因为不同的应用处理和表示错误的方法都不一样。如果发生了错误，请延迟处理结果回调，并过一会儿重试。有关此行为的示例，请参阅 [PagingWithNetwork sample](https://github.com/googlesamples/android-architecture-components/tree/master/PagingWithNetworkSample)。

### 只有数据库

设置 `RecyclerView` 观测本地存储，最好使用 [Room 数据持久化库](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_8_Room_Persistence_Library.md)。这样的话，应用程序的数据库无论何时新增或修改了数据，这些变动都会自动反映到正在显示该数据的 `RecyclerView` 中。

### 网络和数据库都有

当您开始观测数据库后，您可以使用 [`PagedList.BoundaryCallback`](https://developer.android.google.cn/reference/android/arch/paging/PagedList.BoundaryCallback) 来监听数据库的数据是否已经全部加载完毕。您可以从网络获取更多的条目（item）并将其插入数据库。如果您的 UI 正在观测数据库，那么您就无须烦劳操心了。

## 处理网络错误

在使用分页库来从网络加载数据或者将其分页显示时，请务必不要总是把网络视作“可用”和“不可用”的二元状态，因为许多时候网络连接是时断时续或不稳定的：
- 某些服务器也许没能响应网络请求。
- 设备可能连接到了一个网速缓慢或者信号弱的网络。

相反，您的应用应检查每个失败请求，并在网络不可用的情况下尽可能优雅地恢复。例如，您可以提供“重试”按钮，在数据刷新的步骤不起作用时供用户手动选择。如果数据分页的步骤出现了错误，最好自动重试分页请求。

## 更新您已有的应用

如果您的应用已经从数据库或者后端服务器的数据源来获取数据的话，您可以直接将其升级到分页库提供的功能。本小节展示了如何升级一个已有常见设计模式的应用。

### 自定义的分页解决方案

如果您使用自定义的功能来从应用的数据源加载少量的数据子集，那么您可以把它们替换成 `PagedList` 类中的相应逻辑。`PagedList` 实例内建了连接到常见数据源的功能，而且还提供了您的 UI 中可能使用的 `RecyclerView` 的适配器。

### 使用列表而非分页加载数据

如果您使用内存列表来作为 UI 适配器的支持数据结构，那么在列表可能变得很大的情况下，请考虑使用 [`PagedList`](https://developer.android.google.cn/reference/android/arch/paging/PagedList) 来观察数据的更新。`PagedList` 实例可以使用 [`LiveData<PagedList>`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData) 或者 `Observable<List>` 来把数据更新传递到 UI，尽可能减少了数据加载时间和内存消耗。不过，尽量还是把 [`List`](https://developer.android.google.cn/reference/java/util/List) 对象替换成 `PagedList` 比较好，因为这样无须劳烦您更改现有的应用 UI 结构或者数据更新逻辑。

### 使用 CursorAdapter 将数据游标（cursor）与列表视图相关联

您的应用可以使用 [`CursorAdapter`](https://developer.android.google.cn/reference/android/widget/CursorAdapter) 来把 [`Cursor`](https://developer.android.google.cn/reference/android/database/Cursor) 的数据关联到 [`ListView`](https://developer.android.google.cn/reference/android/widget/ListView) 中。在这种情况下，您通常需要把应用中的列表 UI 容器从 `ListView` 迁移到 [`RecyclerView`](https://developer.android.google.cn/reference/android/support/v7/widget/RecyclerView)，然后根据 `Cursor` 是否访问的是 SQLite 数据库，来决定是把其组件替换成 [`Room`](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_8_Room_Persistence_Library.md) 还是 `PositionalDataSource`。

在某些情况下，例如使用 [`Spinner`](https://developer.android.google.cn/reference/android/widget/Spinner) 的时候，您只需提供适配器自身，某个库就会接着处理该适配器中加载的数据并展示给您。这时，把您适配器的数据的类型改成 [`LiveData<PagedList>`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData)，然后将其包装到一个 [`ArrayAdapter`](https://developer.android.google.cn/reference/android/widget/ArrayAdapter) 对象里，最后再使用库将这些数据填充到 UI 中。

### 使用 AsyncListUtil 异步加载内容

如果您使用 [`AsyncListUtil`](https://developer.android.google.cn/reference/android/support/v7/util/AsyncListUtil) 来异步加载并展示分组的信息，分页库能让您的工作更轻松：
- **您的数据无须是按顺序排列的。** 分页库让您使用键（key）直接从后端服务器加载数据。
- **您的数据量可以特别巨大。** 使用分页库，您可以不停加载数据到分页中，直到耗尽数据源。
- **您可以更轻易地观察数据。** 分页库能展示您应用的 ViewModel 中储存的可观察的数据结构。

> **注意**：如果您的应用访问的是 SQLite 数据库，请参阅 [`Room 数据持久化库`](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_8_Room_Persistence_Library.md)。

## 数据库示例

以下代码片段展示了一些可能的整合所有模块的方法。

### 使用 LiveData 来观察分页的数据

如下所示，当数据库中的演唱会事件（concert event）被添加、移除或者修改时，[`RecyclerView`](https://developer.android.google.cn/reference/android/support/v7/widget/RecyclerView.html) 中的内容会被自动而高效地更新：

```java
@Dao
public interface ConcertDao {
    // 整型参数告诉 Room 应当使用 PositionalDataSource 对象，
    // 并由 Room 加载基于位置的数据。
    @Query("SELECT * FROM concerts ORDER BY date DESC")
    DataSource.Factory<Integer, Concert> concertsByDate();
}

public class ConcertViewModel extends ViewModel {
    private ConcertDao mConcertDao;
    public final LiveData<PagedList<Concert>> concertList;

    public ConcertViewModel(ConcertDao concertDao) {
        mConcertDao = concertDao;
        concertList = new LivePagedListBuilder<>(
            mConcertDao.concertsByDate(), /* 分页大小 */ 20).build();
    }
}

public class ConcertActivity extends AppCompatActivity {
    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ConcertViewModel viewModel =
                ViewModelProviders.of(this).get(ConcertViewModel.class);
        RecyclerView recyclerView = findViewById(R.id.concert_list);
        ConcertAdapter adapter = new ConcertAdapter();
        viewModel.concertList.observe(this, adapter::submitList);
        recyclerView.setAdapter(adapter);
    }
}

public class ConcertAdapter
        extends PagedListAdapter<Concert, ConcertViewHolder> {
    protected ConcertAdapter() {
        super(DIFF_CALLBACK);
    }

    @Override
    public void onBindViewHolder(@NonNull ConcertViewHolder holder,
            int position) {
        Concert concert = getItem(position);
        if (concert != null) {
            holder.bindTo(concert);
        } else {
            // Null 定义了一个占位符条目（placeholder item）- 当真正的数据项
						// 从数据库中加载出来时，PagedListAdapter 会自动把这行作废。
            holder.clear();
        }
    }

    private static DiffUtil.ItemCallback<Concert> DIFF_CALLBACK =
            new DiffUtil.ItemCallback<Concert>() {
        // 从数据库中重新加载演唱会数据后，其细节可能已经发生了变化，
        // 然而其 ID 仍然是固定不变的。
        @Override
        public boolean areItemsTheSame(Concert oldConcert, Concert newConcert) {
            return oldConcert.getId() == newConcert.getId();
        }

        @Override
        public boolean areContentsTheSame(Concert oldConcert,
                Concert newConcert) {
            return oldConcert.equals(newConcert);
        }
    };
}
```

### 使用 RxJava2 观察分页的数据

如果您更愿意使用 [RxJava2](https://github.com/ReactiveX/RxJava) 而不是 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html)，您可以创建一个 `Observable` 或者 `Flowable` 对象：

```java
public class ConcertViewModel extends ViewModel {
    private ConcertDao mConcertDao;
    public final Flowable<PagedList<Concert>> concertList;

    public ConcertViewModel(ConcertDao concertDao) {
        mConcertDao = concertDao;

        concertList = new RxPagedListBuilder<>(
                mConcertDao.concertsByDate(), /* 分页大小 */ 50)
                        .buildFlowable(BackpressureStrategy.LATEST);
    }
}
```

您接下来就可以使用如下代码开始或停止观察数据：

```java
public class ConcertActivity extends AppCompatActivity {
    private ConcertAdapter mAdapter;
    private ConcertViewModel mViewModel;

    private CompositeDisposable mDisposable = new CompositeDisposable();

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        RecyclerView recyclerView = findViewById(R.id.concert_list);

        mViewModel = ViewModelProviders.of(this).get(ConcertViewModel.class);
        mAdapter = new ConcertAdapter();
        recyclerView.setAdapter(mAdapter);
    }

    @Override
    protected void onStart() {
        super.onStart();
        mDisposable.add(mViewModel.concertList.subscribe(
                flowableList -> mAdapter.submitList(flowableList)
        ));
    }

    @Override
    protected void onStop() {
        super.onStop();
        mDisposable.clear();
    }
}
```

`ConcertDao` 与 `ConcertAdapter` 的代码在 [RxJava2](https://github.com/ReactiveX/RxJava) 和 [`LiveData`](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html) 这两种方案中是一致的。

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
