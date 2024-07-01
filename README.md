# CoDWaW: Comprehensive Guide to Sounds and Soundaliases

## Preface
*What are we trying to accomplish?* 

1. We want to play sounds in our WaW map/mod that are not included in the stock game or modtools. 
2. We want to manipulate how we play those sounds in-game. 

*How can we accomplish this?* 

1. We can convert our sounds to a format that can be decoded by WaW’s engine (IW 3.0). 
2. We can create a soundalias file that will allow us to adjust how our sounds play in various ways (volume, pitch, etc.). 

When mapping/modding, it is crucial to consider memory consumption and the engine’s limitations, especially with regards to audio. If we manage our assets poorly, we will quickly find ourselves hitting all kinds of limits. The two limits we are concerned with in this guide are the loaded sounds limit and the overall memory limit. 

> Loaded Sounds Limit = 1600 

- this means we can have no more than 1600 sound (.wav) files contained in our FF files 

> Overall Memory Limit ≈ 315 MB 

- this means that we can have no more than 315 megabytes of total assets loaded at any time 

(a full list of WaW’s limitations can be found [here](https://wiki.zeroy.com/index.php?title=Call_of_Duty_5:_Engine_Limitation))

However, we should never need to worry about hitting either of these limits. There is almost no scenario where we will ever truly need more than 1600 unique sound files in our map/mod, or where we will need to load more than 315 megabytes-worth of assets at once. With proper optimization, we can achieve the highest possible level of detail and quality while avoiding any limits we may encounter. 

**<p align="center">Required Tools** 

[Audacity](https://www.audacityteam.org/download/)

[FFmpeg for Audacity](https://support.audacityteam.org/basics/installing-ffmpeg) (Recommended)



## Part 1: Audio Codecs, Settings, and Format

It is important to remember that every sound is different and will require its own individualized settings to accommodate its distinct timbre and characteristics. Before anything else, we must consider what purpose our sound will serve in-game. 

*Is it music?* 

*Is it ambience? SFX? Dialogue?* 

We need to set audio parameters for each sound according to what that sound actually *is*. First, we must determine what audio codec to use.

### <p align="center">I. Codecs

Audio encoding is the process of converting analog audio signals into a digital format for efficient storage and playback. Encoding is essential in digital audio processing for a variety of purposes, including reducing file size and memory usage, transmitting audio data efficiently, and ensuring compatibility with one or more applications. In general, digitally encoding an audio signal involves an analog waveform being sampled at regular intervals (sampling), and each sample being assigned a specific digital value representing amplitude (quantization). 

![Aspose Words 8bdbfb9a-fa9d-4570-a8c5-3facbca4a0d5](https://github.com/staticsnow04/CoDWaW-Sounds-Soundaliases-Guide/assets/174348593/04c199a5-2502-484d-b161-b4cf7d7583bd)


A **codec** (short for "coder-decoder") is a specific method used to encode audio signals in a digital form. For our purposes we will work with the PCM, ADPCM, and xWMA methods. 

<p align="center">PCM 

PCM stands for Pulse Code Modulation. It encodes audio data as a series of digital values representing each sample’s amplitude. It is the standard method used to digitally encode audio signals and stores raw (uncompressed) data. While this offers lossless audio quality, it also results in large file sizes and high memory usage. For this reason, it is not recommended to use standard PCM for sounds.[ More information here. ](https://wiki.multimedia.cx/index.php/PCM)

<p align="center">ADPCM 

ADPCM stands for Adaptive Differential Pulse Code Modulation. It encodes audio data by calculating the difference between consecutive audio samples rather than the absolute sample values. ADPCM is typically used for telecommunications and is occasionally used for multimedia and gaming applications. [More information here.](https://wiki.multimedia.cx/index.php/Microsoft_ADPCM)

<p align="center">xWMA 

xWMA stands for Xbox Windows Media Audio and is an extension of Microsoft’s WMA (Windows Media Audio) codec. It encodes audio data by determining how perceptible each part of the audio signal is to the human ear and allocating more data for the more perceptible parts and less data for the less perceptible parts. xWMA is specifically designed for use in multimedia and gaming applications on Xbox platforms, but it is also used on Windows platforms in some cases.[ More information here. ](https://wiki.multimedia.cx/index.php/Microsoft_xWMA)

*Which one should we use?* 

This will depend on the purpose of the sound and the characteristics of the sound itself. However, there are some rules of thumb that should make this determination easier. 

- All music, voiceovers, battlechatter, and background ambience should be encoded in ADPCM. This is because these types of sounds should be streamed, and all streamed sounds must be encoded in ADPCM. This is covered in more detail later. 
- All looping sounds should be encoded in ADPCM. The xWMA encoding process tends to add a small segment of silence to the beginning of a sound and cut off a small segment at the end of the sound, which will result in audible pops at the beginning/end of each loop iteration. 
- All weapon sounds (firing, action, reloading) should be encoded in ADPCM. This is because that segment of added silence from the xWMA encoding process (effectively an 85 ms delay) at the beginning of a firing sound can be very noticeable; it will also likely desynchronize reload sounds from their respective animations. 

For other types of sounds, it is up to the user to determine which codec is preferrable. It is advisable to take file size into account when choosing between ADPCM and xWMA. Here is a table displaying the resulting file sizes of three test sounds encoded in ADPCM and in xWMA compared to standard PCM: 

|Sound|PCM Size (KB)|ADPCM Size (KB)|xWMA Size (KB)|
| - | - | - | - |
|Test Sound #1|348|92|31|
|Test Sound #2|1,505|387|99|
|Test Sound #3|4,782|1,226|311|

Dividing PCM Size by Encoded Size will give us the compression ratio: 

|Sound|ADPCM Compression Ratio|xWMA Compression Ratio|
| - | - | - |
|Test Sound #1 |3\.78|11\.23|
|Test Sound #2 |3\.89|15\.20|
|Test Sound #3 |3\.90|15\.38|

We can see that ADPCM maintains a consistent compression ratio of ~3.9 while xWMA maintains a much higher but less consistent compression ratio. This is because, as explained before, ADPCM uses a simple, steady compression method, while xWMA uses a more complex compression method that will adjust for different parts of a sound. 

It is also advisable, however, to take audio quality into account when choosing between ADPCM and xWMA. These two codecs affect sounds very differently. Depending on the sounds’ characteristics, some might sound better when encoded in ADPCM, while others might sound better in xWMA. In cases where the difference in quality is negligible, it is preferrable to use xWMA due to its much smaller file size. 

### <p align="center">II. Settings

Now that we have determined which audio codec to use, we need to prepare our sound to be converted to that codec, which involves choosing appropriate settings and formatting options. To accomplish this, we will be using Audacity, an open-source digital audio editing tool. 

#### <p align="center">Channel Configuration 

First, we must choose our channel configuration. The channel configuration is the arrangement of audio channels within a sound, and it determines how many separate audio tracks or streams are present in the sound. For our purposes, we will work with the Mono and Stereo channel configurations. 

**Mono** - Short for monophonic, meaning audio is only transmitted through one channel (i.e., the center channel) 

**Stereo** - Short for stereophonic, meaning audio is transmitted through two or more channels. For our purposes, we will use no more than two channels (i.e., the left and right). 

*Which one should we use?* 

This will depend on the purpose of the sound and occasionally its characteristics. The general rule is: 

- 3d (spatialized) sounds must be Mono. 
- 2d (non-spatialized) sounds can be either Mono or Stereo. 

There are exceptions to this, however. If a Mono sound contains mostly higher-frequency content, it can suffer some distortion when it is decoded and played back in-game. In this case, the Mono sound can be converted to Stereo, provided that the left and right channels contain identical data, thus simulating a single-channel sound while still containing two channels.\* If the newly converted Stereo sound is a 3d sound, the developer console will send a warning every time the sound is played, but this can be safely ignored. 

[How to Convert Stereo to Mono ](https://www.youtube.com/watch?v=UnPXkW2RzDo)[How to Convert Mono to Stereo ](https://www.youtube.com/watch?v=LJWe8evOebY)

> *Alternatively, a 2d Mono sound can be converted to stereo using pseudo-stereo techniques.[ More information here. ](https://www.csounds.com/journal/issue14/PseudoStereo.html)

It is worth noting that converting Mono to Stereo effectively doubles the audio data contained in the sound and will result in roughly twice the file size and memory usage, so this should only be done when necessary. ![](Aspose.Words.8bdbfb9a-fa9d-4570-a8c5-3facbca4a0d5.002.png)



#### <p align="center">Sample Rate 

Next, we must choose the sample rate of our sound. The sample rate is the number of samples taken per second to capture an audio waveform, measured in Hertz (Hz). A higher sample rate will offer more potential for audio quality. This is particularly true for sounds with more high-frequency content, where a higher sample rate is required to retain the sound’s detail and complexity. For sounds with mostly low to mid-range frequency content, a lower sample rate is sufficient. 

The sample rate we choose will also depend on which channel configuration and codec we are using. As usual, it is important to consider file size when choosing a sample rate. Here is a table displaying variation in file size depending on channel configuration, sample rate, and codec: 

|Channel Config.|Sample Rate (Hz)|PCM Size (KB)|ADPCM Size (KB)|xWMA Size (KB)|
| - | :- | :- | :- | :- |
|Stereo|48000|348|92|31|
|Stereo|41000|319|84|33|
|Stereo|32000|232|62|33|
|Stereo|22050|160|43|33|
|Stereo|16000|116|32|118|
|Mono|48000|174|47|176|
|Mono|41000|160|43|18|
|Mono|32000|116|32|18|
|Mono|22050|80|23|18|
|Mono|16000|58|17|60|

We can see that changing the sample rate of our initial PCM sound has a significant impact on the file size of the ADPCM files but does not have the same impact on the file size of the xWMA files. 

[How to Change Sample Rate](https://www.youtube.com/watch?v=PBaaPY-v3EQ)

<p align="center">Sample Rate for xWMA Sounds

It seems that, when encoding a sound in xWMA, our converter will only accept the sample rates of either 48000 Hz or 44100 Hz for a Stereo sound and only 44100 Hz for a Mono sound. If the converter encounters some other sample rate, it will either change it to 44100 or simply fail to encode the sound. Further, whether a Stereo sound has a 48000 Hz sample rate or 44100 Hz, the impact on file size is negligible (in many cases, file size is smaller with 48000 Hz, strangely enough). 

Taking all of this into account, choosing a sample rate for xWMA sounds is a very simple process. 

- For Stereo sound, the sample rate should be 48000 Hz. 
- For Mono sound, the sample rate should be 44100 Hz. 

<p align="center">Sample Rate for ADPCM Sounds

Unfortunately, choosing a sample rate for ADPCM sounds is not as straightforward. We need to choose a conservative sample rate that will save file size and memory while not decreasing audio quality too much. For most sounds, 32000 Hz is an acceptable sample rate. Sounds with mostly low frequency content, like low ambient tones, can go lower to 22050 Hz or 16000 Hz. Sounds with more high frequency content, like music and weapon sounds, might need a sample rate of 44100 Hz or 48000 Hz. 

The especially tricky part, however, is that sounds with lower sample rates are more susceptible to distortion from the ADPCM encoding process. For example, if we lower a sound’s sample rate to 32000 Hz and it retains acceptable quality in PCM form, it may sound much worse once converted to ADPCM, and we might decide to revert to the original, higher sample rate. 

Keep in mind that lowering the sample rate of a sound, like lowering the resolution of an image, means that some data is irreversibly lost. Keep a backup of any sound if we plan to lower its sample rate in case we need to revert later. 

### <p align="center"> III. Format

Once we have chosen the channel configuration and sample rate of our sound, we need to export it from Audacity in the correct audio format. An **audio format** is the structure and organization of audio data within a file. Audio formats define various parameters, including the encoding method, channel configuration, sample rate, and bit depth (the number of bits allocated for each audio sample). There are many different audio formats, among the most common being the MP3, WAV, AAC, FLAC, and OGG formats. For the converter to encode our sound, it must be in WAV (Waveform Audio File Format). 

Navigate to File > Export > Export as WAV. 

![Aspose Words 8bdbfb9a-fa9d-4570-a8c5-3facbca4a0d5 003](https://github.com/staticsnow04/CoDWaW-Sounds-Soundaliases-Guide/assets/174348593/80eb5a74-a1a2-49b9-8a7e-286fc4777408)

Next, select ‘Signed 16-bit PCM’ as the encoding method, and save. 

![Aspose Words 8bdbfb9a-fa9d-4570-a8c5-3facbca4a0d5 004](https://github.com/staticsnow04/CoDWaW-Sounds-Soundaliases-Guide/assets/174348593/e24042c5-969b-4a02-9222-4852095b36ed)

Our sound is now ready to be converted for use in WaW. 

## Part 2: Sound Conversion 

When converting our PCM sound files to ADPCM or xWMA, we cannot use any third- party audio manipulation tools like FFmpeg or SoX to convert them directly. This is because our sounds need a PRIV (private) chunk: a block of data used to store proprietary information used by WaW’s audio engine. This PRIV chunk can only be created by WaW’s supplied audio conversion tool, MODSound (contained in root/bin). 

By default, MODSound does not encode our sounds in ADPCM or xWMA when we convert them; it simply adds the PRIV chunk to the PCM file. In order to encode our sounds, we will need the conversion kit supplied in the introduction of this guide. This conversion kit is designed to mimic the WaW root folder. 

First, we must place the Conversion\_Kit folder in a directory that has no spaces.

> **Bad Example**: C:\Users\Desktop\Mod Tools\Conversion\_Kit 

> **Good Example**: C:\Users\Desktop\Modtools\Conversion\_Kit 

Next, we must place our uncompressed PCM sound (the one we exported from Audacity) in ‘Conversion\_Kit\XXX\_Encoder\sound\_assets\raw\sound’ according to which codec we wish to encode our sound in. 

Now we need to navigate to ‘Conversion\_Kit\XXX\_Encoder\raw\soundaliases’ and open the sound\_list.csv file in a text editor or spreadsheet program. Within sound\_list.csv, we need to put a name for each sound in the ‘name’ column, as well as the path to each sound file in the ‘file’ column, excluding “sound\_assets\raw\sound.” Examples are provided here: 

![Aspose Words 8bdbfb9a-fa9d-4570-a8c5-3facbca4a0d5 005](https://github.com/staticsnow04/CoDWaW-Sounds-Soundaliases-Guide/assets/174348593/cdedc5cb-9050-45a6-a034-7a51d4811aa2)

Finally, we must navigate to ‘Conversion\_Kit\XXX\_Encoder\bin’ and run convert.bat (convert\_overwrite.bat will overwrite any previously converted sounds). Our newly converted sounds will be located in ‘Conversion\_Kit\XXX\_Encoder\raw\sound.’ 

## Part 3: Soundaliases 

Perhaps the part of WaW’s sound system that most modders initially struggle with is the soundalias system; this is unfortunate, as the soundalias system is one of the most straightforward aspects of the audio engine, as well as an extremely powerful tool for audio manipulation. A **soundalias** is the name by which WaW’s engine and script will know a sound or a group of sounds. Rather than referring directly to a sound file, we must refer to a soundalias whenever we tell the game to play a sound. We define our soundaliases within a CSV (comma- separated values) file located in raw/soundaliases. 

### <p align="center">I. Parameters.

The soundalias system employed by WaW controls how each sound is implemented in- game. *Everything*, bar the character of the sound itself, is controlled by soundaliases: volume, pitch, reverb, spatialization, audio occlusion, audible range, etc. It does this by using the CSV file to define parameters for each column (the “header”) and allowing the user to input settings in each parameter’s corresponding column. An extensive list of valid parameters can be found[ here. ](https://wiki.zeroy.com/index.php?title=Call_of_Duty_5:_Audio_System_Parameters)

#### <p align="center">Variants 

If we wish to have several sound files assigned to a single soundalias, here is what we must do: 

1) Grab our sound files and rename them using the following template: *prefix*\_*##*.wav 

\- where *prefix* is a whatever label we want to give the sound and *##* is a two-digit number. 

So if we had four unique explosion sound files, we could name them like so: 

kaboom\_00.wav kaboom\_01.wav kaboom\_02.wav kaboom\_03.wav 

Each of these sound files is called a **variant**. 

2) Make a new folder named *prefix* and place our sound files in there. 

So for our four explosion sounds, they would be placed in a new folder called “kaboom” 

We will place this folder in our desired path under raw/sound. 

3) In the ‘file’ column of our soundalias, we must put the path to our new **folder**, not to our individual sound files. We must also add “\_##” to the file path, so the engine knows to look for variants with a two-digit suffix within that folder. 

So for our explosion sounds, our input for the ‘file’ column for our soundalias could look like this: 

SFX\explosions\kaboom\_## 

Now, whenever the soundalias is called upon by the engine or script, it will pick a random variant within that folder. 

#### <p align="center">Load Specification 

In order to make sure our compiler loads our soundaliases’ information, we need to define a load specification for our soundaliases. Under the ‘loadspec’ column in our soundalias CSV file, we need to input a code-word (or several code-words) that our compiler will look for. 

Example: 

|name |file |loadspec|
| - | - | - |
|custom\_sound|` `path\to\custom\sound.wav|` `all\_sp |

We also need to include our loadspec code-word in our soundalias line in our zone\_source file. For example, if our soundalias CSV file was named “custom\_soundaliases.csv” and we wanted our compiler to look for soundaliases with a loadspec of “all\_sp,” our zone\_source line would look like this: 

sound,custom\_soundaliases,,all\_sp 

Now the compiler will load the information for all soundaliases with the “all\_sp” loadspec within custom\_soundaliases.csv. 

### <p align="center">II. Packing

Finally, we must pack our sounds into our mod. There are two methods of doing this. Our sounds must be either loaded or streamed.  

**Loaded Sounds** - Sounds that are packed into our FF file. Upon loading up our map/mod, loaded sounds will *always* be loaded and will *always* be taking up memory.  

**Streamed Sounds** - Sounds that are packed into our IWD file. Streamed sounds are only loaded  and will only take up memory when called upon by the engine or script. 

*Which one should we use?* 

This will depend on the codec and purpose of our sounds. The general rules are: 

- All xWMA sounds must be loaded. xWMA sounds **cannot** be streamed. 
- All weapon sounds should be loaded. 
- All SFX should be loaded (footsteps, emitters, explosions, etc.) 
- All music, voiceovers, battlechatter, and background ambience should be streamed. 

To define whether a sound is loaded or streamed, we must input either “loaded” or “streamed” under the ‘type’ column in our soundalias CSV file. 

**<p align="center">References**

1. [https://wiki.zeroy.com/index.php?title=Call_of_Duty_5:_Engine_Limitation ](https://wiki.zeroy.com/index.php?title=Call_of_Duty_5:_Engine_Limitation)
2. [https://wiki.multimedia.cx/index.php/PCM ](https://wiki.multimedia.cx/index.php/PCM) 
3. [https://wiki.multimedia.cx/index.php/Microsoft_ADPCM ](https://wiki.multimedia.cx/index.php/Microsoft_ADPCM)
4. [https://wiki.multimedia.cx/index.php/Microsoft_xWMA ](https://wiki.multimedia.cx/index.php/Microsoft_xWMA)
5. [https://www.csounds.com/journal/issue14/PseudoStereo.html ](https://www.csounds.com/journal/issue14/PseudoStereo.html)
6. https://wiki.zeroy.com/index.php?title=Call\_of\_Duty\_5:\_Audio\_System\_Parameters 
