# 🤖 Claude Code + Android Studio 两天全自动 Android App 开发 SOP

> **适用对象**: Claude Code (AI Agent) 全自动执行
> **目标**: 从零到 Google Play 上架，两天 (16 小时) 完成
> **技术栈**: Kotlin + Jetpack Compose + MVVM + Room
> **最低要求**: Android Studio Hedgehog+, API 34+, Google Play 开发者账号 ($25 一次性)

---

## 🧠 Claude Code 技能调用矩阵

| 技能标识 | 技能名称 | 说明 | Claude Code 工具 |
|---------|---------|------|-----------------|
| `[SHELL]` | Shell 执行 | 执行 bash、gradlew、adb、git | `Bash` |
| `[WRITE]` | 文件写入 | 创建/修改 .kt .kts .xml .gradle 等 | `Edit` |
| `[READ]` | 文件读取 | 读取现有代码、日志、配置 | `Read` |
| `[DIALOG]` | 用户交互 | 提问、确认、展示结果 | 对话输出 |
| `[GENERATE]` | 代码生成 | 生成 Kotlin/Compose 代码 | `Edit` |
| `[REVIEW]` | 代码审查 | 审查代码质量、Material Design 合规 | `Read`→分析 |
| `[DEBUG]` | 调试分析 | 分析编译错误并修复 | `Read`+`Bash`+`Edit` |
| `[RESEARCH]` | 知识检索 | 查阅 Android 文档、MD3 规范 | 内置知识 |
| `[VALIDATE]` | 验证检查 | 清单逐项验证 | `Bash`+`Read` |
| `[GIT]` | 版本控制 | git add/commit/tag | `Bash` |

### 技能调用原则

```
1. 每个 Phase 开始前 → [RESEARCH] 查阅 Android/Kotlin/Compose 最佳实践
2. 每次 WRITE 后 → ./gradlew assembleDebug → [DEBUG] 自动修复
3. 编译失败 → 读日志 → 定位文件 → Edit 修复 → 重编译 (最多 5 次)
4. 每个 Phase 结束 → [VALIDATE] → [GIT] commit
5. 不可逆操作 → [DIALOG] 用户确认
```

### 全局编译验证

```bash
source /tmp/sop_android.env 2>/dev/null
cd ${PROJECT_DIR}
./gradlew assembleDebug 2>&1 | tail -20
# [DEBUG] 分析 BUILD FAILED  → 修复 → 重编译
```

---

## 📋 总览时间线

```
Day 1 (8h)                              Day 2 (8h)
├─ [0.0h] 环境检查 & 项目初始化          ├─ [0.0h] Day 2 启动校验
├─ [0.5h] 产品需求确认                   ├─ [0.5h] 集成测试 & 边界情况
├─ [1.0h] PRD 产品需求文档               ├─ [2.0h] 性能优化 & 无障碍
├─ [1.5h] 高保真原型 (Compose Preview)  ├─ [3.0h] Google Play 素材准备
├─ [2.5h] 架构搭建 (MVVM + Hilt)        ├─ [4.0h] 内购 (Google Play Billing)
├─ [3.5h] 数据层 (Room + DataStore)     ├─ [5.0h] 构建签名 & 内部测试
├─ [5.5h] ViewModel + Repository        ├─ [6.5h] Google Play Console 提交
├─ [7.0h] UI 层 (Jetpack Compose)       └─ [8.0h] 归档 & 文档
└─ [8.0h] 自测 & 代码 Review
```

---

## ⚠️ 执行前提

```bash
# [SHELL] 1. 确认 Android Studio 已安装
if [ -d "/Applications/Android Studio.app" ]; then
    echo "✅ Android Studio 已安装"
else
    echo "❌ 请从 https://developer.android.com/studio 下载"
    exit 1
fi

# [SHELL] 2. 确认 JDK 17+
java -version 2>&1 | grep "17\|21" || echo "⚠️ 需要 JDK 17+"

# [SHELL] 3. 确认 Git
git --version || { echo "❌ Git 未安装"; exit 1; }

# [SHELL] 4. Google Play 账号
echo "请确认: 已注册 Google Play 开发者账号 ($25)"
# [DIALOG] 等待用户确认
```

---

## Phase 0: 环境初始化 (0:00-0:30)

> **[SHELL]** + **[DIALOG]**

### Step 0.1 — 创建项目

```bash
PROJECT_NAME="MyAndroidApp"
PACKAGE_NAME="com.example.myandroidapp"
DISPLAY_NAME="我的Android App"

cat > /tmp/sop_android.env << ANDROIDENV
PROJECT_NAME="$PROJECT_NAME"
PACKAGE_NAME="$PACKAGE_NAME"
DISPLAY_NAME="$DISPLAY_NAME"
PROJECT_DIR="$(pwd)/$PROJECT_NAME"
ANDROIDENV
```

> **[DIALOG]** Android Studio → New Project → Empty Activity → Kotlin + Jetpack Compose → API 34+

### Step 0.2 — 验证 & 构建

```bash
source /tmp/sop_android.env 2>/dev/null && cd ${PROJECT_DIR}
./gradlew assembleDebug 2>&1 | tail -10
```

### Step 0.3 — 初始化 Git & 目录结构

```bash
cd ${PROJECT_DIR}
mkdir -p app/src/main/java/${PACKAGE_NAME//./\/}/{data/{local,model,repository},ui/{screens,components,theme},di}
git init && git checkout -b main
git add -A && git commit -m "Initial Android project"
```

---

## Phase 1: 产品需求确认 (0:30-1:00)

> **[DIALOG]** 结构化问卷，同 iOS/鸿蒙 SOP Phase 1

---

## Phase 1.2: PRD 产品需求文档 (1:00-1:30)

> **[GENERATE]** + **[WRITE]** 基于 SPECS.md 生成 PRD.md (7 章节，同 iOS SOP Phase 1.2)

---

## Phase 1.5: 高保真原型 (1:30-2:30)

> **[GENERATE]** Compose Preview 作为原型工具

```kotlin
// app/src/main/java/.../ui/screens/HomePrototype.kt
@Composable
@Preview(showSystemUi = true)
fun HomePrototype() {
    MaterialTheme {
        Scaffold(
            topBar = { TopAppBar(title = { Text("我的任务") }) },
            floatingActionButton = {
                FloatingActionButton(onClick = {}) {
                    Icon(Icons.Default.Add, "添加")
                }
            }
        ) { padding ->
            LazyColumn(modifier = Modifier.padding(padding)) {
                items(5) { index ->
                    TaskCard(
                        title = "任务 ${index + 1}",
                        priority = if (index < 2) "高" else "中",
                        isCompleted = index == 3
                    )
                }
            }
        }
    }
}
```

---

## Phase 2: 架构搭建 (2:30-3:00)

### Design Tokens + Theme

```kotlin
// ui/theme/Color.kt
object AppColors {
    val Primary = Color(0xFF007AFF)
    val Background = Color(0xFFF2F2F7)
    val Surface = Color(0xFFFFFFFF)
    val Error = Color(0xFFFF3B30)
    val Success = Color(0xFF34C759)
    val Warning = Color(0xFFFF9500)
}

// ui/theme/Type.kt — Material 3 Typography
// ui/theme/Theme.kt — 统一主题
@Composable
fun AppTheme(content: @Composable () -> Unit) {
    MaterialTheme(
        colorScheme = lightColorScheme(
            primary = AppColors.Primary,
            error = AppColors.Error,
            background = AppColors.Background,
            surface = AppColors.Surface
        ),
        content = content
    )
}
```

### Hilt DI 配置

> **[WRITE]** `app/build.gradle.kts` 添加 Hilt:

```kotlin
plugins { id("com.google.dagger.hilt.android") version "2.51" kotlin("kapt") }
dependencies {
    implementation("com.google.dagger:hilt-android:2.51")
    kapt("com.google.dagger:hilt-compiler:2.51")
    implementation("androidx.hilt:hilt-navigation-compose:1.2.0")
}
```

> **[WRITE]** `MyApplication.kt`:

```kotlin
@HiltAndroidApp
class MyApplication : Application()
```

> **[WRITE]** `di/AppModule.kt`:

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides @Singleton
    fun provideDatabase(@ApplicationContext context: Context): AppDatabase {
        return Room.databaseBuilder(context, AppDatabase::class.java, "app.db").build()
    }

    @Provides fun provideTaskDao(db: AppDatabase) = db.taskDao()
    @Provides fun provideRepository(dao: TaskDao) = TaskRepository(dao)
}
```

### Data Model

```kotlin
// data/model/TaskItem.kt
@Entity(tableName = "tasks")
data class TaskItem(
    @PrimaryKey val id: String = UUID.randomUUID().toString(),
    val title: String,
    val description: String = "",
    val priority: Int = 1,    // 0=LOW, 1=MEDIUM, 2=HIGH, 3=URGENT
    val isCompleted: Boolean = false,
    val dueDate: Long = 0,
    val createdAt: Long = System.currentTimeMillis()
)
```

---

## Phase 3: 数据层 (3:00-3:30)

### Room Database

```kotlin
// data/local/TaskDao.kt
@Dao
interface TaskDao {
    @Query("SELECT * FROM tasks ORDER BY created_at DESC")
    fun getAll(): Flow<List<TaskItem>>

    @Insert
    suspend fun insert(task: TaskItem)

    @Update
    suspend fun update(task: TaskItem)

    @Delete
    suspend fun delete(task: TaskItem)
}

// data/local/AppDatabase.kt
@Database(entities = [TaskItem::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun taskDao(): TaskDao
}
```

### Repository

```kotlin
// data/repository/TaskRepository.kt
class TaskRepository(private val dao: TaskDao) {
    val allTasks: Flow<List<TaskItem>> = dao.getAll()
    suspend fun insert(task: TaskItem) = dao.insert(task)
    suspend fun update(task: TaskItem) = dao.update(task)
    suspend fun delete(task: TaskItem) = dao.delete(task)
}
```

---

## Phase 4: ViewModel + DI (3:30-5:00)

```kotlin
// ui/screens/home/HomeViewModel.kt
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val repository: TaskRepository
) : ViewModel() {
    private val _searchQuery = MutableStateFlow("")
    private val _filterOption = MutableStateFlow(FilterOption.ALL)

    val uiState: StateFlow<HomeUiState> = combine(
        repository.allTasks, _searchQuery, _filterOption
    ) { tasks, query, filter ->
        val filtered = tasks
            .filter { when(filter) {
                FilterOption.ACTIVE -> !it.isCompleted
                FilterOption.COMPLETED -> it.isCompleted
                else -> true
            }}
            .filter { it.title.contains(query, ignoreCase = true) }
        HomeUiState(tasks = filtered)
    }.stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), HomeUiState())

    fun addTask(title: String) {
        viewModelScope.launch {
            repository.insert(TaskItem(title = title))
        }
    }
}

data class HomeUiState(val tasks: List<TaskItem> = emptyList())
```

---

## Phase 5: UI 层 (5:00-7:30)

```kotlin
// ui/screens/home/HomeScreen.kt
@Composable
fun HomeScreen(viewModel: HomeViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    Scaffold(
        topBar = { TopAppBar(title = { Text("我的任务") }) },
        floatingActionButton = {
            FloatingActionButton(onClick = { viewModel.addTask("新任务") }) {
                Icon(Icons.Default.Add, "添加任务")
            }
        }
    ) { padding ->
        if (uiState.tasks.isEmpty()) {
            EmptyState(modifier = Modifier.padding(padding))
        } else {
            LazyColumn(modifier = Modifier.padding(padding)) {
                items(uiState.tasks, key = { it.id }) { task ->
                    TaskCard(task = task)
                }
            }
        }
    }
}

// ui/components/TaskCard.kt
@Composable
fun TaskCard(task: TaskItem, modifier: Modifier = Modifier) {
    Card(modifier = modifier.fillMaxWidth().padding(horizontal = 16.dp, vertical = 4.dp)) {
        Row(modifier = Modifier.padding(16.dp), verticalAlignment = Alignment.CenterVertically) {
            Checkbox(checked = task.isCompleted, onCheckedChange = null)
            Column(modifier = Modifier.weight(1f).padding(horizontal = 12.dp)) {
                Text(task.title, style = MaterialTheme.typography.bodyLarge)
                if (task.dueDate > 0) {
                    Text("截止: ${formatDate(task.dueDate)}", color = MaterialTheme.colorScheme.error)
                }
            }
            PriorityBadge(priority = task.priority)
        }
    }
}
```

---

### DetailScreen & SettingsScreen

```kotlin
// ui/screens/detail/DetailScreen.kt
@Composable
fun DetailScreen(taskId: String?, onBack: () -> Unit, viewModel: HomeViewModel = hiltViewModel()) {
    val task = taskId?.let { viewModel.getTask(it) }
    var title by remember { mutableStateOf(task?.title ?: "") }
    var priority by remember { mutableIntStateOf(task?.priority ?: 1) }

    Scaffold(
        topBar = { TopAppBar(
            title = { Text(if (taskId != null) "编辑任务" else "新建任务") },
            navigationIcon = { IconButton(onClick = onBack) { Icon(Icons.AutoMirrored.Filled.ArrowBack, "返回") } },
            actions = { TextButton(onClick = {
                viewModel.saveTask(taskId, title, priority); onBack()
            }) { Text("保存") } }
        )}
    ) { padding ->
        Column(modifier = Modifier.padding(padding).padding(16.dp), verticalArrangement = Arrangement.spacedBy(16.dp)) {
            OutlinedTextField(value = title, onValueChange = { title = it }, label = { Text("任务标题") }, modifier = Modifier.fillMaxWidth())
            Text("优先级", style = MaterialTheme.typography.labelLarge)
            Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
                listOf("低" to 0, "中" to 1, "高" to 2, "紧急" to 3).forEach { (label, value) ->
                    FilterChip(selected = priority == value, onClick = { priority = value }, label = { Text(label) })
                }
            }
        }
    }
}

// ui/screens/settings/SettingsScreen.kt
@Composable
fun SettingsScreen(onBack: () -> Unit) {
    Scaffold(
        topBar = { TopAppBar(title = { Text("设置") }, navigationIcon = { IconButton(onClick = onBack) { Icon(Icons.AutoMirrored.Filled.ArrowBack, "返回") } }) }
    ) { padding ->
        LazyColumn(modifier = Modifier.padding(padding)) {
            item { SwitchRow("触觉反馈", checked = true, onCheckedChange = {}) }
            item { InfoRow("版本", "v1.0.0") }
            item { InfoRow("关于", "© 2026") }
        }
    }
}
```

---

## Phase 6: 自测 & Day 1 收尾 (7:30-8:00)

```bash
source /tmp/sop_android.env 2>/dev/null && cd ${PROJECT_DIR}
./gradlew test 2>&1 | tail -10
./gradlew assembleDebug 2>&1 | tail -5

cat > DAY1_REPORT.md << EOF
# Day 1 完成报告 — ${PROJECT_NAME}
- 架构: ✅ MVVM + Hilt DI + Room
- ViewModel: ✅ HomeViewModel + Flow
- UI: ✅ Jetpack Compose + Material 3
- 编译: ✅ assembleDebug 通过
EOF
git add -A && git commit -m "Day 1 complete: MVVM + Room + Compose"
```

---

## Phase 7: 集成测试 (Day 2, 0:00-1:00)

### Compose UI 测试

> **[WRITE]** `app/src/androidTest/java/.../HomeScreenTest.kt`:

```kotlin
@RunWith(AndroidJUnit4::class)
class HomeScreenTest {
    @get:Rule val composeTestRule = createComposeRule()

    @Test
    fun addTask_displaysInList() {
        composeTestRule.setContent { HomeScreen() }
        composeTestRule.onNodeWithText("暂无任务").assertExists()
        composeTestRule.onNodeWithText("添加任务...").performTextInput("Buy milk")
        composeTestRule.onNodeWithText("添加").performClick()
        composeTestRule.onNodeWithText("Buy milk").assertExists()
    }
}
```

```bash
# [SHELL] 运行测试
./gradlew connectedAndroidTest 2>&1 | tail -10
```

---

## Phase 8: 性能优化 (Day 2, 1:00-2:00)

```kotlin
// [WRITE] app/src/main/AndroidManifest.xml — 添加 profileable
<application android:profileable="true" ...>

// [WRITE] Baseline Profile 生成
// 在 Android Studio: Run → Generate Baseline Profile
```

```bash
# [SHELL] R8 混淆检查
./gradlew assembleRelease
# 检查 build/outputs/mapping/ 下的混淆映射
```

---

## Phase 9: Google Play 素材 (Day 2, 2:00-3:00)

```
□ 应用图标: 512×512 px PNG (Play Console → 图形资源)
□ Feature Graphic: 1024×500 px (商店首页横幅)
□ 截图: ≥ 2 张 (分辨率 ≥ 1080p)
□ 应用描述: 短描述 80字符 + 长描述 4000字符
□ 隐私政策 URL (必须, 可访问)
□ 内容分级问卷 (Play Console → 政策 → 应用内容)
```

---

## Phase 10: Google Play Billing (Day 2, 3:00-4:00)

```kotlin
// [WRITE] app/build.gradle.kts
dependencies {
    implementation("com.android.billingclient:billing-ktx:7.0.0")
}

// BillingManager.kt
class BillingManager(private val activity: Activity) {
    private val billingClient = BillingClient.newBuilder(activity)
        .setListener { billingResult, purchases ->
            // 处理购买结果
        }
        .enablePendingPurchases()
        .build()

    fun queryProducts() {
        val params = QueryProductDetailsParams.newBuilder()
            .setProductList(listOf(
                QueryProductDetailsParams.Product.newBuilder()
                    .setProductId("premium_monthly")
                    .setProductType(BillingClient.ProductType.SUBS)
                    .build()
            )).build()
        billingClient.queryProductDetailsAsync(params) { _, productDetailsList ->
            // 展示商品列表
        }
    }

    fun launchBillingFlow(productDetails: ProductDetails) {
        val params = BillingFlowParams.newBuilder()
            .setProductDetailsParamsList(listOf(
                BillingFlowParams.ProductDetailsParams.newBuilder()
                    .setProductDetails(productDetails)
                    .build()
            )).build()
        billingClient.launchBillingFlow(activity, params)
    }
}
```

---

## Phase 11: 签名 & App Bundle (Day 2, 4:00-5:00)

```bash
# [SHELL] 生成上传密钥 (仅首次)
keytool -genkey -v -keystore upload-keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 -alias upload

# [WRITE] app/build.gradle.kts — 签名配置
android {
    signingConfigs {
        create("release") {
            storeFile = file("upload-keystore.jks")
            storePassword = System.getenv("KEYSTORE_PASSWORD")
            keyAlias = "upload"
            keyPassword = System.getenv("KEY_PASSWORD")
        }
    }
    buildTypes {
        release {
            signingConfig = signingConfigs.getByName("release")
            isMinifyEnabled = true
            proguardFiles(getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro")
        }
    }
}

# [SHELL] 构建 App Bundle (推荐格式)
./gradlew bundleRelease
# 产物: app/build/outputs/bundle/release/app-release.aab
```

---

## Phase 12: Google Play Console 提交 (Day 2, 5:00-6:30)

> **[DIALOG]** Google Play 审核核心要求:

```
═══════════════════════════════════════
Google Play 提交清单 (26 项)
═══════════════════════════════════════

一、应用信息
□ 1.1 应用名称 (30字符内)
□ 1.2 短描述 (80字符) + 长描述 (4000字符)
□ 1.3 图标 (512×512) + Feature Graphic (1024×500)
□ 1.4 截图 ≥ 2 张 (手机 + 平板各至少1张)

二、构建 & 签名
□ 2.1 App Bundle (.aab) 已上传
□ 2.2 签名密钥安全保管 (upload keystore)
□ 2.3 versionCode 递增, versionName 正确
□ 2.4 targetSdkVersion ≥ 34 (Google Play 强制)

三、内容评级
□ 3.1 完成内容分级问卷
□ 3.2 年龄分级正确
□ 3.3 无违规内容

四、隐私 & 安全
□ 4.1 隐私政策 URL 可访问
□ 4.2 Data safety 标签已填写
□ 4.3 敏感权限声明完整
□ 4.4 无恶意软件

五、测试
□ 5.1 内部测试轨道已创建 (至少 20 位测试者 或 闭源测试 12人/14天)
□ 5.2 生产发布前必须完成封闭测试 (2023新规)

═══════════════════════════════════════
[DIALOG] 用户确认后 → Play Console → 发布 → 提交审核
审核周期: 通常 1-7 天
═══════════════════════════════════════
```

---

## Phase 13: 归档 (Day 2, 6:30-8:00)

```bash
# [GIT] 最终提交
git tag v1.0.0
git add -A && git commit -m "Release v1.0.0 - Google Play submission ready"

echo "🎉 Android App 开发完成!"
echo "   下一步: Google Play 审核通过后 → 进入运营监控 (operations/App_Operations_SOP.md)"
```

---

## 📚 附录

### A. 命令速查

```bash
./gradlew assembleDebug          # 调试构建
./gradlew assembleRelease        # 发布构建
./gradlew test                   # 单元测试
./gradlew bundleRelease          # App Bundle (推荐)
adb install app-debug.apk        # 安装到设备
adb logcat | grep "MyApp"        # 过滤日志
```

### B. iOS/鸿蒙 → Android 概念映射

| iOS (SwiftUI) | 鸿蒙 (ArkUI) | Android (Compose) |
|---------------|-------------|-------------------|
| `@State` | `@State` | `mutableStateOf` / `StateFlow` |
| `@Binding` | `@Link` | `onValueChange` callback |
| `@ObservedObject` | `@Observed` | `collectAsStateWithLifecycle()` |
| `NavigationStack` | `Navigation` | `NavHost` + `NavController` |
| `List` | `List` | `LazyColumn` |
| `TabView` | `Tabs` | `NavigationBar` + `Scaffold` |
| `CoreData/SwiftData` | `RelationalStore` | `Room` |
| `UserDefaults` | `Preferences` | `DataStore` |
| `URLSession` | `@ohos.net.http` | `Retrofit` / `Ktor` |
| `Xcode` | `DevEco Studio` | `Android Studio` |

---

> **SOP 版本**: 1.0.0 | **最后更新**: 2026-06-08
> **技术栈**: Kotlin + Jetpack Compose + MVVM + Room + Hilt
