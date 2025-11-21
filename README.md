# xmpp-communicator
fork of conversations xmpp app

-download android studio  
-open https://codeberg.org/iNPUTmice/Conversations in android studio.  
-edit some files, and send app to device, by pressing play in android studio.  

-since github is brainded when it comes to adding colors to text, remove + in front of green (added) lines. sorry.  

[add_colors.md](add_colors.md)

[How to use **different colors** in chat, based on accounts:  

1. add green lines, remove + at start, to /home/b/StudioProjects/Conversations/src/main/res/values/strings.xml:

```diff
<?xml version="1.0" encoding="utf-8"?>
<resources>
+    <string name="pref_use_chat_colors_based_on_accounts_title">Accounts based chat colors</string>
+    <string name="pref_use_chat_colors_based_on_accounts_summary">Every account uses different color for chats</string>
    <string name="action_settings">Settings</string>
```

2. add to add to /home/b/StudioProjects/Conversations/src/main/res/xml/preferences_interface.xml:

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
+        // Detect if system is in dark theme
+        int nightModeFlags;
+        nightModeFlags = activity.getResources().getConfiguration().uiMode & Configuration.UI_MODE_NIGHT_MASK;
+        boolean isDarkMode = (nightModeFlags == Configuration.UI_MODE_NIGHT_YES);

+        // Light theme pastel palette
+        final int[] lightColors = {
+                0xFFFFF9C4, // light yellow
+                0xFFE1F5FE, // light blue
+                0xFFE8F5E9, // light green
+                0xFFF3E5F5, // light purple
+                0xFFFFEBEE, // light red
+                0xFFFFF3E0, // light orange
+                0xFFE0F7FA, // light cyan
+                0xFFF1F8E9, // pale lime
+                0xFFFBE9E7, // light peach
+                0xFFEDE7F6  // lavender
+        };

+        // Dark theme palette (muted desaturated tones)
+        final int[] darkColors = {
+                0xFF616161, // medium gray
+                0xFF455A64, // blue gray
+                0xFF37474F, // dark teal gray
+                0xFF4E342E, // dark brown
+                0xFF5D4037, // warm brown
+                0xFF283593, // indigo
+                0xFF00695C, // teal
+                0xFF2E7D32, // dark green
+                0xFF6A1B9A, // purple
+                0xFFAD1457  // magenta
+       };

+        int index = Math.abs(account.getJid().asBareJid().toString().hashCode())
+                % lightColors.length;

+        return isDarkMode ? darkColors[index] : lightColors[index];
+    }

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



```]()

How to use **different colors** in chat, based on accounts:  

1. add green lines, remove + at start, to /home/b/StudioProjects/Conversations/src/main/res/values/strings.xml:

```diff
<?xml version="1.0" encoding="utf-8"?>
<resources>
+    <string name="pref_use_chat_colors_based_on_accounts_title">Accounts based chat colors</string>
+    <string name="pref_use_chat_colors_based_on_accounts_summary">Every account uses different color for chats</string>
    <string name="action_settings">Settings</string>
```

2. add to add to /home/b/StudioProjects/Conversations/src/main/res/xml/preferences_interface.xml:

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
+        // Detect if system is in dark theme
+        int nightModeFlags;
+        nightModeFlags = activity.getResources().getConfiguration().uiMode & Configuration.UI_MODE_NIGHT_MASK;
+        boolean isDarkMode = (nightModeFlags == Configuration.UI_MODE_NIGHT_YES);

+        // Light theme pastel palette
+        final int[] lightColors = {
+                0xFFFFF9C4, // light yellow
+                0xFFE1F5FE, // light blue
+                0xFFE8F5E9, // light green
+                0xFFF3E5F5, // light purple
+                0xFFFFEBEE, // light red
+                0xFFFFF3E0, // light orange
+                0xFFE0F7FA, // light cyan
+                0xFFF1F8E9, // pale lime
+                0xFFFBE9E7, // light peach
+                0xFFEDE7F6  // lavender
+        };

+        // Dark theme palette (muted desaturated tones)
+        final int[] darkColors = {
+                0xFF616161, // medium gray
+                0xFF455A64, // blue gray
+                0xFF37474F, // dark teal gray
+                0xFF4E342E, // dark brown
+                0xFF5D4037, // warm brown
+                0xFF283593, // indigo
+                0xFF00695C, // teal
+                0xFF2E7D32, // dark green
+                0xFF6A1B9A, // purple
+                0xFFAD1457  // magenta
+       };

+        int index = Math.abs(account.getJid().asBareJid().toString().hashCode())
+                % lightColors.length;

+        return isDarkMode ? darkColors[index] : lightColors[index];
+    }

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



