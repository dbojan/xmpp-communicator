install text to speech on you android phone, if you have more than one, set one as default in settings

use google one, or free one: SherpaTTS (say gb-Alba), or RHVoice

look for:   
//added tts   
//end  

in modified files:

Conversations/src/main/res/xml/preferences_interface.xml  
Conversations/src/main/java/eu/siacs/conversations/services/XmppConnectionService.java  
Conversations/src/main/java/eu/siacs/conversations/services/NotificationService.java  

also modified, cause I could not installed to my device (newer version already installed):  

Conversations/build gradle:  
defaultConfig {  
  versionCode 42160  
 versionName "2.19.6"  
