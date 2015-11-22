---
layout: post
title: How to use the Harman Android SDK
---

In an attempt to be more developer friendly and to reach out to the tech community, Harman Kardon has released an SDK for their Omni speakers. Now, any developer with a speaker can create speaker-related apps with their products.   
  
  Here's the rub: their SDK is pretty terribly documented.  
   
 [Carl Saldanha wrote a tutorial that shows how to use their SDK](http://cjds.github.io/me/2015/08/31/Harman-Kardon-SDK-For-Android/), but this tutorial will be more explanatory: I'll be introducing how to use the SDK by explaining, showing, and identifying some potential pitfalls. 
   
 Let's get into it!
 
### HarmanSDK Util

In the sample app that Harman provides, there is a `Util.java` file. It's incredibly useful because it abstracts away lots of functions for interacting with their SDK. We'll definitely be using it in this tutorial - [find the code for that here](https://github.com/cjds/wirelessomnitutorial/blob/master/Util.java) (renamed to `HarmanSDKUtil.java` for this tutorial)
  
### Step 1: Pairing the speakers
The first thing you have to do is download the HKController app onto your phone and pair the speakers. 
The process is pretty straightforward. Here's what you will be doing *(note that this is manual mode)*:  
+ the speaker will emit its own wifi network. You'll choose your speaker from a list of omni networks
+ your phone will connect to the speaker  
+ the app will ask you to find the original network you were connected to (your home network) so that it can tell the speaker to connect to it too.   
+ after about 5-20, the speaker's light should be a solid white color and be connected to your wifi network.   
+ your phone can now connect directly to the speaker and controlit  
> Note: when you selecting your home network after selecting the speaker to connect, You might not be able to find yours. Just press cancel and choose the speaker again. It'll show up eventually.

### Step 2: Setting up the app
Before anything else, [get the sdk here](http://www.developer.harman.com/fileMedia/download/268786f1-30c1-44c4-9197-e34b3e76edde).
  
Download, unzip it, go to the `/lib` directory and move the `HKWirelessHD.jar` file into your project's `/app/libs` directory. If you don't have one, create one! It's where all your local libraries should go anyway.   
  
After it's there, add this line to the dependancies section of your build.gradle file:  
  
	compile files('libs/HKWirelessHD.jar')
  
This gives you access to all of the classes that Harman provides for your apps to use to interact with their speakers.
  
Next, create a `jniLibs` folder in your `/app/src` folder and copy the `/examples/wirelessomni/libs/armeabi/` and `/examples/wirelessomni/libs/armeabi-v7a/` folders from the example into it. These are `.so` files that I have no idea why are needed, but the SDK will complain if you don't have them.  

### Step 3: Initialize the SDK & connect to the speaker
Now you need to initialixe the SDK so you can do stuff with the speaker. *(remember `Util.java` has been renamed to `HarmanSDKUtil.java`)*.  
  
    HarmanSDKUtil harmanSDKUtil = HarmanSDKUtil.getInstance()
    if (!harmanSDKUtil.hkwireless.isInitialized()) {
        harmanSDKUtil.hkwireless.initializeHKWirelessController("some key");
        if (harmanSDKUtil.hkwireless.isInitialized()) {
            // init was success!
        } else {
            // init failed :( 
        }
    }

I'm not sure what the key is for, but that's what the method accepts. Other than that, pretty straightforward right?
  
Next, you have to add all the speakers connected to the same network as your device into an internal device list located inside the `HarmanSDKUtil` class. Do that with this line:    
  
	harmanSDKUtil.initDeviceInfor();
  
 > Note that it might not add the devices on the first call work the first time. Your best bet is to have a system set up to where the user can easily retry and call this method again.
 
 Now you can get a list of all of the connected devices (devices that are on the same wifi network) with the `getDevices` method. Like so:    
  
	List<HarmanSDKUtil.DeviceData> devices = harmanSDKUtil.getDevices();
     
The `DeviceData` object doesn't contain much besides an identifier for the device. You probably won't have to mess with it.   
After getting the connected devices, you can add it to the session:  
  
	harmanSDKUtil.addDeviceToSession(device.deviceObj.deviceId);
  
All devices added to the session will be controlled together. (there's also a `removeDeviceFromSession` method)   

### Step 4: Playing some music
In order to start playing music, we need an instance of the `AudioCodecHandler` class:    
  
	AudioCodecHandler pcmCodec = new AudioCodecHandler();
  
And now you can play music like so:  
  
	String songName = songUri.substring(songUri.lastIndexOf("/"));
	pcmCodec.playCAFFromCertainTime(songUri, songName, 0);
  
The `songUri` is a string form of the url to the song on the device. Here's an example:    
  
	String songUri = "/storage/emulated/0/Music/12 Cece's Interlude.mp3";
  
Here's a function that gets the song info and songs from your device:  
  
	public class SongSearch{
		public class Song{
			public String title, artist, uri;
			
			public Song(String title, String artist, String uri){
				this.title = title;
				this.artist = artist;
				this.uri = uri;
			}
		}

		public List<Song> searchSongs(Context context, String term) {
			ArrayList<Song> songList = new ArrayList<>();
			term = term.trim(); // removes the whitespace from the string
			Cursor musicCursor = context.getContentResolver().query(
					MediaStore.Audio.Media.EXTERNAL_CONTENT_URI,
					null,
					MediaStore.Audio.Media.TITLE + " LIKE ? or " + MediaStore.Audio.Media.ARTIST + " LIKE ?",
					new String[]{"%" + term + "%", "%" + term + "%"},
					MediaStore.Audio.Media.TITLE + " ASC");
	
			int titleColumn = musicCursor.getColumnIndex(android.provider.MediaStore.Audio.Media.TITLE);
			int artistColumn = musicCursor.getColumnIndex(android.provider.MediaStore.Audio.Media.ARTIST);
			int uriColumn = musicCursor.getColumnIndex(MediaStore.Audio.Media.DATA);
	
			while (musicCursor.moveToNext()) {
				String uri = musicCursor.getString(uriColumn);
				String title = musicCursor.getString(titleColumn);
				String artist = musicCursor.getString(artistColumn);
				
				songList.add(new Song(uri, title, artist));
			}
			
			return songList;
		}
	}
  
This class has a method that accepts a string and uses it to search the songs on a user's phone. Heres's a rundown of how it works:  
+ gets the content resolver and queries the media on the device in a very sql-esque manner  
+ gets the indexes of the the columns of the information that we want. (check out the [inherited constants](http://developer.android.com/reference/android/provider/MediaStore.Audio.Media.html#inhconstants) of the `MediaStore.Audio.Media` class to find a complete list of the fields that are available to you.)  
+ iterates over the results of the `ContentResolver` query to get the information we want (using the column ids we got earlier), creates a `Song` object with them, and adds it to a list  
+ returns the list of `Song` objects  

So now that we have songs, we can play them just creating a method that accepts the uri:  
  
	public void play(String songUri){
		String songName = songUri.substring(songUri.lastIndexOf("/"));
		pcmCodec.playCAFFromCertainTime(songUri, songName, 0);
	}
  

> **This is extremely important so listen up:*+ only one application can use the SDK at a time. So if you're getting an error saying ` A component of name 'OMX.qcom.audio.decoder.aac' already exists, ignoring this one.`, it's because another app (probably the HKController app) is using the SDK. Kill that app, and your should start working.


Check out the other methods on the `AudioCodecHandler` object! You can pause and stop songs too.

### Step 5: Reacting to Music events
So now that we have the music playing, we need to do stuff when certain things happen. Luckily the SDK has an `HKListener` class. Register it by using:  
  
    harmanSDKUtil.hkwireless.registerHKWirelessControllerListener(new HKWirelessListener() {
            @Override
            public void onDeviceStateUpdated(long l, int i) {
                // 
            }

            @Override
            public void onPlaybackStateChanged(int i) {
				// the song was paused/stopped!!
            }

            @Override
            public void onVolumeLevelChanged(long l, int i, int i1) {
				// the volume changed!
            }

            @Override
            public void onPlayEnded() {
				// the song ended!
            }

            @Override
            public void onPlaybackTimeChanged(int i) {
	            // someone scrubbed through the music!
            }

            @Override
            public void onErrorOccurred(int i, String s) {
	            // on no! Something went wrong :(
            }
        });
  
Pretty straightforward right?

### Conclusion
Make sure to look at the other methods inside of the `HarmanSDKUtil` and other classes in order to see all of the things you can do with the SDK. There definitely many more things you can do with the SDK than mentioned here. 

Definitely check out [Carl Saldanha's tutorial too for a full example](http://cjds.github.io/me/2015/08/31/Harman-Kardon-SDK-For-Android/).

Good Luck and Happy Hacking!