+++
title = '【Flutter】1回限りのアイテム購入を払い戻す際、「資格情報を削除」し忘れた場合の対処法【Google Play】'
date = 2026-02-01T12:27:19+08:00
draft = false
+++

## Google Playの課金アイテムの動作確認時、事件は起きた

個人開発で、アプリ内の広告を削除できる課金アイテムを用意した。
動作確認のため、

1. アプリ内でテストカードを使って購入
2. Google Play Consoleで「資格情報を削除」にチェックを入れて払い戻し

を繰り返していた。

しかし、どこかのタイミングで「資格情報を削除」にチェックを入れ忘れて払い戻してしまったらしい。
その結果、使用していたテストアカウントで購入テストができなくなってしまった。

## 解決方法

非消費アイテムであっても、Android用のconsumePurchaseメソッドで購入処理をすると、未購入の状態に戻すことができるらしい。
AIやGoogleに聞いた結果、Flutter内に以下のようなコードを書き、設定画面に配置したボタンに紐づけてから押下することで解決した。

前提として、状態管理のために`flutter_riverpod`と、課金処理のために`in_app_purchase`を使用している。

### サンプルコード

`settings_screen`
こちらは設定画面で、今回ここにボタンを配置した。
```dart

// ウィジェット記述
                  if (!kIsWeb && Platform.isAndroid && kDebugMode) ...[
                    const SizedBox(height: 12),
                    _buildForceConsumeButton(purchaseState),
                  ],

// 中略

  Widget _buildForceConsumeButton(PurchaseState purchaseState) {
    return Container(
      decoration: BoxDecoration(
        color: Theme.of(context).colorScheme.surface,
        borderRadius: BorderRadius.circular(12),
      ),
      child: ListTile(
        title: const Text('購入テスト用：広告削除を消費'),
        subtitle: Text(
          'Play ストアの購入状態を未購入に戻します（Android のみ）',
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
                ? '広告削除を消費しました。再度購入テストができます。'
                : '消費に失敗しました。（未購入の場合は対象がありません）',
          ),
        ),
      );
    }
```

`purchase_provider.dart`
こちらは購入処理プロバイダー。
先ほどの画面のボタンが押下されると、購入処理プロバイダー内の以下のメソッドが呼ばれる。
```dart
  /// 広告削除を強制消費（Android のみ）。購入テスト再実行用。成功で true。
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
こちらは購入処理サービス。
上記の購入処理プロバイダーのメソッドから、以下のメソッドが呼び出される。
```dart
  /// 広告削除を強制消費（Android のみ）。Google Play 購入テストの再実行用。
  /// 非消耗型でも consume することで Play ストア側を「未購入」に戻せる。
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

この処理をすると、Google Play Consoleから払い戻しをしなくても利用資格が削除されるようで、いちいちGoogle Play Consoleを開かなくても、ビルドしたアプリ上から払い戻し（のような挙動）が実現できる。
思わぬ副産物ではあったが、せっかくなので、デバッグビルド時は上記ボタンが出現するようにした（上記コードもその設定になっている）。
