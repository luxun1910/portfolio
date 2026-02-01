+++
title = '【Flutter】一次性商品退款时忘记勾选「删除授权」的解决方法【Google Play】'
date = 2026-02-01T12:27:19+08:00
draft = false
+++

## Google Play 内购商品测试时发生的问题

个人开发中，做了一个用于去除应用内广告的内购商品。为了做功能测试，我反复进行：

1. 在应用内用测试卡购买
2. 在 Google Play Console 中勾选「删除授权」后退款

但似乎有一次忘记勾选「删除授权」就进行了退款。结果，使用的测试账号无法再进行购买测试了。

## 解决方法

即使是非消耗型商品，用 Android 的 consumePurchase 方法处理购买后，也可以恢复为未购买状态。问了 AI 和 Google 后，在 Flutter 里写了如下代码，绑定到设置页的按钮上，点击后问题就解决了。

前提是，使用 `flutter_riverpod` 做状态管理，使用 `in_app_purchase` 处理内购。

### 示例代码

`settings_screen`
这是设置页，这次在这里放了一个按钮。
```dart

// 组件描述
                  if (!kIsWeb && Platform.isAndroid && kDebugMode) ...[
                    const SizedBox(height: 12),
                    _buildForceConsumeButton(purchaseState),
                  ],

// 中间省略

  Widget _buildForceConsumeButton(PurchaseState purchaseState) {
    return Container(
      decoration: BoxDecoration(
        color: Theme.of(context).colorScheme.surface,
        borderRadius: BorderRadius.circular(12),
      ),
      child: ListTile(
        title: const Text('购买测试用：消耗去广告'),
        subtitle: Text(
          '将 Play 商店的购买状态恢复为未购买（仅 Android）',
          style: Theme.of(context).textTheme.bodySmall?.copyWith(
                color: Theme.of(context).colorScheme.onSurfaceVariant,
              ),
        ),
        trailing: purchaseState.isLoading
            ? const SizedBox(
                width: 24,
                height: 24,
                child: CircularProgressIndicator(strokeWidth: 2),
              )
            : IconButton(
                icon: const Icon(Icons.refresh),
                onPressed: () => _forceConsumeRemoveAds(),
              ),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(12),
        ),
      ),
    );
  }

  Future<void> _forceConsumeRemoveAds() async {
    final success =
        await ref.read(purchaseProvider.notifier).forceConsumeRemoveAds();
    if (mounted) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text(
            success
                ? '已消耗去广告。可以再次进行购买测试。'
                : '消耗失败。（未购买时无此项目）',
          ),
        ),
      );
    }
```

`purchase_provider.dart`
这是购买逻辑的 Provider。点击设置页的按钮后，会调用下面这个方法。
```dart
  /// 强制消耗去广告（仅 Android）。用于再次进行购买测试。成功返回 true。
  Future<bool> forceConsumeRemoveAds() async {
    state = state.copyWith(isLoading: true, error: null);
    try {
      final ok = await _purchaseService.forceConsumeRemoveAds();
      if (ok) {
        _settingsNotifier.setAdsRemoved(false);
        state = state.copyWith(isPurchased: false);
      }
      state = state.copyWith(isLoading: false);
      return ok;
    } catch (e) {
      state = state.copyWith(error: e.toString(), isLoading: false);
      return false;
    }
  }
```
　
`purchase_service.dart`
这是购买逻辑的 Service。由上面的 Provider 调用下面的方法。
```dart
  /// 强制消耗去广告（仅 Android）。用于再次进行 Google Play 购买测试。
  /// 非消耗型商品也可以通过 consume 将 Play 商店侧恢复为「未购买」。
  Future<bool> forceConsumeRemoveAds() async {
    final androidAddition =
        _inAppPurchase.getPlatformAddition<InAppPurchaseAndroidPlatformAddition>();
    if (androidAddition == null) return false;

    final QueryPurchaseDetailsResponse response =
        await androidAddition.queryPastPurchases();
    if (response.error != null) return false;

    final removeAdsList = response.pastPurchases
        .where((d) => d.productID == removeAdsProductId)
        .toList();
    if (removeAdsList.isEmpty) return false;

    final result =
        await androidAddition.consumePurchase(removeAdsList.first);
    return result.responseCode == BillingResponse.ok;
  }
```

这样处理后，不需要在 Google Play Console 里操作退款，购买资格也会被清除；不用每次打开 Google Play Console，在已安装的应用里就能实现类似退款的效果。虽然是顺带得到的做法，但既然有用，就在调试构建时显示上述按钮（上面代码里也已经按此设置）。
