How to use **different colors** in chat, based on accounts:  

1. add green lines, remove + at start, to /home/b/StudioProjects/Conversations/src/main/res/values/strings.xml:

```diff
<?xml version="1.0" encoding="utf-8"?>
<resources>
+    <string name="pref_use_chat_colors_based_on_accounts_title">Accounts based chat colors</string>
+    <string name="pref_use_chat_colors_based_on_accounts_summary">Every account uses different color for chats</string>
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
+		<SwitchPreferenceCompat
+			android:key="use_chat_colors_based_on_accounts"
+			android:title="@string/pref_use_chat_colors_based_on_accounts_title"
+			android:summary="@string/pref_use_chat_colors_based_on_accounts_summary"
+			android:defaultValue="true"/>

    </PreferenceCategory>
    <PreferenceCategory android:title="@string/pref_category_operating_system">
        <SwitchPreferenceCompat
            android:defaultValue="@bool/allow_screenshots"
            android:icon="@drawable/ic_screenshot_24dp"
```

3. add to in /home/b/StudioProjects/Conversations/src/main/java/eu/siacs/conversations/ui/adapter/ConversationAdapter.java:

```diff

   @NonNull
    @Override
    public ConversationViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        return new ConversationViewHolder(
                DataBindingUtil.inflate(
                        LayoutInflater.from(parent.getContext()),
                        R.layout.item_conversation,
                        parent,
                        false));
    }

+    //added for diff colors
+    private boolean useAccountColors() {
+        return PreferenceManager
+                .getDefaultSharedPreferences(activity)
+                .getBoolean("use_chat_colors_based_on_accounts", true); // default true
+    }

+    //added  for diff colors
+    private int getColorForAccount(Account account) {
+        // 1. Get Index
+        int accountIndex = 0;
+        if (activity != null && activity.xmppConnectionService != null) {
+            accountIndex = activity.xmppConnectionService.getAccounts().indexOf(account);
+            if (accountIndex < 0) accountIndex = 0;
+        }

+        // 2. Determine Hue (Color)
+        // We multiply by a "Golden Angle" (approx 137.5 degrees) to ensure
+        // colors are visually distinct and don't repeat quickly.
+        float goldenAngle = 137.508f;
+        float hueOffset = 240f; // Start at Blue (240 degrees) instead of Red (0 degrees)

+        // Applying the offset to shift the entire palette
+        float hue = ( (accountIndex * goldenAngle) + hueOffset ) % 360;

+        // Simplified Logic for Dark Mode Value (V) Adjustment
+        // Base Value is 0.65. Range of adjustment is +/- 0.05.
+        float vBase = 0.65f;
+        float vRange = 0.05f;

+        // 3. Determine Saturation and Value (Brightness)
+        // Adjust these based on Dark Mode
+        int nightModeFlags = activity.getResources().getConfiguration().uiMode & Configuration.UI_MODE_NIGHT_MASK;
+        boolean isDarkMode = (nightModeFlags == Configuration.UI_MODE_NIGHT_YES);

+        float saturation;
+        float value;

+        if (isDarkMode) {
+            // Dark Theme: Lower saturation (pastel/muted), Lower brightness
+            saturation = 0.6f;
+            value = 0.65f; //7
+        } else {
+            // Light Theme: Low saturation (pastel), High brightness
+            saturation = 0.4f;//0.25
+            value = 0.9f;//1
+        }

+        // 4. Generate Color
+        return android.graphics.Color.HSVToColor(new float[]{hue, saturation, value});
+    }

+    //old code
    @Override
    public void onBindViewHolder(@NonNull ConversationViewHolder viewHolder, int position) {
        Conversation conversation = conversations.get(position);
        if (conversation == null) {
            return;
        }
        CharSequence name = conversation.getName();
        if (name instanceof Jid) {
            viewHolder.binding.conversationName.setText(
                    IrregularUnicodeDetector.style(activity, (Jid) name));
        } else {
            viewHolder.binding.conversationName.setText(name);
        }

+//old code
        if (conversation == ConversationFragment.getConversation(activity)) {
            // Highlight selected conversation
            viewHolder.binding.frame.setBackgroundResource(
                    R.drawable.background_selected_item_conversation);
+            //old code, commented out
+            // // viewHolder.binding.frame.setBackgroundColor(MaterialColors.getColor(viewHolder.binding.frame, com.google.android.material.R.attr.colorSurfaceDim));
        } else {
+            //added new code
+            // Color conversations by account
+            final Account account = conversation.getAccount();
+            if (useAccountColors() && account != null) {
+                int color = getColorForAccount(account);
+                viewHolder.binding.frame.setBackgroundColor(color);
+            } else {
+                // Fallback to theme surface color
+                //old code
                viewHolder.binding.frame.setBackgroundColor(
                        MaterialColors.getColor(
                                viewHolder.binding.frame,
                                com.google.android.material.R.attr.colorSurface));
            }
        }



```
