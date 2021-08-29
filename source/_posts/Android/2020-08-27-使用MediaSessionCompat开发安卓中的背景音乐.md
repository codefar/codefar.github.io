---
title: 使用MediaSessionCompat开发安卓中的背景音乐
categories:
  - Android
comments: true
tags:
  - Android
  - MediaSessionCompat
description: node
date: 2021-08-27 10:36:10
---


移动设备最受欢迎的用途之一是通过音乐流媒体服务播放音乐，下载音乐或其他音频源。  虽然这个功能很常见，但是很难实现，因为需要正确构建许多不同的部分以便为用户提供完整的Android体验。 

在本教程中，您将了解如何使用Android支持库中的MediaSessionCompat为用户创建合适的后台音频服务。

# 设置
需要做的第一件事是将Android支持库引入项目。 将以下行添加到module级的build.gradle文件中。

```
compile 'com.android.support:support-v13:24.2.1'
```

同步项目之后，创建一个新的Java类。 对于这个例子我将调用类```BackgroundAudioService```。   我们还将实现以下接口：```MediaPlayer.OnCompletionListener和AudioManager.OnAudioFocusChangeListener```。


现在```MediaBrowserServiceCompat```实现已创建，让我们花点时间更新**AndroidManifest.xml**，然后返回此类。  在xml文件中，您需要申请WAKE_LOCK权限。

```
<uses-permission android:name="android.permission.WAKE_LOCK" />
```
接下来，在```application```节点内，使用以下```intent-filter```项目声明新服务。  这些将允许服务拦截设备的控制按钮，耳机事件和媒体浏览，例如Android Auto（尽管我们不会在本教程中对Android Auto做任何事情，但仍然需要```MediaBrowserServiceCompat```的基本支持）。

```
<service android:name=".BackgroundAudioService">
    <intent-filter>
        <action android:name="android.intent.action.MEDIA_BUTTON" />
        <action android:name="android.media.AUDIO_BECOMING_NOISY" />
        <action android:name="android.media.browse.MediaBrowserService" />
    </intent-filter>
</service>
```

最后，需要声明使用```MediaButtonReceiver``` Android支持库。  这将允许您拦截运行在KitKat系统和更早版本设备上的媒体控制按钮交互和耳机事件。

```
<receiver android:name="androidx.media.session.MediaButtonReceiver">
    <intent-filter>
        <action android:name="android.intent.action.MEDIA_BUTTON" />
        <action android:name="android.media.AUDIO_BECOMING_NOISY" />
    </intent-filter>
</receiver>
```

现在***AndroidManifest.xml***文件已经完成，可以关闭它了。  我们还将创建一个名为```MediaStyleHelper```的类，这是由Google开发者倡导者Ian Lake撰写用来清理媒体样式通知的。

```
import android.content.Context;
import android.support.v4.media.MediaDescriptionCompat;
import android.support.v4.media.MediaMetadataCompat;
import android.support.v4.media.session.MediaControllerCompat;
import android.support.v4.media.session.MediaSessionCompat;
import android.support.v4.media.session.PlaybackStateCompat;

import androidx.core.app.NotificationCompat;
import androidx.media.session.MediaButtonReceiver;

public class MediaStyleHelper {
    /**
     * Build a notification using the information from the given media session. Makes heavy use
     * of {@link MediaMetadataCompat#getDescription()} to extract the appropriate information.
     * @param context Context used to construct the notification.
     * @param mediaSession Media session to get information.
     * @return A pre-built notification with information from the given media session.
     */
    public static NotificationCompat.Builder from(
            Context context, MediaSessionCompat mediaSession) {
        MediaControllerCompat controller = mediaSession.getController();
        MediaMetadataCompat mediaMetadata = controller.getMetadata();
        MediaDescriptionCompat description = mediaMetadata.getDescription();
 
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context);
        builder
                .setContentTitle(description.getTitle())
                .setContentText(description.getSubtitle())
                .setSubText(description.getDescription())
                .setLargeIcon(description.getIconBitmap())
                .setContentIntent(controller.getSessionActivity())
                .setDeleteIntent(
                        MediaButtonReceiver.buildMediaButtonPendingIntent(context, PlaybackStateCompat.ACTION_STOP))
                .setVisibility(NotificationCompat.VISIBILITY_PUBLIC);
        return builder;
    }
}
```
一旦创建，请关闭该文件。  下一节将介绍背景音频服务。

# 构建背景音频服务

现在是时候介绍创建媒体应用程序的核心了。  有几个成员变量需要首先声明：```MediaPlayer``` 用于实际播放以及```MediaSessionCompat```用于管理元数据和播放控件状态的对象。

```
private MediaPlayer mMediaPlayer;
private MediaSessionCompat mMediaSessionCompat;
```

此外，还需要```BroadcastReceiver```来监听耳机状态的更改。  为了简单起见，该接收者将暂停MediaPlayer播放。
```
private BroadcastReceiver mNoisyReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        if( mMediaPlayer != null && mMediaPlayer.isPlaying() ) {
            mMediaPlayer.pause();
        }
    }
};
```
对于final类型的成员变量，需要创建  ```MediaSessionCompat.Callback```对象，用于在媒体会话变化时处理播放状态

```
private MediaSessionCompat.Callback mMediaSessionCallback = new MediaSessionCompat.Callback() {
     
    @Override
    public void onPlay() {
        super.onPlay();
    }
 
    @Override
    public void onPause() {
        super.onPause();
    }
 
    @Override
    public void onPlayFromMediaId(String mediaId, Bundle extras) {
        super.onPlayFromMediaId(mediaId, extras);
    }
};
```

我们将在本教程的后面重新阅读上述方法，因为它们将用于驱动我们的媒体应用程序。

我们还需要声明两个方法，虽然它们不需要为本教程做任何事情：```onGetRoot()```和```nLoadChildren()```。  可以默认使用以下代码。
```
@Nullable
@Override
public BrowserRoot onGetRoot(@NonNull String clientPackageName, int clientUid, @Nullable Bundle rootHints) {
    if(TextUtils.equals(clientPackageName, getPackageName())) {
        return new BrowserRoot(getString(R.string.app_name), null);
    }
 
    return null;
}
 
//Not important for general audio service, required for class
@Override
public void onLoadChildren(@NonNull String parentId, @NonNull Result<List<MediaBrowserCompat.MediaItem>> result) {
    result.sendResult(null);
}

```

最后，需要重写```onStartCommand()```方法，这是```Service```的入口。  该方法将使用传递给```Service```并将其发送到```MediaButtonReceiver```类的```Intent``` 。
```
@Override
public int onStartCommand(Intent intent, int flags, int startId) {
    MediaButtonReceiver.handleIntent(mMediaSessionCompat, intent);
    return super.onStartCommand(intent, flags, startId);
}
```

# 初始化
现在，基本成员变量已经创建了，是时候初始化所有内容了。  我们将调用```onCreate()```里面的各种辅助方法来完成初始化。

```
@Override
public void onCreate() {
    super.onCreate();
 
    initMediaPlayer();
    initMediaSession();
    initNoisyReceiver();
}
```

第一个方法```initMediaPlayer()```将初始化我们在类顶部创建的**MediaPlayer**对象，请求部分唤醒锁（这就是为什么我们需要在AndroidManifest.xml中需要相关权限），并设置播放器的音量。
```
private void initMediaPlayer() {
    mMediaPlayer = new MediaPlayer();
    mMediaPlayer.setWakeMode(getApplicationContext(), PowerManager.PARTIAL_WAKE_LOCK);
    mMediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
    mMediaPlayer.setVolume(1.0f, 1.0f);
}
```
第二个方法```initMediaSession(```)是初始化```MediaSessionCompat```对象并将其连接到允许我们处理播放和用户输入的媒体按钮和控制方法。  该方法首先创建一个指向Android支持库的```MediaButtonReceiver```类的```ComponentName```对象，并使用它来创建一个新的```MediaSessionCompat```。  然后我们将之前创建的```MediaSession.Callback```对象传递给它，并设置接收媒体按钮输入和控制信号所需的标志。  接下来，我们创建一个新的```Intent```用于处理在```Lollipo```设备之前的媒体按钮输入，并为我们的服务设置媒体会话令牌。
```
private void initMediaSession() {
    ComponentName mediaButtonReceiver = new ComponentName(getApplicationContext(), MediaButtonReceiver.class);
    mMediaSessionCompat = new MediaSessionCompat(getApplicationContext(), "Tag", mediaButtonReceiver, null);
 
    mMediaSessionCompat.setCallback(mMediaSessionCallback);
    mMediaSessionCompat.setFlags( MediaSessionCompat.FLAG_HANDLES_MEDIA_BUTTONS | MediaSessionCompat.FLAG_HANDLES_TRANSPORT_CONTROLS );
 
    Intent mediaButtonIntent = new Intent(Intent.ACTION_MEDIA_BUTTON);
    mediaButtonIntent.setClass(this, MediaButtonReceiver.class);
    PendingIntent pendingIntent = PendingIntent.getBroadcast(this, 0, mediaButtonIntent, 0);
    mMediaSessionCompat.setMediaButtonReceiver(pendingIntent);
 
    setSessionToken(mMediaSessionCompat.getSessionToken());
}
```

最后，注册我们在类顶部创建的```BroadcastReceiver```以便监听耳机更改事件。
```
private void initNoisyReceiver() {
    //Handles headphones coming unplugged. cannot be done through a manifest receiver
    IntentFilter filter = new IntentFilter(AudioManager.ACTION_AUDIO_BECOMING_NOISY);
    registerReceiver(mNoisyReceiver, filter);
}
```
# 处理音频焦点

现在，已经初始化了```BroadcastReceiver```、MediaSessionCompat``````和```MediaPlayer```对象，是时候处理音频的焦点了。 

虽然我们可能会认为自己的音频应用程序是当前最重要的，但是设备上的其他音频应用程序会竞争，例如电子邮件通知或手机游戏。  为了处理这些情况，Android系统使用音频焦点来确定如何处理音频。 

我们要处理的第一种情况是开始播放并尝获取设备焦点。  在```MediaSessionCompat.Callback```对象中，进入```onPlay()```方法并添加以下条件检查。
```
@Override
public void onPlay() {
    super.onPlay();
    if( !successfullyRetrievedAudioFocus() ) {
        return;
    }
}
```
上面的代码将调用一个辅助程序尝试获取焦点，如果失败它将return。  在真正的应用程序中，会更加优雅地处理失败的音频播放。   ```successfullyRetrievedAudioFocus()```方法将获得对系统```AudioManager```的引用，并尝试请求音频焦点用于流式音乐。  然后它将返回一个```boolean```值来表示是否请求成功。
```
private boolean successfullyRetrievedAudioFocus() {
    AudioManager audioManager = (AudioManager) getSystemService(Context.AUDIO_SERVICE);
 
    int result = audioManager.requestAudioFocus(this,
            AudioManager.STREAM_MUSIC, AudioManager.AUDIOFOCUS_GAIN);
     
    return result == AudioManager.AUDIOFOCUS_GAIN;
}
```
你会注意到，我们也在传递this给与服务 *```OnAudioFocusChangeListener```关联的```requestAudioFocus()```方法 。  你需要监听几种不同的状态以便遵循设备应用程序生态系统。

 *```AudioManager.AUDIOFOCUS_LOSS```：当另一个应用程序请求音频焦点时发生这种情况 。  发生这种情况时，您应该停止在应用程序中播放音频。
 *```AudioManager.AUDIOFOCUS_LOSS_TRANSIENT```：当另一个应用程序想要播放音频时进入此状态，但它只需要短时间内需要对焦。  您可以使用此状态来暂停音频播放。
 *```AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK```：当请求音频焦点时，会引发“可以”的状态，这意味着可以继续播放，但应将音量降低一点。  当设备播放通知声音时，可能会发生这种情况。
 *```AudioManager.AUDIOFOCUS_GAIN```我们将讨论的最终状态是```AUDIOFOCUS_GAIN```。  当可以播放的音频播放完成后，应用程序可以恢复到以前的级别。
```onAudioFocusChange()```回调可能如下所示：

```
@Override
public void onAudioFocusChange(int focusChange) {
    switch( focusChange ) {
        case AudioManager.AUDIOFOCUS_LOSS: {
            if( mMediaPlayer.isPlaying() ) {
                mMediaPlayer.stop();
            }
            break;
        }
        case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT: {
            mMediaPlayer.pause();
            break;
        }
        case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK: {
            if( mMediaPlayer != null ) {
                mMediaPlayer.setVolume(0.3f, 0.3f);
            }
            break;
        }
        case AudioManager.AUDIOFOCUS_GAIN: {
            if( mMediaPlayer != null ) {
                if( !mMediaPlayer.isPlaying() ) {
                    mMediaPlayer.start();
                }
                mMediaPlayer.setVolume(1.0f, 1.0f);
            }
            break;
        }
    }
}
```

# 理解MediaSessionCompat.Callback

现在有了```Service的基本结构```，是时候研究```MediaSessionCompat.Callback```了。  在最后一节，给```onPlay()```添加代码以检查音频焦点是否被授予。  在条件语句下面，需要将```MediaSessionCompat```对象设置为活动状态```STATE_PLAYING```，指定合适的```action```在```Lollipop```版本以前创建暂停按钮。

```
@Override
public void onPlay() {
    super.onPlay();
    if( !successfullyRetrievedAudioFocus() ) {
        return;
    }
 
    mMediaSessionCompat.setActive(true);
    setMediaPlaybackState(PlaybackStateCompat.STATE_PLAYING);
     
    ...
}
```

```setMediaPlaybackState()```方法用来辅助创建PlaybackStateCompat.Builder``````对象，并给它指定适当的动作和状态，然后构建```PlaybackStateCompat```并将其与```MediaSessionCompat```对象关联。

```
private void setMediaPlaybackState(int state) {
    PlaybackStateCompat.Builder playbackstateBuilder = new PlaybackStateCompat.Builder();
    if( state == PlaybackStateCompat.STATE_PLAYING ) {
        playbackstateBuilder.setActions(PlaybackStateCompat.ACTION_PLAY_PAUSE | PlaybackStateCompat.ACTION_PAUSE);
    } else {
        playbackstateBuilder.setActions(PlaybackStateCompat.ACTION_PLAY_PAUSE | PlaybackStateCompat.ACTION_PLAY);
    }
    playbackstateBuilder.setState(state, PlaybackStateCompat.PLAYBACK_POSITION_UNKNOWN, 0);
    mMediaSessionCompat.setPlaybackState(playbackstateBuilder.build());
}
```
重要的是要注意，需要在操作中同时使用```ACTION_PLAY_PAUSE``` 和```ACTION_PAUSE```或```ACTION_PLAY```标记以便在```Android Wear```上获得正确的控制。

Media notification on Android Wear
https://cms-assets.tutsplus.com/cdn-cgi/image/width=1700/uploads/users/798/posts/27030/image/wear.png

回到```onPlay()```中，您将希望通过使用我们之前定义的```MediaStyleHelper```类来显示与```MediaSessionCompat```对象关联的播放通知，然后显示该通知。
```
private void showPlayingNotification() {
    NotificationCompat.Builder builder = MediaStyleHelper.from(BackgroundAudioService.this, mMediaSessionCompat);
    if( builder == null ) {
        return;
    }
 
 
    builder.addAction(new NotificationCompat.Action(android.R.drawable.ic_media_pause, "Pause", MediaButtonReceiver.buildMediaButtonPendingIntent(this, PlaybackStateCompat.ACTION_PLAY_PAUSE)));
    builder.setStyle(new NotificationCompat.MediaStyle().setShowActionsInCompactView(0).setMediaSession(mMediaSessionCompat.getSessionToken()));
    builder.setSmallIcon(R.mipmap.ic_launcher);
    NotificationManagerCompat.from(BackgroundAudioService.this).notify(1, builder.build());
}
```
最后，在```onPlay()```方法启动```MediaPlayer```。

```
@Override
public void onPlay() {
    super.onPlay();
 
    ...
 
    showPlayingNotification();
    mMediaPlayer.start();
}
```
Media control notification on an Android Nougat device
https://cms-assets.tutsplus.com/cdn-cgi/image/width=1700/uploads/users/798/posts/27030/image/lollipop.png


当回调接收到暂停命令时，调用```onPause()```方法。  在这里，您将暂停```MediaPlayer```，将状态设置为```STATE_PAUSED```，并显示已暂停的通知。

```
@Override
public void onPause() {
    super.onPause();
 
    if( mMediaPlayer.isPlaying() ) {
        mMediaPlayer.pause();
        setMediaPlaybackState(PlaybackStateCompat.STATE_PAUSED);
        showPausedNotification();
    }
}
```
```showPausedNotification()```辅助方法看起来类似于```showPlayNotification()```方法。

```
private void showPausedNotification() {
    NotificationCompat.Builder builder = MediaStyleHelper.from(this, mMediaSessionCompat);
    if( builder == null ) {
        return;
    }
 
    builder.addAction(new NotificationCompat.Action(android.R.drawable.ic_media_play, "Play", MediaButtonReceiver.buildMediaButtonPendingIntent(this, PlaybackStateCompat.ACTION_PLAY_PAUSE)));
    builder.setStyle(new NotificationCompat.MediaStyle().setShowActionsInCompactView(0).setMediaSession(mMediaSessionCompat.getSessionToken()));
    builder.setSmallIcon(R.mipmap.ic_launcher);
    NotificationManagerCompat.from(this).notify(1, builder.build());
}
```
我们将讨论回调中的下一个方法  ```onPlayFromMediaId()```，以``` String```和 ```Bundle```为参数。  这是用来更改应用程序中音轨/内容的回调方法。 

对于本教程，我们将简单地接受原始资源ID并尝试播放，然后重新初始化会话的元数据。  当被允许将Bundle传入此方法时，可以使用它来自定义媒体播放的其他方面，例如为曲目设置自定义背景音。
```
@Override
public void onPlayFromMediaId(String mediaId, Bundle extras) {
    super.onPlayFromMediaId(mediaId, extras);
 
    try {
        AssetFileDescriptor afd = getResources().openRawResourceFd(Integer.valueOf(mediaId));
        if( afd == null ) {
            return;
        }
 
        try {
            mMediaPlayer.setDataSource(afd.getFileDescriptor(), afd.getStartOffset(), afd.getLength());
 
        } catch( IllegalStateException e ) {
            mMediaPlayer.release();
            initMediaPlayer();
            mMediaPlayer.setDataSource(afd.getFileDescriptor(), afd.getStartOffset(), afd.getLength());
        }
 
        afd.close();
        initMediaSessionMetadata();
 
    } catch (IOException e) {
        return;
    }
 
    try {
        mMediaPlayer.prepare();
    } catch (IOException e) {}
 
    //Work with extras here if you want
}
```
现在，我们已经讨论了该回调中使用的两个主要方法，重要的是要知道还有其他可选的方法用来自定义服务。  包括```onSeekTo()```方法，它允许更改内容的播放位置，```onCommand()```方法接受```String```表示命令的类型，```Bundle```表示有关该命令的额外信息，最后是```ResultReceiver```回调方法，允许发送自定义命令到```Service```。

```
@Override
public void onCommand(String command, Bundle extras, ResultReceiver cb) {
    super.onCommand(command, extras, cb);
    if( COMMAND_EXAMPLE.equalsIgnoreCase(command) ) {
        //Custom command here
    }
}
 
@Override
public void onSeekTo(long pos) {
    super.onSeekTo(pos);
}
```
# 撕毁
当我们的音频文件完成后，我们将要决定我们的下一个动作是什么。  虽然你可能想在你的应用程序中播放下一个音频，但是为了简单起见就释放MediaPlayer。
```
@Override
public void onCompletion(MediaPlayer mediaPlayer) {
    if( mMediaPlayer != null ) {
        mMediaPlayer.release();
    }
}
```
最后，我们要在```Service```的```onDestroy()```方法中做一些事情。  首先，获取对系统服务```AudioManager```的引用，并调用```abandonAudioFocus()```方法，传参为```AudioFocusChangeListener```，这将通知设备上的其他应用程序您将放弃音频焦点。  接下来，反注册监听耳机更改的```BroadcastReceiver```，并释放```MediaSessionCompat```对象。 最后，取消播放控制通知。

```
@Override
public void onDestroy() {
    super.onDestroy();
    AudioManager audioManager = (AudioManager) getSystemService(Context.AUDIO_SERVICE);
    audioManager.abandonAudioFocus(this);
    unregisterReceiver(mNoisyReceiver);
    mMediaSessionCompat.release();
    NotificationManagerCompat.from(this).cancel(1);
}
```
现在，应该有一个正在工作的基本背景音频```Service```，```MediaSessionCompat```用于跨设备播放控制。  虽然创建服务就已经涉及了很多知识，您应该能够从应用程序控制播放、通知、Lollipop设备前锁定屏幕（Lollipop设备及以后使用锁定屏幕上的通知）以及从外部设备（如Android Wear）控制这些操作，一旦Service启动后。

Media lock screen controls on Android Kit Kat
从Activity开始和控制内容
虽然大多数控件都是自动的，但仍然需要从应用控件中启动和控制媒体会话。  最起码，在应用中创建```MediaBrowserCompat.ConnectionCallback```、```MediaControllerCompat.Callback```、  ```MediaBrowserCompat和MediaControllerCompat```对象。

```MediaControllerCompat.Callback有个onPlaybackStateChanged()```方法 接收播放状态的变化，可用于保持同步UI。

```
private MediaControllerCompat.Callback mMediaControllerCompatCallback = new MediaControllerCompat.Callback() {
 
    @Override
    public void onPlaybackStateChanged(PlaybackStateCompat state) {
        super.onPlaybackStateChanged(state);
        if( state == null ) {
            return;
        }
         
        switch( state.getState() ) {
            case PlaybackStateCompat.STATE_PLAYING: {
                mCurrentState = STATE_PLAYING;
                break;
            }
            case PlaybackStateCompat.STATE_PAUSED: {
                mCurrentState = STATE_PAUSED;
                break;
            }
        }
    }
};
```
```MediaBrowserCompat.ConnectionCallback```有个onConnected()``````方法在```MediaBrowserCompat```对象被创建和连接时被调用。  你可以用它来初始化```MediaControllerCompat```对象，将其链接到```MediaControllerCompat.Callback```，并将其与```Service```的```MediaSessionCompat```关联。  完成后，从这个方法开始音频播放。

```
private MediaBrowserCompat.ConnectionCallback mMediaBrowserCompatConnectionCallback = new MediaBrowserCompat.ConnectionCallback() {
 
    @Override
    public void onConnected() {
        super.onConnected();
        try {
            mMediaControllerCompat = new MediaControllerCompat(MainActivity.this, mMediaBrowserCompat.getSessionToken());
            mMediaControllerCompat.registerCallback(mMediaControllerCompatCallback);
            setSupportMediaController(mMediaControllerCompat);
            getSupportMediaController().getTransportControls().playFromMediaId(String.valueOf(R.raw.warner_tautz_off_broadway), null);
 
        } catch( RemoteException e ) {
 
        }
    }
};
```
您会注意到上述代码段使用```getSupportMediaController().getTransportControls()```方法与媒体会话进行通信。  使用相同的技术，您可以在音频服务的  ```MediaSessionCompat.Callback```对象调用```onPlay()```和```onPause()```。
```
if( mCurrentState == STATE_PAUSED ) {
    getSupportMediaController().getTransportControls().play();
    mCurrentState = STATE_PLAYING;
} else {
    if( getSupportMediaController().getPlaybackState().getState() == PlaybackStateCompat.STATE_PLAYING ) {
        getSupportMediaController().getTransportControls().pause();
    }
 
    mCurrentState = STATE_PAUSED;
}
```
完成音频播放后，可以暂停音频服务并断开MediaBrowserCompat对象，Activity被摧毁时执行该操作。
```
@Override
protected void onDestroy() {
    super.onDestroy();
    if( getSupportMediaController().getPlaybackState().getState() == PlaybackStateCompat.STATE_PLAYING ) {
        getSupportMediaController().getTransportControls().pause();
    }
 
    mMediaBrowserCompat.disconnect();
}
```


# 总结 
哇！  正如你所见，本文研究了如何创建和使用背景音频服务。

在本教程中，创建了一个播放简单音频文件的服务，监听音频焦点更改以及指向MediaSessionCompat的链接在Android设备（包括手机和Android Wear）上提供通用播放控制。  如果在本教程中遇到问题，我强烈建议查看Envato Tuts + GitHub上相关的Android项目代码。