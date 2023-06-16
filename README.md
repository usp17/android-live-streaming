# Android Live Streaming with Video SDK

It’s often that you’ve consumed a fair amount of live streams if you’re an avid consumer of online content. With live streaming becoming the much preferred source of learning and entertainment, it’s hard to miss out on live broadcasts. whether it’s for following sports events, attending online classes, watching fitness lessons, or engaging with celebrities.

Many of the live streaming applications rely on HTTP Live Streaming (HLS - a widely used protocol) to deliver content to their audiences. In fact, if you’ve ever watched an Instagram live stream, you’ve already had a brush with the magic of HLS.

If you’re a developer looking to build a top-notch live streaming experience in your Android app, this article is for you.

# Why choose the Video SDK?

Video SDK is the perfect choice for those seeking a live streaming platform that offers features to create high-quality streams. This platform supports various features like screen sharing, Real-time Messaging, allows broadcasters to invite audience to the stage and supports 100 Participants, ensuring that your live streams are interactive and engaging. With Video SDK, you can also use your own custom designed layout template for live streaming.

In terms of integration, Video SDK is quite simple and quick to integrate, that allows you to seamlessly integrate live streaming in your app. This ensures that you can enjoy the benefits of live streaming without any technical difficulties or lengthy implementation processes.

Moreover, Video SDK is budget-friendly, making it an affordable option for businesses of all sizes. You can enjoy the benefits of a feature-rich live streaming platform without breaking the bank, making it an ideal choice for startups and small businesses.

# To Build a Live Streaming Android app

The below steps will give you all the information to quickly build an interactive live streaming app. Please carefully follow along. having any trouble? Let us know right away on Discord and we will be happy to help you.

# Prerequisite

- Java Development Kit.
- Android Studio 3.0 or later.
- Android SDK API Level 21 or higher.
- A mobile device that runs Android 5.0 or later.
- A token from Video SDK dashboard

# Setup the project
## Create new Project

- In Android Studio, create a Phone and Tablet Android project with an Empty Activity.
![](https://docs.videosdk.live/assets/images/android_newProject-a43ee8e23168f392da4bcb2ae9ac6fb3.png)

- Next step is to provide a name. We have set the name as HLSDemo.
![](https://www.videosdk.live/_next/image?url=https%3A%2F%2Fassets.videosdk.live%2Fstatic-assets%2Fghost%2F2023%2F04%2FScreenshot-from-2023-04-05-12-18-04.png&w=1040&q=75)

## Integrate Video SDK

- Add the repository to project’s settings.gradle file.

```js
dependencyResolutionManagement{
  repositories {
    // ...
    google()
    mavenCentral()
    maven { url 'https://jitpack.io' }
    maven { url "https://maven.aliyun.com/repository/jcenter" }
  }
}
```

- Add the following dependency in your app’s build.gradle.

```js
dependencies {
  implementation 'live.videosdk:rtc-android-sdk:0.1.15'
  // library to perform Network call to generate a meeting id
  implementation 'com.amitshekhar.android:android-networking:1.0.2'
  // other app dependencies
  }
```

If your project has set android.useAndroidX=true, then set android.enableJetifier=true in the gradle.properties file to migrate your project to AndroidX and avoid duplicate class conflict.

## Add permissions into your project

In `/app/Manifests/AndroidManifest.xml`, add the following permissions after `</application>`.

```js
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS"/>
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.CAMERA" />
```

# Structure of project

We will create two activity and two fragments. First activity is `JoinActivity` , which allows user to create/join the meeting, and another one is `MeetingActivity` ,which will initialize meeting and replace `mainLayout` with `SpeakerFragment` or with `ViewerFragment` according to user's choice.

Our project structure would look like this.

```js
app
   ├── java
   │    ├── packagename
   │         ├── JoinActivity
   │         ├── MeetingActivity
   │         ├── SpeakerAdapter
   │         ├── SpeakerFragment
   |         ├── ViewerFragment
   ├── res
   │    ├── layout
   │    │    ├── activity_join.xml
   │    │    ├── activity_meeting.xml
   |    |    ├── fragment_speaker.xml
   |    |    ├── fragment_viewer.xml
   │    │    ├── item_remote_peer.xmlCopy
```

You have to set JoinActivity as Launcher activity.

# App Architecture
![](https://www.videosdk.live/_next/image?url=https%3A%2F%2Fcdn.videosdk.live%2Fwebsite-resources%2Fdocs-resources%2Fandroid_ils_quickstart_app_structure.png&w=1040&q=75)

# Step 1: Creating Joining Screen

Create a new Activity named `JoinActivity`

## Creating UI for Joining Screen

The Joining screen will include :

1. Create Button — This button will create a new meeting for you.
2. TextField for Meeting Id — This text field will contain the meeting Id you want to join.
3. Join as Host Button — This button will join the meeting as host with `meetingId` you provided. 
4. Join as Viewer Button — This button will join the meeting as viewer with `meetingId` you provided.

In `/app/res/layout/activity_join.xml` file, replace the content with the following.

```js
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/createorjoinlayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/black"
    android:gravity="center"
    android:orientation="vertical">

    <Button
        android:id="@+id/btnCreateMeeting"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Create Meeting"
        android:textAllCaps="false" />

    <TextView
        android:id="@+id/tvText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:paddingVertical="5sp"
        android:text="OR"
        android:textColor="@color/white"
        android:textSize="20sp" />

    <EditText
        android:id="@+id/etMeetingId"
        android:theme="@android:style/Theme.Holo"
        android:layout_width="250dp"
        android:layout_height="wrap_content"
        android:hint="Enter Meeting Id"
        android:textColor="@color/white"
        android:textColorHint="@color/white" />

    <Button
        android:id="@+id/btnJoinHostMeeting"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="8sp"
        android:text="Join as Host"
        android:textAllCaps="false" />

    <Button
        android:id="@+id/btnJoinViewerMeeting"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Join as Viewer"
        android:textAllCaps="false" />

</LinearLayout>
```

## Integration of Create Meeting API

1. Create field `sampleToken` in `JoinActivity` which will hold the generated token from the [VideoSDK dashboard](https://app.videosdk.live/api-keys). This token will be used in VideoSDK config as well as generating meetingId.

```js
class JoinActivity : AppCompatActivity() {

  //Replace with the token you generated from the VideoSDK Dashboard
  private var sampleToken = ""

  override fun onCreate(savedInstanceState: Bundle?) {
    //...
  }
}
```

2. On Join Button as Host onClick events, we will navigate to MeetingActivity with token, meetingId and mode as CONFERENCE.

3. On Join Button as Viewer onClick events, we will navigate to MeetingActivity with token, meetingId and mode as Viewer.

```js
class JoinActivity : AppCompatActivity() {

   //Replace with the token you generated from the VideoSDK Dashboard
   private var sampleToken = "" 

   override fun onCreate(savedInstanceState: Bundle?) {
      super.onCreate(savedInstanceState)
      setContentView(R.layout.activity_join)

      val btnCreate = findViewById<Button>(R.id.btnCreateMeeting)
      val btnJoinHost = findViewById<Button>(R.id.btnJoinHostMeeting)
      val btnJoinViewer = findViewById<Button>(R.id.btnJoinViewerMeeting)
      val etMeetingId = findViewById<EditText>(R.id.etMeetingId)

      // create meeting and join as Host
      btnCreate.setOnClickListener {
          createMeeting(
              sampleToken
          )
      }

      // Join as Host
      btnJoinHost.setOnClickListener {
          val intent = Intent(this@JoinActivity, MeetingActivity::class.java)
          intent.putExtra("token", sampleToken)
          intent.putExtra("meetingId", etMeetingId.text.toString().trim { it <= ' ' })
          intent.putExtra("mode", "CONFERENCE")
          startActivity(intent)
      }

      // Join as Viewer
      btnJoinViewer.setOnClickListener {
          val intent = Intent(this@JoinActivity, MeetingActivity::class.java)
          intent.putExtra("token", sampleToken)
          intent.putExtra("meetingId", etMeetingId.text.toString().trim { it <= ' ' })
          intent.putExtra("mode", "VIEWER")
          startActivity(intent)
      }
    }

    private fun createMeeting(token: String) {
      // we will explore this method in the next step
    }
```

4. For Create Button, under createMeeting method we will generate meetingId by calling API and navigate to MeetingActivity with token, generated meetingId and mode as CONFERENCE.

```js
class JoinActivity : AppCompatActivity() {
  //...onCreate
 private fun createMeeting(token: String) {
  // we will make an API call to VideoSDK Server to get a roomId
  AndroidNetworking.post("https://api.videosdk.live/v2/rooms")
      .addHeaders("Authorization", token) //we will pass the token in the Headers
      .build()
      .getAsJSONObject(object : JSONObjectRequestListener {
          override fun onResponse(response: JSONObject) {
            try {
              // response will contain `roomId`
              val meetingId = response.getString("roomId")

              // starting the MeetingActivity with received roomId and our sampleToken
              val intent = Intent(this@JoinActivity, MeetingActivity::class.java)
              intent.putExtra("token", sampleToken)
              intent.putExtra("meetingId", meetingId)
              intent.putExtra("mode", "CONFERENCE")
              startActivity(intent)
            } catch (e: JSONException) {
                e.printStackTrace()
            }
          }

          override fun onError(anError: ANError) {
            anError.printStackTrace()
            Toast.makeText(this@JoinActivity, anError.message, Toast.LENGTH_SHORT)
                .show()
          }
      })
  }
}
```

5. Our App is completely based on audio and video commutation, that’s why we need to ask for runtime permissions `RECORD_AUDIO` and `CAMERA`. So, we will implement permission logic on `JoinActivity`.

```js
class JoinActivity : AppCompatActivity() {
  companion object {
    private const val PERMISSION_REQ_ID = 22
    private val REQUESTED_PERMISSIONS = arrayOf(
        Manifest.permission.RECORD_AUDIO,
        Manifest.permission.CAMERA
    )
  }

  private fun checkSelfPermission(permission: String, requestCode: Int) {
    if (ContextCompat.checkSelfPermission(this, permission) !=
        PackageManager.PERMISSION_GRANTED
    ) {
        ActivityCompat.requestPermissions(this, REQUESTED_PERMISSIONS, requestCode)
    }
  }

  override fun onCreate(savedInstanceState: Bundle?) {
    //... button listeneres
    checkSelfPermission(REQUESTED_PERMISSIONS[0], PERMISSION_REQ_ID)
    checkSelfPermission(REQUESTED_PERMISSIONS[1], PERMISSION_REQ_ID)
  }
}
```

You will get Unresolved reference: MeetingActivity error, but don't worry. It will automatically solved once you create MeetingActivity.

# Step 2: Creating Meeting Screen

## Create a new Activity named MeetingActivity.

Creating the UI for Meeting Screen

In `/app/res/layout/activity_meeting.xml` file, replace the content with the following.

```js
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/mainLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/black"
    tools:context=".MeetingActivity">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:text="Creating a meeting for you"
        android:textColor="@color/white"
        android:textFontWeight="700"
        android:textSize="20sp" />

</RelativeLayout>
```

## Initializing the Meeting

After getting token, meetigId and mode from `JoinActivity`,

1. Initialize VideoSDK.
2. Configure VideoSDK with token.
3. Initialize the meeting with required params such as `meetingId`, `participantName`, `micEnabled`, `webcamEnabled `, `mode` and more.
4. Join the room with `meeting.join()` method.
5. Add `MeetingEventListener` for listening Meeting Join event.
6. Check mode of `localParticipant`, If mode is CONFERENCE than we will replace mainLayout with `SpeakerFragment` otherwise replace with `ViewerFragment`.

```js
class MeetingActivity : AppCompatActivity() {
  var meeting: Meeting? = null
      private set

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_meeting)

    val meetingId = intent.getStringExtra("meetingId")
    val token = intent.getStringExtra("token")
    val mode = intent.getStringExtra("mode")
    val localParticipantName = "John Doe"
    val streamEnable = mode == "CONFERENCE"

    // initialize VideoSDK
    VideoSDK.initialize(applicationContext)

    // Configuration VideoSDK with Token
    VideoSDK.config(token)

    // Initialize VideoSDK Meeting
    meeting = VideoSDK.initMeeting(
        this@MeetingActivity, meetingId, localParticipantName,
        streamEnable, streamEnable, null, mode, null
    )

    // join Meeting
    meeting!!.join()

    // if mode is CONFERENCE than replace mainLayout with SpeakerFragment otherwise with ViewerFragment
    meeting!!.addEventListener(object : MeetingEventListener() {
      override fun onMeetingJoined() {
          if (meeting != null) {
            if (mode == "CONFERENCE") {
              //pin the local partcipant
              meeting!!.localParticipant.pin("SHARE_AND_CAM")
              supportFragmentManager
                  .beginTransaction()
                  .replace(R.id.mainLayout, SpeakerFragment(), "MainFragment")
                  .commit()
              } else if (mode == "VIEWER") {
                supportFragmentManager
                    .beginTransaction()
                    .replace(R.id.mainLayout, ViewerFragment(), "viewerFragment")
                    .commit()
              }
          }
      }
    })
  }
}
```

# Step 3: Implement SpeakerView

After successfully entering into the meeting, it’s time to render speaker’s view and manage controls such as toggle webcam/mic,start/stop HLS and leave the meeting.

1. Create a new fragment named `SpeakerFragment`.
2. In `/app/res/layout/fragment_speaker.xml` file, replace the content with the following.

```js
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/black"
    android:gravity="center"
    android:orientation="vertical"
    tools:context=".SpeakerFragment">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginVertical="8sp"
        android:paddingHorizontal="10sp">

        <TextView
            android:id="@+id/tvMeetingId"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:text="Meeting Id : "
            android:textColor="@color/white"
            android:textSize="18sp"
            android:layout_weight="3"/>

        <Button
            android:id="@+id/btnLeave"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:text="Leave"
            android:textAllCaps="false"
            android:layout_weight="1"/>

    </LinearLayout>

    <TextView
        android:id="@+id/tvHlsState"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Current HLS State : NOT_STARTED"
        android:textColor="@color/white"
        android:textSize="18sp" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/rvParticipants"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_marginVertical="10sp"
        android:layout_weight="1" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center">

        <Button
            android:id="@+id/btnHLS"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Start HLS"
            android:textAllCaps="false" />

        <Button
            android:id="@+id/btnWebcam"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginHorizontal="5sp"
            android:text="Toggle Webcam"
            android:textAllCaps="false" />

        <Button
            android:id="@+id/btnMic"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Toggle Mic"
            android:textAllCaps="false" />

    </LinearLayout>

</LinearLayout>
```

3. Now, let’s set listener for buttons which will allow the participant to toggle media.

```js
class SpeakerFragment : Fragment() {
  private var micEnabled = true
  private var webcamEnabled = true
  private var hlsEnabled = false
  private var btnMic: Button? = null
  private var btnWebcam: Button? = null
  private var btnHls: Button? = null
  private var btnLeave: Button? = null
  private var tvMeetingId: TextView? = null
  private var tvHlsState: TextView? = null
  override fun onAttach(context: Context) {
    super.onAttach(context)
    mContext = context
    if (context is Activity) {
      mActivity = context
      // getting meeting object from Meeting Activity
      meeting = (mActivity as MeetingActivity?)!!.meeting
    }
  }

  override fun onCreateView(
      inflater: LayoutInflater, container: ViewGroup?,
      savedInstanceState: Bundle?
  ): View? {
    // Inflate the layout for this fragment
    val view = inflater.inflate(R.layout.fragment_speaker, container, false)
    btnMic = view.findViewById(R.id.btnMic)
    btnWebcam = view.findViewById(R.id.btnWebcam)
    btnHls = view.findViewById(R.id.btnHLS)
    btnLeave = view.findViewById(R.id.btnLeave)
    tvMeetingId = view.findViewById(R.id.tvMeetingId)
    tvHlsState = view.findViewById(R.id.tvHlsState)
    if (meeting != null) {
      tvMeetingId!!.text = "Meeting Id : " + meeting!!.meetingId
      setActionListeners()
    }
    return view
  }

```

```js
  private fun setActionListeners() {}

  companion object {
    private var mActivity: Activity? = null
    private var mContext: Context? = null
    private var meeting: Meeting? = null
  }
}

private fun setActionListeners() {
    btnMic!!.setOnClickListener {
      if (micEnabled) {
        meeting!!.muteMic()
        Toast.makeText(mContext, "Mic Muted", Toast.LENGTH_SHORT).show()
      } else {
        meeting!!.unmuteMic()
        Toast.makeText(
            mContext,
            "Mic Enabled",
            Toast.LENGTH_SHORT
        ).show()
      }
      micEnabled = !micEnabled
    }
    btnWebcam!!.setOnClickListener {
      if (webcamEnabled) {
        meeting!!.disableWebcam()
        Toast.makeText(
            mContext,
            "Webcam Disabled",
            Toast.LENGTH_SHORT
        ).show()
      } else {
        meeting!!.enableWebcam()
        Toast.makeText(
            mContext,
            "Webcam Enabled",
            Toast.LENGTH_SHORT
        ).show()
      }
      webcamEnabled = !webcamEnabled
    }
    btnLeave!!.setOnClickListener { meeting!!.leave() }
    btnHls!!.setOnClickListener {
      if (!hlsEnabled) {
        val config = JSONObject()
        val layout = JSONObject()
        JsonUtils.jsonPut(layout, "type", "SPOTLIGHT")
        JsonUtils.jsonPut(layout, "priority", "PIN")
        JsonUtils.jsonPut(layout, "gridSize", 4)
        JsonUtils.jsonPut(config, "layout", layout)
        JsonUtils.jsonPut(config, "orientation", "portrait")
        JsonUtils.jsonPut(config, "theme", "DARK")
        JsonUtils.jsonPut(config, "quality", "high")
        meeting!!.startHls(config)
      } else {
        meeting!!.stopHls()
      }
    }
  }
```

4. After adding listeners for buttons, let’s add MeetingEventListener to the meeting and remove all listners in onDestroy() method.

```js
class SpeakerFragment : Fragment() {

  override fun onCreateView(
      inflater: LayoutInflater, container: ViewGroup?,
      savedInstanceState: Bundle?
  ): View? {
    //...
    if (meeting != null) {
      //...
      // add Listener to the meeting
      meeting!!.addEventListener(meetingEventListener)
    }
    return view
  }

  private val meetingEventListener: MeetingEventListener = object : MeetingEventListener() {
      override fun onMeetingLeft() {
        // unpin the local participant
        meeting!!.localParticipant.unpin("SHARE_AND_CAM")
        if (isAdded) {
          val intents = Intent(mContext, JoinActivity::class.java)
          intents.addFlags(
              Intent.FLAG_ACTIVITY_NEW_TASK
                      or Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_CLEAR_TASK
          )
          startActivity(intents)
          mActivity!!.finish()
        }
      }

      @RequiresApi(api = Build.VERSION_CODES.P)
      override fun onHlsStateChanged(HlsState: JSONObject) {
        if (HlsState.has("status")) {
          try {
              tvHlsState!!.text = "Current HLS State : " + HlsState.getString("status")
              if (HlsState.getString("status") == "HLS_STARTED") {
                hlsEnabled = true
                btnHls!!.text = "Stop HLS"
              }
              if (HlsState.getString("status") == "HLS_STOPPED") {
                hlsEnabled = false
                btnHls!!.text = "Start HLS"
              }
            } catch (e: JSONException) {
                e.printStackTrace()
            }
        }
      }
  }

  override fun onDestroy() {
      mContext = null
      mActivity = null
      if (meeting != null) {
          meeting!!.removeAllListeners()
          meeting = null
      }
      super.onDestroy()
  }
}
```

5. Next step is to render speaker’s view. With RecyclerView, we will display a list of participant who joined the meeting as a host.

- Create a new layout for the participant view named `item_remote_peer.xml` in the `res/layout folder`.

```js
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="200dp"
    android:background="@color/cardview_dark_background"
    tools:layout_height="200dp">

    <live.videosdk.rtc.android.VideoView
        android:id="@+id/participantView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:visibility="gone" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom"
        android:background="#99000000"
        android:orientation="horizontal">

        <TextView
            android:id="@+id/tvName"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            android:padding="4dp"
            android:textColor="@color/white" />

    </LinearLayout>

</FrameLayout>
```

- Create a recycler view adapter named SpeakerAdapter which will show the participant list. Create PeerViewHolder in the adapter which will extend RecyclerView.ViewHolder.

```js
class SpeakerAdapter(private val meeting: Meeting) :
    RecyclerView.Adapter<SpeakerAdapter.PeerViewHolder?>() {
    private var participantList: MutableList<Participant> = ArrayList()

    init {
      updateParticipantList()
      // adding Meeting Event listener to get the participant join/leave event in the meeting.
      meeting.addEventListener(object : MeetingEventListener() {
        override fun onParticipantJoined(participant: Participant) {
          // check participant join as Host/Speaker or not
          if (participant.mode == "CONFERENCE") {
              // pin the participant
              participant.pin("SHARE_AND_CAM")
              // add participant in participantList
              participantList.add(participant)
          }
          notifyDataSetChanged()
        }

        override fun onParticipantLeft(participant: Participant) {
          var pos = -1
          for (i in participantList.indices) {
              if (participantList[i].id == participant.id) {
                  pos = i
                  break
              }
          }
          if (participantList.contains(participant)) {
              // unpin participant who left the meeting
              participant.unpin("SHARE_AND_CAM")
              // remove participant from participantList
              participantList.remove(participant)
          }
          if (pos >= 0) {
              notifyItemRemoved(pos)
          }
        }
      })
    }

    private fun updateParticipantList() {
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PeerViewHolder {
    }

    override fun onBindViewHolder(holder: PeerViewHolder, position: Int) {
    }

    override fun getItemCount(): Int {
    }

    class PeerViewHolder(view: View) : RecyclerView.ViewHolder(view) {
    }
}
```

```js
private fun updateParticipantList() {
      // adding the local participant(You) to the list
      participantList.add(meeting.localParticipant)

      // adding participants who join as Host/Speaker
      val participants: Iterator<Participant> = meeting.participants.values.iterator()
      for (i in 0 until meeting.participants.size) {
        val participant = participants.next()
        if (participant.mode == "CONFERENCE") {
            // pin the participant
            participant.pin("SHARE_AND_CAM")
            // add participant in participantList
            participantList.add(participant)
        }
      }
    }

```

```js
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PeerViewHolder {
      return PeerViewHolder(
          LayoutInflater.from(parent.context).inflate(R.layout.item_remote_peer, parent, false)
      )
    }

    override fun onBindViewHolder(holder: PeerViewHolder, position: Int) {
      val participant = participantList[position]
      holder.tvName.text = participant.displayName

      // adding the initial video stream for the participant into the 'VideoView'
      for ((_, stream) in participant.streams) {
        if (stream.kind.equals("video", ignoreCase = true)) {
          holder.participantView.visibility = View.VISIBLE
          val videoTrack = stream.track as VideoTrack
          holder.participantView.addTrack(videoTrack)
          break
        }
      }

      // add Listener to the participant which will update start or stop the video stream of that participant
      participant.addEventListener(object : ParticipantEventListener() {
        override fun onStreamEnabled(stream: Stream) {
          if (stream.kind.equals("video", ignoreCase = true)) {
            holder.participantView.visibility = View.VISIBLE
            val videoTrack = stream.track as VideoTrack
            holder.participantView.addTrack(videoTrack)
          }
        }

        override fun onStreamDisabled(stream: Stream) {
          if (stream.kind.equals("video", ignoreCase = true)) {
            holder.participantView.removeTrack()
            holder.participantView.visibility = View.GONE
          }
        }
      })
    }

    override fun getItemCount(): Int {
      return participantList.size
    }

    class PeerViewHolder(view: View) : RecyclerView.ViewHolder(view) {
      // 'VideoView' to show Video Stream
      var participantView: VideoView
      var tvName: TextView

      init {
        tvName = view.findViewById(R.id.tvName)
        participantView = view.findViewById(R.id.participantView)
      }
    }
```

6. Add this adapter to the `SpeakerFragment`

```js
override fun onCreateView(
      inflater: LayoutInflater, container: ViewGroup?,
      savedInstanceState: Bundle?
  ): View? {
    //...
    if (meeting != null) {
      //...
      val rvParticipants = view.findViewById<RecyclerView>(R.id.rvParticipants)
      rvParticipants.layoutManager = GridLayoutManager(mContext, 2)
      rvParticipants.adapter = SpeakerAdapter(meeting!!)
  }
}

```
# Step 4: Implement ViewerView

When host starts the live streaming, viewer will be able to see the live streaming.
To implement player view, we are going to use ExoPlayer. It will be helpful to play hls stream.
Let's first add dependency into the project.

```js
dependencies {
  implementation 'com.google.android.exoplayer:exoplayer:2.18.5'
  // other app dependencies
}
```

Create a new Fragment named ViewerFragment.

## Creating the UI for Viewer Fragment

The Viewer Fragment will include :

1. TextView for Meeting Id — The meeting Id that you have joined with, will be displayed in this text view.
2. Leave Button — This button will leave the meeting.
3. waitingLayout — This is textView that will be shown when there is no active HLS.
4. StyledPlayerView — This is mediaplayer which will display livestreaming.

In `/app/res/layout/fragment_viewer.xml` file, replace the content with the following.

```js
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/black"
    tools:context=".ViewerFragment">

    <LinearLayout
        android:id="@+id/meetingLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingHorizontal="12sp"
        android:paddingVertical="5sp">

        <TextView
            android:id="@+id/meetingId"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="3"
            android:text="Meeting Id : "
            android:textColor="@color/white"
            android:textSize="20sp" />

        <Button
            android:id="@+id/btnLeave"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Leave" />

    </LinearLayout>

    <TextView
        android:id="@+id/waitingLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="Waiting for host \n to start the live streaming"
        android:textColor="@color/white"
        android:textFontWeight="700"
        android:textSize="20sp"
        android:gravity="center"/>

    <com.google.android.exoplayer2.ui.StyledPlayerView
        android:id="@+id/player_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:visibility="gone"
        app:resize_mode="fixed_width"
        app:show_buffering="when_playing"
        app:show_subtitle_button="false"
        app:use_artwork="false"
        app:show_next_button="false"
        app:show_previous_button="false"
        app:use_controller="true"
        android:layout_below="@id/meetingLayout"/>

</RelativeLayout>
```

## Initialize player and Playing HLS stream

1. Initialize player and play the HLS when the meeting HLS state is HLS_PLAYABLE, and release it when the HLS state is HLS_STOPPED. Whenever the meeting HLS state changes, the event onHlsStateChanged will be triggered.

```js
class ViewerFragment : Fragment() {
  private var meeting: Meeting? = null
  private var playerView: StyledPlayerView? = null
  private var waitingLayout: TextView? = null
  private var player: ExoPlayer? = null
  private var dataSourceFactory: DefaultHttpDataSource.Factory? = null
  private val startAutoPlay = true
  private var downStreamUrl: String? = ""

  override fun onCreateView(
    inflater: LayoutInflater, container: ViewGroup?,
    savedInstanceState: Bundle?
  ): View? {
    // Inflate the layout for this fragment
    val view = inflater.inflate(R.layout.fragment_viewer, container, false)
    playerView = view.findViewById(R.id.player_view)
    waitingLayout = view.findViewById(R.id.waitingLayout)
    if (meeting != null) {
        // set MeetingId to TextView
        (view.findViewById<View>(R.id.meetingId) as TextView).text =
            "Meeting Id : " + meeting!!.meetingId
        // leave the meeting on btnLeave click
        (view.findViewById<View>(R.id.btnLeave) as Button).setOnClickListener { meeting!!.leave() }
        // add listener to meeting
        meeting!!.addEventListener(meetingEventListener)
    }
    return view
  }

  override fun onAttach(context: Context) {
      super.onAttach(context)
      mContext = context
      if (context is Activity) {
        mActivity = context
        // get meeting object from MeetingActivity
        meeting = (mActivity as MeetingActivity?)!!.meeting
      }
  }
```

```js
  private val meetingEventListener: MeetingEventListener = object : MeetingEventListener() {}

  override fun onDestroy() {
  }

  companion object {
    private var mActivity: Activity? = null
    private var mContext: Context? = null
  }
}

private val meetingEventListener: MeetingEventListener = object : MeetingEventListener() {
      override fun onMeetingLeft() {
        if (isAdded) {
          val intents = Intent(mContext, JoinActivity::class.java)
          intents.addFlags(
              Intent.FLAG_ACTIVITY_NEW_TASK
                      or Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_CLEAR_TASK
          )
          startActivity(intents)
          mActivity!!.finish()
        }
      }

      @RequiresApi(api = Build.VERSION_CODES.P)
      override fun onHlsStateChanged(HlsState: JSONObject) {
        if (HlsState.has("status")) {
            try {
              if (HlsState.getString("status") == "HLS_PLAYABLE" && HlsState.has("downstreamUrl")) {
                downStreamUrl = HlsState.getString("downstreamUrl")
                waitingLayout!!.visibility = View.GONE
                playerView!!.visibility = View.VISIBLE
                // initialize player
                initializePlayer()
              }
              if (HlsState.getString("status") == "HLS_STOPPED") {
                // release the player
                releasePlayer()
                downStreamUrl = null
                waitingLayout!!.text = "Host has stopped \n the live streaming"
                waitingLayout!!.visibility = View.VISIBLE
                playerView!!.visibility = View.GONE
              }
            } catch (e: JSONException) {
                e.printStackTrace()
            }
          }
      }
  }
```
  

```js
  private fun initializePlayer() {
  }
 
   private fun releasePlayer() {
   }

private fun initializePlayer() {
    if (player == null) {
      dataSourceFactory = DefaultHttpDataSource.Factory()
      val mediaSource = HlsMediaSource.Factory(dataSourceFactory!!).createMediaSource(
          MediaItem.fromUri(Uri.parse(downStreamUrl))
      )
      val playerBuilder = ExoPlayer.Builder( /* context = */mContext!!)
      player = playerBuilder.build()
      // auto play when player is ready
      player!!.playWhenReady = startAutoPlay
      player!!.setMediaSource(mediaSource)
      // if you want display setting for player then remove this line
      playerView!!.findViewById<View>(com.google.android.exoplayer2.ui.R.id.exo_settings).visibility =
          View.GONE
      playerView!!.player = player
    }
    player!!.prepare()
  }
```

```js
 private fun releasePlayer() {
    if (player != null) {
      player!!.release()
      player = null
      dataSourceFactory = null
      playerView!!.player = null
    }
  }

  override fun onDestroy() {
    mContext = null
    mActivity = null
    downStreamUrl = null
    releasePlayer()
    if (meeting != null) {
        meeting!!.removeAllListeners()
        meeting = null
    }
    super.onDestroy()
  }

```
This is how the viewers will see their screen.

<a href="http://www.youtube.com/watch?feature=player_embedded&v=NIqR32ajQBc" target="_new"><img src="http://img.youtube.com/vi/NIqR32ajQBc/0.jpg" alt="Output" border="10" /></a>

# Run your App

Tadaa!! Our app is ready for live streaming. Easy, isn’t it ?
Install and run the app on two different devices and make sure both of them are connected to the internet.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nx2t0my44hhvld1gtlcj.gif)

# Conclusion

In this blog, we got familiar with VideoSDK and how to create your own Live Streaming Android App with the Video SDK.

Go ahead and create advanced features like screen-sharing, Real-Time messaging and others. Browse Our [documentation](https://docs.videosdk.live/).

To see the full implementation of the app, check out this [GitHub repository](https://github.com/videosdk-live/videosdk-rtc-android-java-sdk-example/tree/one-to-one-demo).

If you face any problem or have questions, Feel free to join our [Discord community](https://discord.com/invite/Gpmj6eCq5u).

# More Android Resources

[Build an Android Video Calling App — docs](https://www.videosdk.live/blog/android-live-streaming#more-android-resources)

[Build an Android Video Calling App using Android Studio and Video SDK — Youtube](https://www.youtube.com/watch?v=Kj7jS3dbJFA)

[quickstart/android-rtc](https://github.com/videosdk-live/quickstart/tree/main/android-rtc)

[quickstart/android-hls](https://github.com/videosdk-live/quickstart/tree/main/android-hls)

[videosdk-rtc-android-java-sdk-example](https://github.com/videosdk-live/videosdk-rtc-android-java-sdk-example)

[videosdk-rtc-android-kotlin-sdk-example](https://github.com/videosdk-live/videosdk-rtc-android-kotlin-sdk-example)

[videosdk-hls-android-java-example](https://github.com/videosdk-live/videosdk-hls-android-java-example)

[videosdk-hls-android-kotlin-example](https://github.com/videosdk-live/videosdk-hls-android-kotlin-example)
