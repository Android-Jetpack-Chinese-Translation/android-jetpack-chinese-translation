# 分页（Paging）库概览
原文链接：[Paging library overview  |  Android Developers](https://developer.android.google.cn/topic/libraries/architecture/paging/)


分页库能让您轻松地把数据批量而优雅地加载到您应用中的 [`RecyclerView`](https://developer.android.com/reference/android/support/v7/widget/RecyclerView?hl=zh-cn)。

许多应用从一个含有大量项目的数据源中加载数据，但每次只显示其中的一小部分。

Paging 库让您的应用观察并展示一个合适的数据子集。该功能有如下的优点：

* 应用的数据请求会消耗更少的网络带宽和系统资源，有助于获得那些流量有限的用户的青睐。
* 在数据更新时，应用仍能快速响应用户的输入操作。

如果您的应用已含有数据分页和展示列表的逻辑，请参阅下面的“更新您已有的应用”。

> **注意**：欲把分页库导入到您的 Android 项目中，请参阅[为您的项目添加 Android 架构组件](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_2_Adding_Components_to_your_Project.md)。

本节提供了如何使用分页库来请求和展示用户想要的数据、同时更经济节省地消耗系统资源的概览。欲了解更多关于您应用架构的特定层次，请参阅如下的页面：

* [UI 组件及其考虑因素](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_7_Paging_library/3_2_7_2_UI_Components_and_Considerations.md)
* [数据组件及其考虑因素](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_7_Paging_library/3_2_7_3_Data_Components_and_Considerations.md)

> **注意**：分页库帮您在 UI 的列表容器中平滑地展示数据，无论您是只使用了设备内部的数据库、或者是从后端服务器获取数据。欲了解更多有关如何基于您应用的数据来源而选择最佳的类库的信息，请参阅下面的“支持不同类型的数据架构”。

## 分页库的框架

分页库的关键组件是 [`PagedList`](https://developer.android.com/reference/android/arch/paging/PagedList?hl=zh-cn) 类，它是一个异步加载您应用数据块（即：页，page）的集合。该类充当了您应用架构中不同组件的中间人。

- 数据

	- 每个 `PagedList` 实例都从其 [`DataSource`](https://developer.android.com/reference/android/arch/paging/DataSource?hl=zh-cn) 中加载您应用数据的一个最新的快照。数据从您应用的后端服务器或数据库流向 `PagedList` 对象。

	- 分页库支持许多种类的应用架构，包括独立的数据库和与后端服务器通讯的数据库。

	- 欲了解更多信息，请参阅[数据组件及其考虑因素](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_7_Paging_library/3_2_7_3_Data_Components_and_Considerations.md)。

- UI

	- `PagedList` 和 `PagedListAdapter` 一起把数据项目加载到 `RecyclerView` 中。这些类齐心协力地加载数据内容，在加载完毕时展示它们，提前加载还没进入视图窗口的内容，并负责绘制内容变化时的动画。

	- 欲了解更多信息，请参阅[UI 组件及其考虑因素](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_7_Paging_library/3_2_7_2_UI_Components_and_Considerations.md)。

分页库实现了[应用架构指南](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/ANDROID_JETPACK/B_Get_started/2_Guide_to_app_architecture.md)所推荐的观查者模式。特别地，该库的核心组件创建了 `LiveData<PagedList>` （或 RxJava2 中的相应 ）类的实例，以便 UI 来观查。接下来，您应用的 UI 就可以在 `PagedList` 对象被生成的时候展示内容，而全程都不会违反 UI 控制器的[生命周期](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_4_Handling_Lifecycles.md)。

## 支持不同的数据架构

分页库支持应用的各种数据架构：无论是您的应用是只从后端服务器、只从设备上的数据库，或者同时从两个数据源获取数据。本小节提供了针对以上各种情形的推荐模式。

我们还在 GitHub 上提供了 [PagingWithNetwork](https://github.com/googlesamples/android-architecture-components/tree/master/PagingWithNetworkSample) 的例子，用于展示针对不同数据架构的推荐模式。

### 只有网络

欲展示从后端服务器获得的数据，请使用异步版本的 [Retrofit API](http://square.github.io/retrofit/) 来把信息加载到[您自定义的 `DataSource` 对象](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_7_Paging_library/3_2_7_3_Data_Components_and_Considerations.md)。

> **注意**：分页库的 `DataSource` 对象并不提供任何错误处理的功能，因为不同的应用处理和表示错误的方法都不一样。如果发生了错误，请延迟您处理加载结果的回调，并过一会儿重试。欲了解这种行为的例子，请参阅 [PagingWithNetwork](https://github.com/googlesamples/android-architecture-components/tree/master/PagingWithNetworkSample)。

### 只有数据库

将您的 `RecyclerView` 设置为观测本地存储，最好使用 [Room 数据持久化库](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_8_Room_Persistence_Library.md)。这样的话，无论您的数据库何时新增或修改了数据，这些变动都会自动反映到正在显示该数据的 `RecyclerView`中。

### 网络和数据库都有

当您开始观测数据库后，您可以使用 [`PagedList.BoundaryCallback`](https://developer.android.com/reference/android/arch/paging/PagedList.BoundaryCallback?hl=zh-cn) 来监听数据库的数据是否已经全部加载完毕。您可以从网络中获取更多的项目并将其插入数据库。如果您的 UI 正在观测数据库，那么您就无须烦劳操心了。

如下的代码展示了使用 boundary callback 的例子：

```java
public class ConcertViewModel {
    public ConcertSearchResult search(String query) {
        ConcertBoundaryCallback boundaryCallback =
                new ConcertBoundaryCallback(query, myService, myCache);
        // 使用 LiveData 对象来把网络状态通讯给应用，如下例所示。
        // 请注意该例中并不包含错误的处理。
        // LiveData<NetworkState> loadingState =
        //      boundaryCallback.getLoadingState();
    }
}

public class ConcertBoundaryCallback
        extends PagedList.BoundaryCallback<Concert> {
    private String mQuery;
    private MyService mService;
    private MyLocalCache mCache;

    public ConcertBoundaryCallback(String query, MyService service,
            MyLocalCache cache) {
        mQuery = query;
        // ...
    }

    // 从网络中请求最初的数据，并用其来取代数据库中当前的所有内容。
    @Override
    public void onZeroItemsLoaded() {
        requestAndReplaceInitialData(mQuery);
    }

    // 从网络中请求额外的数据，并将其添加到数据库中已有项目的末尾。
    @Override
    public void onItemAtEndLoaded(@NonNull Concert itemAtEnd) {
        requestAndAppendData(mQuery, itemAtEnd.key);
    }
}
```

欲了解分页库解决方案从网络和数据库获取数据的更加详细的例子，请导航到[分页库 CodeLab](https://codelabs.developers.google.com/codelabs/android-paging/index.html?hl=zh-cn)，或参阅 GitHub 上的 [PagingWithNetwork](https://github.com/googlesamples/android-architecture-components/tree/master/PagingWithNetworkSample)。

## 处理网络错误

在使用分页库来从网络加载数据或者将其分页显示时，请务必不要总是把网络视作“可用”和“不可用”的二元状态，因为许多时候网络连接是时断时续或不稳定的：

* 某些服务器也许没能相应网络请求。
* 设备可能连接到了一个网速缓慢或者信号弱的网络。

相反，您的应用应检查每个网络请求是否失败、并尽可能优雅地在网络不可用的情况下恢复。例如，您可以提供一个“重试”按钮来，在数据刷新的步骤也不起作用的时候供用户手动选择。如果数据分页的步骤出现了错误，最好自动重试分页请求。

## 更新您已有的应用

如果您的应用已经从数据库或者后端服务器的数据源来获取数据的话，您可以直接将其升级到分页库提供的功能。本小节展示了如何升级一个已有常见设计模式的应用。

### 自定义的分页解决方案

如果您使用自己定义的功能来从应用的数据源加载少量的数据子集，那么您可以把它们替换成 `PagedList` 类中的相应逻辑。`PagedList` 实例内建了连接到常见数据源的功能，而且还提供了您的 UI 中可能使用的 `RecyclerView` 的适配器。

### 使用列表而非分页加载数据

如果您使用一个内存中的列表来作为 UI 适配器的支持数据结构，那么在列表可能变得很大的情况下，请考虑使用一个 `PagedList` 来观察数据的更新。`PagedList` 实例可以使用 `LiveData<PagedList>` 或者 `Observable<List>` 来把数据更新传入您应用的 UI，尽可能减少数据加载时间和内存消耗。不过，尽量还是把 `List` 对象替换成 `PagedList` 比较好，因为这样无须劳烦您更改现有的应用 UI 结构或者数据更新逻辑。

### 使用 CursorAdapter 把一个数据游标（cursor）联系到相应的列表视图

您的应用可以使用一个 [`CursorAdapter`](https://developer.android.com/reference/android/widget/CursorAdapter?hl=zh-cn`) 来把 [`Cursor`](https://developer.android.com/reference/android/database/Cursor?hl=zh-cn) 的数据联系到 [`ListView`](https://developer.android.com/reference/android/widget/ListView?hl=zh-cn)。在这种情况下，您往往需要把应用中的列表 UI 容器从 `ListView` 迁移到 `RecyclerView` ；然后，根据 `Cursor` 是否访问的是 SQLite 数据库，来决定是把 `Cursor` 组件替换成 [`Room`](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_8_Room_Persistence_Library.md) 还是 `PositionalDataSource`。

在某些情况下，例如使用 [`Spinner`](https://developer.android.com/reference/android/widget/Spinner?hl=zh-cn) 的时候，您只需提供适配器自身，某个库就会接着处理该适配器中加载的数据并展示给您。这时，把您适配器的数据的类型改成 `LiveData<PagedList>`，然后将其包装到一个 [`ArrayAdapter`](https://developer.android.com/reference/android/widget/ArrayAdapter?hl=zh-cn) 对象中，最后再使用库将这些数据填充到 UI 中。

### 使用 AsyncListUtil 异步加载内容

如果您使用 [`AsyncListUtil`](https://developer.android.com/reference/android/support/v7/util/AsyncListUtil?hl=zh-cn) 对象来异步加载并展示分组的信息，分页库能让您的工作更轻松：

* **您的数据无须是按顺序排列的。**分页库让您使用键（key）直接从后端服务器加载数据。
* **您的数据量无须特别巨大。**使用分页库，您可以不停加载数据到分页中，直到用光数据源。
* **您可以更轻易地观察数据。**分页库能展示您应用的 ViewModel 中储存的可观察的数据结构。

> **注意**：如果您的应用访问的是 SQLite 数据库，请参阅 [`Room 数据持久化`](https://github.com/Android-Jetpack-Chinese-Translation/android-jetpack-chinese-translation/blob/master/DOCS/B_Guides/3_Core_topics/3_2_Architecture_Components/3_2_8_Room_Persistence_Library.md)。


## 数据库范例

如下的代码范例展示了一些可能的整合所有模块的方法：

### 使用 LiveData 来观察分页的数据

如下例所示，当数据库中的演唱会事件被添加、移除或者修改时，`RecyclerView` 中的内容会被自动而高效地更新：

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
            // Null 定义了一个占位符项目：当真正的项目从数据库中加载出来时，
            // PagedListAdapter 会自动把这行作废。
            holder.clear();
        }
    }

    private static DiffUtil.ItemCallback<Concert> DIFF_CALLBACK =
            new DiffUtil.ItemCallback<Concert>() {
        // 从数据库中重新加载演唱会后，其细节可能已经发生了变化，
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

如果您更愿意选择 [RxJava2](https://github.com/ReactiveX/RxJava) 而不是 `LiveData`，您就可以创建一个 `Observable` 或者 `Flowable` 对象：

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

您接下来就可以使用如下的代码开始或停止观察数据：

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

基于 RxJava2 和 LiveData 的解决方案里的 `ConcertDao` 和 `ConcertAdapter` 的代码是一样的。

## 其他资源

欲了解更多有关分页库的信息，请参阅以下教程和视频：

* [CodeLab](https://codelabs.developers.google.com/codelabs/android-paging/index.html?hl=zh-cn)：学习如何按部就班地把分页库添加到应用中。
* [PagingWithNetwork](https://github.com/googlesamples/android-architecture-components/tree/master/PagingWithNetworkSample)：学习如何让分页库与后端 API 合作。
* [Google I/O 会议]：分页库的的介绍视频。






