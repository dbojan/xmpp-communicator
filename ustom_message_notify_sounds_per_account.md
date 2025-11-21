1.edit /home/b/StudioProjects/Conversations/src/main/res/values/strings.xml:

```diff
<?xml version="1.0" encoding="utf-8"?>
<resources>
+    <string name="pref_use_custom_sounds_for_notify_message_old_title">Custom message sounds</string>
+    <string name="pref_use_custom_sounds_for_notify_message_old_summary">Use custom message notification sounds, based on accounts, for older Android versions. (Notifications/message1.ogg for first account, message2.ogg for second account ...)</string>
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
+      			android:key="use_custom_sounds_for_notify_message_old"
+      			android:title="@string/pref_use_custom_sounds_for_notify_message_old_title"
+      			android:summary="@string/pref_use_custom_sounds_for_notify_message_old_summary"
+      			android:defaultValue="true"/>
```

3. edit /home/b/StudioProjects/Conversations/src/main/java/eu/siacs/conversations/services/NotificationService.java

```diff
+    //added for custom notify per account
+    private String getChannelIdForAccount(Account account) {
+        int accountIndex = mXmppConnectionService.getAccounts().indexOf(account) + 1;
+        return "messages_channel_" + accountIndex;
+    }

+    //added for custom notify per account
+    private void createAccountNotificationChannel(Account account) {
+        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {

+            int accountIndex = mXmppConnectionService.getAccounts().indexOf(account) + 1;
+            String channelId = getChannelIdForAccount(account);
+            String channelName = "Messages for Account " + accountIndex;

+            // Custom sound URI
+            String soundFileName = "message" + accountIndex + ".ogg";
+            File soundFile = new File(Environment.getExternalStorageDirectory(), "Notifications/" + soundFileName);

+            //  Uri soundUri = soundFile.exists() ? Uri.fromFile(soundFile) : RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);

+            Uri soundUri = null;
+            boolean isCustomSoundFound = false;

+            if (soundFile.exists()) {
+                soundUri = Uri.fromFile(soundFile);
+                isCustomSoundFound = true;
+            }

+            if (soundUri == null) {
+                soundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
+            }

+            AudioAttributes audioAttributes = new AudioAttributes.Builder()
+                    .setUsage(AudioAttributes.USAGE_NOTIFICATION)
+                    .setContentType(AudioAttributes.CONTENT_TYPE_SONIFICATION)
+                    .build();

+            NotificationManager manager = mXmppConnectionService.getSystemService(NotificationManager.class);
+            NotificationChannel existingChannel = manager.getNotificationChannel(channelId);

+            // Only delete the existing channel if a custom sound was found
+            // This ensures a clean slate when using a custom sound.
+            // If we are falling back to default, we rely on the existing channel
+            // being created later, or we let the NotificationManager handle it.
+            if (isCustomSoundFound && existingChannel != null) {
+                // Delete existing channel to recreate it with the *new* custom sound
+                manager.deleteNotificationChannel(channelId);
+                existingChannel = null; // Mark as deleted for creation logic below
+            }

+            // If we are falling back to default, we SKIP the deletion/recreation here
+            // and rely on the existing channel's configuration to use the system default,
+            // OR we create the channel with the default sound if it doesn't exist.

+            if (existingChannel == null || isCustomSoundFound) {
+                // Create a NEW channel if it doesn't exist OR if we found a custom sound
+                // NOTE: If existingChannel is NOT null and we are using the default sound,
+                // we do nothing, trusting the existing channel uses the default sound.

+                NotificationChannel channel = new NotificationChannel(
+                        channelId,
+                        channelName,
+                        NotificationManager.IMPORTANCE_DEFAULT
+                );

+                // Set the sound based on what was found (custom or default)
+                final Uri finalSoundUri = soundUri; // Create final variable for setSound call
+                channel.setSound(finalSoundUri, audioAttributes);
+                channel.enableVibration(true);

+                manager.createNotificationChannel(channel);

+            }
+        }
+    }

+//old code

    private Builder buildSingleConversations(
        final ArrayList<Message> messages, final boolean notify) {

+        // old code, commented out

+//        final var channel = notify ? MESSAGES_NOTIFICATION_CHANNEL : "silent_messages";
+//        final Builder notificationBuilder =
+//                new NotificationCompat.Builder(mXmppConnectionService, channel);
+//

+//        if (messages.isEmpty()) {
+//            return notificationBuilder;
+//        }
+//        final Conversation conversation = (Conversation) messages.get(0).getConversation();


+        //added for custom notify per account

+        if (messages.isEmpty()) {
+            final Builder notificationBuilder = new NotificationCompat.Builder(mXmppConnectionService, "silent_messages");
+            return notificationBuilder;
+        }

+        final Conversation conversation = (Conversation) messages.get(0).getConversation();
+        final Account convAccount = conversation.getAccount();

+        // 1. Get SharedPreferences
+        SharedPreferences prefs = PreferenceManager.getDefaultSharedPreferences(mXmppConnectionService);

+        // 2. Check the custom sound preference.
+        // The default value 'true' is used if the user hasn't touched the setting,
+        // which matches your XML: android:defaultValue="true"
+        final boolean useCustomSounds = prefs.getBoolean("use_custom_sounds_for_notify_message_old", true);

+        String channel;

+        if (notify && useCustomSounds) {

+            // Check if the custom file EXISTS
+            int accountIndex = mXmppConnectionService.getAccounts().indexOf(convAccount) + 1;
+            String soundFileName = "message" + accountIndex + ".ogg";
+            File soundFile = new File(Environment.getExternalStorageDirectory(), "Notifications/" + soundFileName);

+            if (soundFile.exists()) {
+                // CASE A: Custom file found. Use the custom channel.
+                createAccountNotificationChannel(convAccount);
+                channel = getChannelIdForAccount(convAccount);
+            } else {
+                // CASE B: Custom file NOT found. Fall back to the GENERIC default channel.
+                // DO NOT call createAccountNotificationChannel() here.
+                // This prevents the account channel from getting stuck on silent.
+                channel = "messages"; // Use the channel ID guaranteed to have default sound.
+            }

+        } else {
+            // CASE C: notify is false OR custom sounds are disabled.
+            // Use the original 'messages' or 'silent_messages' channel.
+            channel = notify ? "messages" : "silent_messages";
+        }

+        final Builder notificationBuilder = new NotificationCompat.Builder(mXmppConnectionService, channel);

        //old code, continue

        notificationBuilder.setLargeIcon(
                mXmppConnectionService
                        .getAvatarService()
                        .get(
                                conversation,
                                AvatarService.getSystemUiAvatarSize(mXmppConnectionService)));
        notificationBuilder.setContentTitle(conversation.getName());
        if (Config.HIDE_MESSAGE_TEXT_IN_NOTIFICATION) {
            int count = messages.size();
            notificationBuilder.setContentText(
                    mXmppConnectionService
                            .getResources()
                            .getQuantityString(R.plurals.x_messages, count, count));
        } else {

```








