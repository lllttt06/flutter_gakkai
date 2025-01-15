---
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.svg')
---

<!-- _class: title -->

# **Flutter × GraphQL**

### **宣言的モバイルアプリ開発**

#### 2025/2/7
#### 吉田 航己

---

## 目次

#### 1. GraphQL の概要
#### 2. ジャンプTOON の GraphQL 活用
#### 3. Flutter で GraphQL を使う利点
#### 4. Flutter × GraphQL 開発の工夫
#### 5. おわりに

---

<!-- header: 1. GraphQL の概要 -->

### GraphQL とは？

**REST の問題を解決するための API クエリ言語およびランタイム**

- データを Node と Edge のグラフで構造化し、必要に応じてそのグラフの一部を利用する
- 単一のエンドポイントから必要なデータだけを Query するため、オーバー(アンダー)フェッチを防げる

---

<!-- header: 2. ジャンプTOON での GraphQL 活用 -->

### 使用パッケージ

GraphQL クライアント：[graphql_flutter](https://pub.dev/packages/graphql_flutter) ([graphql](https://pub.dev/packages/graphql)) 

GraphQL 自動生成関連：[graphql_codegen](https://pub.dev/packages/graphql_codegen)

---

<!-- header: 2. ジャンプTOON での GraphQL 活用 -->

<style scoped>
  pre {
    max-height: 450px;
    overflow-y: auto;
    white-space: pre-wrap;
  }
 .columns {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1rem;
  }
</style>

### SettingScreen の例

データの取得を Screen 内の `useQuery$Setting` で行う


<div class="columns">
<div>

```dart
// setting_screen.dart
class SettingScreen extends HookConsumerWidget {
  const SettingScreen({super.key});
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final isGuestUser = useIsGuestUser();
    final query = useQuery$Setting(options);
    return Scaffold(
      // ...
      body: GraphQLQueryContainer(
        query: query,
        onLoadingWidget: SkeletonSettingScreen(isGuestUser: isGuestUser),
        onErrorWidget: (error, stackTrace) => ErrorContainer(
          error: error,
          stackTrace: stackTrace,
          onAction: query.refetch,
        ),
        child: (data) {
          return SingleChildScrollView(
            // ...
          );
        },
      ),
    );
  }
}
```
</div>
<div>

```dart
// graphql_query_container.dart
class GraphQLQueryContainer extends HookWidget {
  const GraphQLQueryContainer({
    required this.query,
    required this.child,
    this.onLoadingWidget,
    this.onEmptyWidget,
    this.onErrorWidget,
    super.key,
  });

  final QueryHookResult query;
  final Widget Function(TParsed data) child;
  final Widget? onLoadingWidget;
  final Widget? onEmptyWidget;
  final Widget Function(GraphQLException error, StackTrace? stackTrace)?
      onErrorWidget;

  @override
  Widget build(BuildContext context) {
    if (result.hasException && onErrorWidget != null) {
      // エラーが発生した場合
      return onErrorWidget!(
        GraphQLException.fromOperationException(result.exception),
        StackTrace.current,
      );
    } else if (result.data == null && result.isNotLoading) {
      // データがない場合
      return onEmptyWidget ?? const SizedBox();
    } else if (result.data == null && result.isLoading) {
      // ローディング状態
      return onLoadingWidget ?? const SizedBox();
    } else {
      // データが存在する場合
      return child(result.parsedData as TParsed);
    }
  }
}
```
</div>
</div>

---

<style scoped>
  pre > code {
    max-height: 600px;
    overflow-y: auto;
    white-space: pre-wrap;
  }
 .columns {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1rem;
  }
</style>

<!-- header: 2. ジャンプTOON での GraphQL 活用 -->

### query の定義と自動生成ファイル

query を `.graphql` に記述し、`.dart` のコードを自動生成する

<div class="columns">
<div>

```graphql
# setting_screen_query.graphql
query Setting {
  me {
    id
  }
}
```
</div>
<div>

```dart
// setting_screen_query.graphql.dart
class Query$Setting {
  Query$Setting({
    required this.me,
    this.$__typename = 'Query',
  });

  factory Query$Setting.fromJson(Map<String, dynamic> json) {
    final l$me = json['me'];
    final l$$__typename = json['__typename'];
    return Query$Setting(
      me: Query$Setting$me.fromJson((l$me as Map<String, dynamic>)),
      $__typename: (l$$__typename as String),
    );
  }

  final Query$Setting$me me;

  final String $__typename;

  Map<String, dynamic> toJson() {
    final _resultData = <String, dynamic>{};
    final l$me = me;
    _resultData['me'] = l$me.toJson();
    final l$$__typename = $__typename;
    _resultData['__typename'] = l$$__typename;
    return _resultData;
  }

  @override
  int get hashCode {
    // ...
  }

  @override
  bool operator ==(Object other) {
    // ...
  }
}

class Query$Setting$me {
  Query$Setting$me({
    required this.id,
    this.$__typename = 'User',
  });

  factory Query$Setting$me.fromJson(Map<String, dynamic> json) {
    final l$id = json['id'];
    final l$$__typename = json['__typename'];
    return Query$Setting$me(
      id: (l$id as String),
      $__typename: (l$$__typename as String),
    );
  }

  final String id;

  final String $__typename;

  Map<String, dynamic> toJson() {
    final _resultData = <String, dynamic>{};
    final l$id = id;
    _resultData['id'] = l$id;
    final l$$__typename = $__typename;
    _resultData['__typename'] = l$$__typename;
    return _resultData;
  }

  @override
  int get hashCode {
    // ...
  }

  @override
  bool operator ==(Object other) {
    // ...
  }
}

// ...

graphql_flutter.QueryHookResult<Query$Setting> useQuery$Setting(
        [Options$Query$Setting? options]) =>
    graphql_flutter.useQuery(options ?? Options$Query$Setting());

// ...
```
</div>
</div>

---

<!-- header: 2. ジャンプTOON での GraphQL 活用 -->

<style scoped>
  pre > code {
    max-height: 400px;
    overflow-y: auto;
    white-space: pre-wrap;
  }
</style>

### Fragment Colocation
コンポーネントとそこで使用するデータ郡(fragment)を1:1対応させ、近くに配置 ([参考資料](https://speakerdeck.com/quramy/fragment-composition-of-graphql))

```
ui/
├── component/
│   └── text/
│       ├── user_name_text_fragment.graphql
│       ├── user_name_text_fragment.graphql.dart
│       └── user_name_text.dart
└──screen/
    └── root/
        └── my_page/
            ├── my_page_query.graphql
            ├── my_page_query.graphql.dart
            └── my_page_screen.dart
```

---
<!-- header: 3. Flutter × GraphQL の利点 -->

## Flutter × GraphQL の利点

#### 1. スキーマファースト開発
#### 2. クライアントキャッシュ
#### 3. 宣言的 UI との親和性

---

<!-- header: 3. Flutter × GraphQL の利点 -->

## スキーマファースト開発

---

<!-- header: 3. Flutter × GraphQL の利点 -->

## クライアントキャッシュ

---
<!-- header: 3. Flutter × GraphQL の利点 -->

## 宣言的 UI との親和性

---

