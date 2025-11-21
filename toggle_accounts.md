**toggle accounts** from the main menu:

1. edit /home/b/StudioProjects/Conversations/src/main/res/values/strings.xml:

```diff
<?xml version="1.0" encoding="utf-8"?>
<resources>
+    <string name="pref_show_account_toggles_in_menu_title">Accounts toggle in menu</string>
+    <string name="pref_show_account_toggles_in_menu_summary">Show a toggle for each account in the main menu to quickly enable or disable them</string>
    <string name="action_settings">Settings</string>
```

2. edit /home/b/StudioProjects/Conversations/src/main/res/xml/preferences_interface.xml:

```diff
<?xml version="1.0" encoding="utf-8"?>
<PreferenceScreen xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <PreferenceCategory android:title="@string/appearance">
        <SwitchPreferenceCompat
            android:icon="@drawable/ic_palette_24dp"
            android:key="dynamic_colors"
            android:summary="@string/pref_dynamic_colors_summary"
            android:title="@string/pref_dynamic_colors" />
... some text here ...
+        <SwitchPreferenceCompat
+      			android:key="show_account_toggles_in_menu"
+      			android:title="@string/pref_show_account_toggles_in_menu_title"
+      			android:summary="@string/pref_show_account_toggles_in_menu_summary"
+      			android:defaultValue="true" />

    </PreferenceCategory>
    <PreferenceCategory android:title="@string/pref_category_operating_system">
        <SwitchPreferenceCompat
            android:defaultValue="@bool/allow_screenshots

```



3. edit /home/b/StudioProjects/Conversations/src/main/res/menu/activity_conversations.xml

```diff
    <item
        android:id="@+id/action_scan_qr_code"
        android:icon="@drawable/ic_qr_code_scanner_24dp"
        android:orderInCategory="10"
        android:title="@string/scan_qr_code"
        android:visible="@bool/show_qr_code_scan"
        app:showAsAction="always" />
+    <group
+        android:id="@+id/menu_accounts_group"
+        android:checkableBehavior="all" />
</menu>
```

4. edit /home/b/StudioProjects/Conversations/src/main/java/eu/siacs/conversations/ui/ConversationsActivity.java

using menu, import account classes needed from eu siacs conv.. after import
import SharedPreferences PreferenceManager (pref manager, use AndroidX)


```diff
+    //old code
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.activity_conversations, menu);
        final MenuItem qrCodeScanMenuItem = menu.findItem(R.id.action_scan_qr_code);
        if (qrCodeScanMenuItem != null) {
            if (isCameraFeatureAvailable()) {
                final var fragment =
                        getSupportFragmentManager().findFragmentById(R.id.main_fragment);
                boolean visible =
                        getResources().getBoolean(R.bool.show_qr_code_scan)
                                && fragment instanceof ConversationsOverviewFragment;
                qrCodeScanMenuItem.setVisible(visible);
            } else {
                qrCodeScanMenuItem.setVisible(false);
            }
        }

+        // Added account toggles dynamically
+        SharedPreferences prefs = PreferenceManager.getDefaultSharedPreferences(this);
+        boolean showAccountToggles = prefs.getBoolean("show_account_toggles_in_menu", true);

+        if (showAccountToggles && xmppConnectionService != null) {
+            final Fragment fragment = getSupportFragmentManager().findFragmentById(R.id.main_fragment);
+            if (fragment instanceof ConversationsOverviewFragment) {
+                List<Account> accounts = xmppConnectionService.getAccounts();
+                if (accounts != null && !accounts.isEmpty()) {
+                    for (Account account : accounts) {
+                        MenuItem item = menu.add(
+                                R.id.menu_accounts_group,
+                                Menu.NONE,
+                                Menu.NONE,
+                                account.getJid().asBareJid().toString()
+                        );
+                        item.setCheckable(true);
+                        item.setChecked(account.isEnabled());
+                        item.setOnMenuItemClickListener(i -> {
+                            boolean newState = !account.isEnabled();
+                            account.setOption(Account.OPTION_DISABLED, !newState);
+                            xmppConnectionService.updateAccount(account);
+                            i.setChecked(newState);
+                            return true;
+                        });
+                    }
+                }
+            }
+        }

+//old code continues

        return super.onCreateOptionsMenu(menu);
    }

    @Override
    public void onConversationSelected(Conversation conversation) {
        clearPendingViewIntent();
```
