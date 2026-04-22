---
trigger: always_on
description: 开发需求要，要先了解整个项目的结构和可用的模块，记住下方模块的要求和特点
---

# Architecture Rules

## Modules

开发需求要，要先了解整个项目的结构和可用的模块，记住下方模块的要求和特点

### core/common
- 存放 `DataStore<Preferences>` 和用户 token 等，另外多语言文本（/res/values/ 和 /res/values-zh-rCN）和图片资源等公共资源也放在这个模块

### core/ui
- 存放所有自定义控件，主题，颜色等相关内容，特别注意，下拉刷新和加载更多控件需要统一使用 GSYPullRefresh, 目前有 `GSYGeneralLoadState`, `GSYPullRefresh`, `GSYTopAppBar`, `GSYNavigator`, `GSYNavHost`, `GSYLoadingDialog` 等通用 UI 组件

### core/network
- 是网络请求模块 ，网络数据的实体都在这个模块的 model/ 目录下，接口地址是 api/ 下的 GitHubApiService，同时也有 config 配置，比如  PAGE_SIZE

### core/database
- 是所有数据库模块，包括所有数据库能力，有 xxDao、xxEntiny，而每次修改数据库如果设计增删字段，需要修改增加 AppDatabase 的数据库版本

### data
- 模块是数据操作处理，包括所有 xxxRepository ，另外所有数据库的  toEntity 和 toXXX 网络数据的实体，都写在 mapper/DataMappers.kt 内统一处理
- data 模块不存放实体 Model , 实体 Model 在 core/network 的 model/ 目录下

### feature
- 模块是页面功能模块，内部每个模块每个模块的页面 xxxScreen 和 xxxViewModel

## 一个常规页面模块结构

最重要的一点， import 某个 class 的时候，要确定这个路径是否存在和正确，比如 `BaseUiState` 和  `BaseViewModel` 的路径是：
```
import com.shuyu.gsygithubappcompose.data.repository.vm.BaseUiState
import com.shuyu.gsygithubappcompose.data.repository.vm.BaseViewModel
```

如果使用了 `BaseViewModel`, 则需要同步使用 `BaseScreen` 来包裹，从而实现 Toast 展示

### 1. 模块职责划分

*   **`feature/xxx` 模块**:
  *   `xxxScreen.kt`: 负责 UI 层的展示，接收 `xxxViewModel` 提供的 UI 状态并渲染。
  *   `xxxViewModel.kt`: 负责 UI 逻辑和数据编排，管理 UI 状态，与 `xxxRepository` 交互。
*   **`data` 模块**:
  *   `xxxRepository.kt`: 负责数据访问逻辑，协调网络和数据库，实现数据缓存策略。
*   **`core/network` 模块**:
  *   `model/`: 存放网络数据实体。
  *   `api/GitHubApiService`: 定义网络接口。
*   **`core/database` 模块**:
  *   `dao/`: 存放数据访问对象 (DAO)。
  *   `entity/`: 存放数据库实体。
*   **`core/ui` 模块**:
  *   存放所有自定义控件，主题，颜色等相关内容。
  *   提供 `GSYGeneralLoadState`, `GSYPullRefresh`, `GSYTopAppBar`, `GSYNavigator`, `GSYNavHost`, `GSYLoadingDialog` 等通用 UI 组件。
*   **`core/common` 模块**:
  *   存放 `DataStore<Preferences>` 和用户 token 等。
  *   多语言文本（`/res/values/` 和 `/res/values-zh-rCN`）和图片资源等公共资源。

### 2. `xxxScreen.kt` (Composable UI) 实现细节

*   **ViewModel 注入**:
  *   在 Composable 函数签名中使用 `viewModel: XxxViewModel = hiltViewModel()`。
  *   确保导入 `androidx.hilt.lifecycle.viewmodel.compose.hiltViewModel`。
*   **UI 状态观察**:
  *   使用 `val uiState by viewModel.uiState.collectAsState()` 从 ViewModel 收集 UI 状态。
  *   `uiState` 通常是一个 `data class`，包含页面所需的所有状态信息（数据列表、加载状态、错误信息等）。
*   **初始数据加载**:
  *   使用 `LaunchedEffect(Unit) { viewModel.doInitialLoad() }` 在 Composable 首次组合时触发初始数据加载。
  *   对于需要参数的页面（如 `RepoDetailInfoScreen`），`LaunchedEffect` 的 key 应包含这些参数，例如 `LaunchedEffect(owner, name) { viewModel.loadRepoDetailInfo(owner, name) }`，以确保参数变化时重新加载数据。
*   **通用加载状态处理**:
  *   使用 `GSYGeneralLoadState` 包裹主要内容，处理首次加载、错误和重试机制。
  *   `isLoading` 参数应根据 `uiState.isPageLoading` 和数据是否为空来判断。
    ```kotlin
    GSYGeneralLoadState(
        isLoading = uiState.isPageLoading && uiState.repositories.isEmpty(), // 根据实际 UI 状态调整
        error = uiState.error,
        retry = { viewModel.refresh() }
    ) {
        // 主要内容在此处，例如 GSYPullRefresh
    }
    ```
*   **下拉刷新和加载更多**:
  *   使用 `GSYPullRefresh` 实现刷新和加载更多功能。
  *   参数如 `isRefreshing`, `onRefresh`, `isLoadMore`, `onLoadMore`, `hasMore`, `itemCount`, `loadMoreError` 都应绑定到 `uiState` 中的对应状态。
    ```kotlin
    GSYPullRefresh(
        isRefreshing = uiState.isRefreshing,
        onRefresh = { viewModel.refresh() },
        isLoadMore = uiState.isLoadingMore, // 根据 ViewModel 状态判断是否正在加载更多
        onLoadMore = { viewModel.loadMore() },
        hasMore = uiState.hasMore,
        itemCount = uiState.repositories.size,
        loadMoreError = uiState.loadMoreError,
        contentPadding = PaddingValues(5.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        // 条件性 UI 渲染，例如根据 uiState.repoDetail 是否为空显示头部信息
        uiState.repoDetail?.let { headerData ->
            item {
                RepositoryDetailInfoHeader(repositoryDetailModel = headerData)
            }
        }
        // 动态内容渲染，如 SegmentedButton
        item {
            SegmentedButton(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(horizontal = 16.dp, vertical = 8.dp),
                items = listOf(stringResource(R.string.events), stringResource(R.string.commits)),
                selectedIndex = when (uiState.selectedItemType) {
                    RepoDetailItemType.EVENT -> 0
                    RepoDetailItemType.COMMIT -> 1
                },
                onItemSelected = { index ->
                    when (index) {
                        0 -> viewModel.setSelectedItemType(RepoDetailItemType.EVENT)
                        1 -> viewModel.setSelectedItemType(RepoDetailItemType.COMMIT)
                    }
                }
            )
        }
        items(uiState.repoDetailList) { item ->
            // 显示单个项目，通常会有一个对应的 Composable Item
            when (item) {
                is RepoDetailListItem.EventItem -> {
                    EventItem(event = item.event)
                }
                is RepoDetailListItem.CommitItem -> {
                    CommitItem(commit = item.commit)
                }
            }
        }
    }
    ```
*   **标题栏**:
  *   使用 `GSYTopAppBar` 作为页面标题栏。
*   **导航**:
  *   导入 `import com.shuyu.gsygithubappcompose.core.ui.LocalNavigator`。
  *   通过 `val navigator = LocalNavigator.current` 获取导航器实例。
  *   使用 `navigator.navigate()` 或 `navigator.replace()` 进行页面跳转。
*   **Toast**：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CarGuo/GSYGithubAppCompose](https://github.com/CarGuo/GSYGithubAppCompose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
