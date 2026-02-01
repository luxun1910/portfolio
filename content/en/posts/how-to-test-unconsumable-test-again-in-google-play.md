+++
title = '[Flutter] How to Re-test When You Forgot to Check "Revoke" on Google Play Refund'
date = 2026-02-01T12:27:19+08:00
draft = false
+++

## What Happened During Google Play In-App Purchase Testing

For a solo project, I added an in-app purchase that removes ads.
To verify the flow, I was repeating:

1. Purchase in the app using a test card
2. Refund in Google Play Console with "Revoke" checked

At some point I apparently refunded without checking "Revoke."
As a result, the test account could no longer be used for purchase testing.

## Solution

Even for non-consumable items, calling the Android `consumePurchase` API can reset the purchase state so you can test again.
After checking with AI and Google docs, I added the code below in Flutter, wired it to a button on the settings screen, and used that to fix the situation.

The app uses `flutter_riverpod` for state and `in_app_purchase` for billing.

### Sample Code

`settings_screen`  
This is the settings screen where the button was added.

```dart

// Widget definition
                  if (!kIsWeb && Platform.isAndroid && kDebugMode) ...[
                    const SizedBox(height: 12),
                    _buildForceConsumeButton(purchaseState),
                  ],

// ... (omitted)

  Widget _buildForceConsumeButton(PurchaseState purchaseState) {
    return Container(
      decoration: BoxDecoration(
        color: Theme.of(context).colorScheme.surface,
        borderRadius: BorderRadius.circular(12),
      ),
      child: ListTile(
        title: const Text('For testing: Consume ad removal'),
        subtitle: Text(
          'Resets Play Store purchase state to unpurchased (Android only)',
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
                ? 'Ad removal consumed. You can run purchase tests again.'
                : 'Consume failed. (Nothing to consume if not purchased)',
          ),
        ),
      );
    }
```

`purchase_provider.dart`  
This is the purchase provider. Tapping the button on the settings screen calls the method below.

```dart
  /// Force-consumes ad removal (Android only). For re-running purchase tests. Returns true on success.
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
This is the purchase service. The provider method above calls the method below.

```dart
  /// Force-consumes ad removal (Android only). For re-running Google Play purchase tests.
  /// Consuming a non-consumable also resets Play Store to "unpurchased."
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

With this in place, the entitlement is removed without going through a refund in Google Play Console, so you can effectively "refund" from the built app without opening the console each time.
It was an unexpected side effect, but I kept it and only show this button in debug builds (as in the code above).
