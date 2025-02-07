# DVD Rip Prep

DVD Rip Prep (drp) is a bash script that attempts to prepare a MakeMKV DVD rip for upscaling.  It might also work with rips made with other software, but I have only tested MakeMKV rips.  It will probably not be of any use with other video files.  It only accepts NTSC or PAL SD video and that video ideally should not have been edited or reencoded since being ripped from DVD.  It is possible that it will work with other SD sources like digitized VHS or Laserdisc video, but I don't have any to try.

Before you read further, this is a hobby project that I made for my own needs.  I am not a video professional and this is as much a learning project for me as anything.  I don't really know what I am doing.  The script is just built around ffmpeg and I am probably not even making full use of its capabilities.  So, keep your expectations minimal and perhaps you will be pleasantly surprised.

There are other well established tools for doing this kind of work, but they weren't a good solution for me because they run on Windows and generally work best with rips that were not made by MakeMKV.  I run Linux and I have thousands of MakeMKV rips and I really didn't want to start over.  I also wanted a tool that was automatic, as much as possible.  There is no question that I could get better results doing everything manually, but I only have so much time to spend.  My goal was to get a good result, but not necessarily the best possible result, with the lowest possible amount of effort per video.

## Updates

The Jan 26, 2025 verson finally accomplishes all of the goals that I originally had for this project.  It is able to reliably separate, diagnose, and unify telecined and interlaced video in any arrangement that I have tested.  That's not to say that I think it is perfect (far from it) or that I won't come up with additional ideas, but it does what I wanted it to do.  I solved a bunch of small problems in getting the right segment boundaries.  The mapper can now detect changes between hard telecined and interlaced sections, which was the last remaining big problem.  The unfortunate consequence of all this success has been slower performance.  This script now often takes longer to process a video than it would take to upscale it.  I don't really see this as a big problem, since it is designed to work unattended, but I would like to find ways to speed it up in the future.

I have changed my philosophy behind the default settings.  I previously used defaults that resulted in the fastest processing, but I now use defaults that give the best result.  So there is a lot of room to make things go faster if you are OK with a result that is closer to 99% right than 99.9% right.

Previous versions benefitted a lot more from tinkering with the settings.  I have now arrived at a very good set of defaults, so I no longer recommend changing settings like ALLOWLOSS, RATELOCK, MINSEG, or SEGSIZE unless you have read the script and really understand what you are doing.

Because this project is now so complex and because I have made so many changes, many of them hastily implemented, there are probably things that I broke and just haven't discovered yet.  The script has gotten rather bloated and many of the settings are no longer as important, so it might be time to rewrite and simplify a lot of this.

The format of the segment map file has changed.  It now contains frame numbers instead of times, as I learned that ffmpeg deals much more accurately with frames than timestamps.

The default size of the memory filesystem has changed to 16GB because it was running out of space in some cases.  This is now a max size rather than absolute.  If the file that you are processing is smaller, the script will only use as much space as necessary.

Now that I have something that works pretty well, the next big step might be to port it to a more capable language, like Python.  I have never written anything in Python, but I'll probably give it a shot.

## Background

To understand why this exists, you need to know a few things about video, DVDs, and upscaling.  Until recently, most movies and TV shows were shot on film, at a frame rate of 24 FPS (actually ~23.976025 or 24000/1001).  The rate at which TV screens refresh is tied to the cycle rate of the power that they use.  In North America, this was 30 FPS (~29.97003 or 30000/1001) and, in Europe, it was 25 FPS.  The North American standard is known as NTSC and the European standard is PAL.  There are other standards as well.  Screens are now somewhat more flexible than in the days of tube TVs, but we still live in a world where the standards were set in the past.  The problem here is that the rate at which the video was recorded and the rate at which is is displayed are different.  Several methods are used to make 24 FPS video play at 30 FPS or 25 FPS.  In North America, we used telecine to make 24 FPS content play like 30 FPS.  In Europe, since 24 and 25 FPS are so close, they usually just played 24 FPS content at 25 FPS. The video and audio play about 4% faster, but the difference isn't very noticeable.

### Telecine

Telecine is the process of adding 20% more frames to video in order to make it play at 30 FPS instead of 24 FPS.  This is done by inserting an extra frame after every set of four, but the extra frame is special.  Telecine takes the fourth frame and splits it into two interlaced frames.  The original video consisted of four progressive frames: PPPP.  The telecined version becomes PPP(II).  This accomplishes the job of getting the video to play at 30 FPS, but it introduces a motion irregularity, because those two interlaced frames are copies of the same image.  This wasn't as noticeable on CRTs, but it is easier to spot on modern screens.  We're so used to it that you might never notice, but there is a way that you can make it worse and more noticeable and that is by running a deinterlace on the telecined video.  That takes the PPP(II) and turns it into PPP(PP).  There are now two full, duplicate frames and your display won't be aware of it.  The solution is to remove the telecine instead of deinterlacing. But, before you can do that, you need to know if your video is telecined.

There are also two forms of telecine: soft and hard.  In soft telecine, the video is actually stored as progressive content on the DVD, but the player is instructed to create and insert the interlaced frames.  If you know this, then you can just extract the progressive frames and move ahead.  In hard telecine, the video is stored in telecined format, so the interlaced frames actually exist on the disc and the player just reads them off like anything else.  To remove hard telecine, you need to recombine those interlaced pairs, hopefully without altering anything else.  This process is known as inverse telecine or, sometimes, detelecine.

Another variation on telecine that I have been surprised to see in professionally produced DVDs is improperly deinterlaced hard telecine.  These will consist of progressive frames at 30 FPS, with two duplicate frames in every five.  As discussed above, this results in a baked-in motion irregularity.  It is possible to correct this, but probably not always with a result that is as good as inverse telecine on the original video would have been.

### Interlacing

Video can also be fully interlaced.  While telecine uses interlacing, it only interlaces 25% of the original frames.  Full interlacing is a signature of content that was either shot or edited on video.  Filmed content is almost always telecined.  Content that was recorded on a video camera is generally interlaced.  True interlacing means that every frame of video is spread across two interlaced fields.  Depending on how the video was recorded, those two fields might contain either two partial copies of the same image or two partial copies of different images.  Deinterlacers can deal with this in two ways: by combining the two fields into a single frame, which results in a frame rate of 30 FPS, which is appropriate when the fields are derived from a single image, or by extrapolating the missing lines in each field, resulting in a doubled frame rate of 60 FPS, which is appropriate when the fields were derived from separate images.  In most cases, the second case is the safer option.

You will find these techniques used in different places.  Almost all North American DVDs of movies are either soft or hard telecined and that's it.  European DVDs of movies are usually just progressive content played at the slightly faster 25 FPS.  You can just slow it down to 24 FPS and the job is done.  TV is where it gets complicated.  TV made before the 1980s was usually shot and edited on film and is either soft or hard telecined on disc.  TV made after about 2005 seems most often to be soft telecined, but fully interlaced is also not unusual.  It is those years in between that are a real problem.  From around 1980 to 2005, video was often shot on one format and edited on another.  The original bits and the edited bits were then spliced back together.  Often, this will result in combinations of soft telecine and hard telecine or even full interlacing.  This is really hard to deal with automatically.  If you inverse telecine soft telecined frames, then you drop 20% of the frames.  If you deinterlace telecined content, then you make the interlaced frames progressive and turn the original 24 FPS video into 30 FPS with a stutter.  When the techniques are mixed in a video, if you apply a single method to reverse them, you will fix some of the video but break the rest.

When telecine is removed from filmed content, the frame rate is restored to 24 FPS.  When fully interlaced content is deinterlaced, then the frame rate is either 30 or 60 FPS.  This is no problem when only one of these techniques is used, but, when they are combined, there is no way to get to uniformly progressive frames without either increasing the frame rate of the 24 FPS material or dropping frames from the 30/60 FPS material.  After all, by removing telecine, we have removed the original solution to this rate mismatch.  There is no single solution to this problem.  In many cases, videos consist of something like 95% telecined content and 5% interlaced.  In such cases, the interlacing tends to be in the titles and credits and you might never notice if you dropped 20% of the frames from those sections.

In other cases, the 30 FPS sections might be special effects and dropping frames would be really noticeable.  In that case, the only solution is to increase the overall frame rate of the video to something that can accommodate both rates.  The closest frame rate that is wholly divisible by 24, 30, and 60 is 120.  So, a video that mixes 24 FPS filmed content and 30 FPS video content can be increased to 120 FPS by quintupling the 24 FPS frames and quadrupling the 30 FPS frames.  This maintains the original cadence of all content, but it requires a 120Hz display to play it.  Not every display is capable of running at 120Hz, although it is increasingly becoming the standard.

When changing frame rates, the script does not perform any interpolation, so there should be no apparent difference in the video.  24FPS frames are simply displayed five times and 30FPS frames are displayed four times.  The video looks the same.  When frame rate is increased with interpolation, which is something that Video AI can do, new intermediate frames are created and the effect is to smooth out motion, sometimes referred to as the "soap opera effect."  It is also important to understand that the kind of frame rate multiplication used by the script does not increase the file size.  There are two ways to increase the frame rate, resulting in either constant or varible frame rate.  Constant frame rate involves inserting more copies of each frame into the video and increases the size correspondingly.  Variable frame rate just repeats a given frame as many times as necessary to achieve the specified rate, so the file does not need to be larger.  Although these two methods look the same when the video is played, it is an important difference when the video is upscaled, because the upscaler will need to process every frame that is actually in the video.  It would take 4-5 times as long to upscale a constant frame rate video at 120FPS as it would to upscale the same video with a variable frame rate.  Video AI will actually give you a time prediction as if the video had a constant frame rate, but you will find that it completes the upscale in a third or less of the predicted time.

Writing this script has been quite an education and, if I had known what I know now in the beginning, I might have dropped the idea.  The ways that these techniques are used on TV shows, even within a given season of a given show, is surprisingly inconsistent and, in some cases, looks outright crazy.  There are, of course, a lot of shows that use just one technique or use a mix but at least in a predictable pattern.  But there are also a surprisingly large number of shows that go from one technique to another from episode to episode and that mix them in ways that seem quite strange.  Some use mixes in a way that almost seems intentionally designed to defeat any future efforts to disentagle them.  Animation, in particular, seems to make use of really irregular combinations of telecine and interlacing that are hard to resolve.  I have been able to figure out solutions for a lot of shows, but there are some that may never be possible to process automatically without more sophisticated tools than I am using here.

### Upscaling

Most people who are interested in this script are probably intending to upscale video with Topaz Video AI or other software, which is the reason why I wrote it. TVAI can deal with the easy cases, as long as you know what to tell it what to do.  TVAI does not recognize soft telecined content, which you can see when you load the video.  It will see the original frame rate as 29.97 and will output the same.  So, you are better off re-encoding soft telecined videos before importing to TVAI.  TVAI can perform inverse telecine for hard telecined video using the setting found under Input/Edit, but this will wreck anything that isn't actually hard telecined.  And TVAI can deinterlace using the very same function that I use in the script, but the problem is that there are very few instances of movies or TV on DVD that are actually interlaced all the way through.  And Topaz has nothing that will deal with the problem of mixed methods in the video.  I use Linux and, therefore, an older version of TVAI.  It is always possible that the newer versions have new solutions that I am unaware of.

If you tell TVAI to do the wrong things, your results generally won't be as good as they could be.  You might not notice the problems introduced by misapplications of deinterlacing because TVAI output often looks so much better that it is easy to overlook the small problems that it can introduce.  I had upscaled a lot of videos, which took a lot of time, before I started to notice stutters and artifacts related to improper deinterlacing.  I then spent even more time fiddling around with TVAI settings to no great success because the real problem was that I was feeding it defective video.

So, the first function of drp is to look at MKVs and decide if they are soft telecined, hard telecined, interlaced, or some horrifying combination of these techniques (which is, unfortunately, common).  Originally, this was my only goal for the script.  I wanted something that would analyze the videos and give me a report so that I could then apply the appropriate techniques to recover fully progressive video for upscaling.  As these things tend to go, I then wondered if I could make the script actually do all of the work and fix the video.  In a lot of cases, now it can.

## Features

### Analysis

The most basic feature of the script is to diagnose what kind of frame structure a DVD rip has.  You can use the script in dry run mode, which will just output a report in the same directory that you run it from that lists the diagnosis for each file.  To use this feature, just change to a directory containing your MKV files and run drp -d.

The script will output two files: diagreport.txt and maplist.txt.  The diagreport.txt file lists the diagnosis for each file.  The maplist.txt file contains a list of files that use mixed methods and need further analysis.  If AUTOMAP is enabled, then the script will continue to analyze and produce segment maps for each of these files.

#### Segment Mapping

If you are lucky, most of the content that you want to upscale will either use a single mode of frame rate manipulation or it will only make brief use of a secondary method that can be overlooked, but, if your video was made in the 80s, 90s, or early 2000s, there is a good chance that it uses mixed methods.  To fix such a video, the script needs to break it up into segments and then work on each segment.  The script uses a segment map file to do this work.

It is possible to manually create a segment map, which is just a list of start and end frames for each segment, but the script now does this automatically.  Drp uses an accessory script, smap, to do the mapping.  If you have AUTOMAP enabled, which is now the default, then the whole process takes place automatically.  If AUTOMAP is disabled, then you need to run the smap script on each file yourself (smap file).

Smap divides the file into approximately 1/2 second segments and analyzes each of those to determine where frame rate changes occur in the file.  It then outputs this data as a segment map file.  Once the segment map file has been created, you can run drp normally and it will use the map file.

Automatic mapping is a slow process that generates a lot of small files and can use a lot of disk space.  I'm not sure at what point all these writes become a wear and tear problem for drives, but it seems like a good idea to avoid them if possible, so the smap script has an option to create and use a memory filesystem.  This is controlled by the USERAMDISK setting at the top of the smap script.  If you have AUTOMAP enabled in drp and USERAMDISK enabled in smap, then you should run drp with sudo.  Smap will set up and tear down the memory filesystem with every run and it needs root permissions to do that.  Without using sudo, it will prompt for authentication on each run and will wait for you to come back and type your password.

### Conversions

The script is best at converting single method videos.  It can do the following conversions pretty reliably:

* 30FPS soft telecined video to 24FPS progressive video  
* 30FPS hard telecined video to 24FPS progressive video  
* 30FPS improperly deinterlaced hard telecined video to 24FPS progressive video  
* 30FPS interlaced video to 30 or 60FPS progressive video  
* 25FPS PAL (interlaced or progressive) to 24 or 48FPS progressive video  

These conversions should give you a file with a consistent frame rate, and no introduced stutters, that can be processed in TVAI without resorting to an interlaced method.  It is worth noting again that the aim of this script is not perfection.  Except for soft telecined and progressive content, it will probably never produce a result that does not contain a single interlaced frame.  They are going to continue to lurk in the transitions and in sequences of fast motion where ffmpeg cannot find a good match, but the occasional interlaced frame is usually going to go unnoticed in casual viewing and it is not worth the extra effort that would be required to eliminate them.

By default, the script outputs files in the FFV1 format, which is lossless.  That means that the file sizes can be pretty large, usually at least five times the size of the MKV you started with.  Keep that in mind when batch processing.  The scripts do not currently check available disk space, although I should probably add that.

Mixed method conversion is still a work in progress.  It works most of the time, but there are some videos that have such a large number of transitions that there is no way to get a good result.  This is a very slow process - the more segments, the longer it takes.  Between mapping and converting the segments, it is not unusual for the script to take three or four hours per hour of video.  The more segments, the longer it will take, because ffmpeg has to read from the beginning to the segment start every time.

The script now creates a temp directory named 00DRP that contains log files and subdirectories for the output of all the intermediate steps.  If you find that a video didn't complete, check the log files and you are bound to discover that one of the segments needs adjustment.

By default, the final output of the script will be both a "rejoined" MKV that contains only the video and a "merged" MKV that add the source MKV file back in.  You should use the rejoined file for upscaling.  The merged file is primarily there so that you can check it and make sure that the audio still syncs and you can safely delete it.  You can also configure the script to output unjoined segments, so that you can upscale them separately and rejoin them yourself, if you prefer.  This might be a good idea with video that includes computer generated special effect sequences.

#### PAL Conversion

As far as I can tell, any content that you find on a PAL format DVD that was originally shot on film is just sped up from 24 to 25 FPS.  Drp will convert this by slowing the video back down to 24 FPS.  It will also output an audio file that is slowed to the same rate so that you can ensure that the audio still syncs up.  Assuming that the DVD contains multiple audio and subtitle tracks, you will need to slow those down to match the video.  I recommend doing this with MKVtoolnix using the stretch option.  Just enter 1.042708333 (25/(24000/1001)) for the stretch value for each track.  If you don't want PAL conversion, set CONVPAL to 0 in the drp script.

Converted interlaced PAL video will be output at two frame rates: 24FPS and 48FPS.  This is because there is no easy way to determine whether the source was originally produced on video with different interlaced fields or if it was filmed and the interlaced fields produced by splitting frames.  In general, material that was shot on film will look best at 24FPS and material that was shot on video will look best at 48FPS.  You should keep the frame rate that looks best to you.

I plan to improve PAL conversion in the future so that all audio and subtitles are converted automatically.  I just don't have a lot of PAL DVDs, so it hasn't been a high priority.

## Installation

The script requires a system that runs a bash shell (Linux/Unix/Mac or Windows with the Linux subsystem) and ffmpeg, ffprobe, and mkvtoolnix must be installed.  I have only tested it on Ubuntu Linux, so there might well be minor compatibility problems with other distros or systems.  I will do my best to fix problems that crop up on other platforms, but my experience with them is limited.  This could probably be ported to run natively on Windows without too much trouble, since the tools that it uses are all available in Windows native versions, but I haven't run Windows in 20 years, so I am probably not the one to do it.

The two scripts, drp and smap, are just bash scripts and can be installed anywhere that is convenient.  You might just keep them in your home directory.  If you want them to be in path, then the /usr/local/bin directory wouldn't be a bad place to put them.

You can run them through bash, like 'bash drp' or 'bash smap', but if you want to run them directly as commands, then you will need to make them executable:

> chmod +x drp  
> chmod +x smap  

## Usage

| Command | Description |
| --- | --- |
| **drp** | Batch mode - it will try to convert every MKV file in the current directory. |
| **drp -a** | Animation mode - experimental and doesn't do anything useful yet. |
| **drp -d** | Dry run mode - it will try to diagnose every MKV file and then exit. |
| **drp -i** file | Single file mode - it will diagnose and/or convert the named file only. |
| **drp -o** directory | Override drp OUTDIR. |
| **drp -p** PAL or NTSC | Override video standard.  Can be useful when video has been altered after ripping. |
| **drp -s** | Enable STRICT mode. |
| **drp -r** directory | Resume a job that was stopped.  Use the numerical directory name. |
| **smap** file | Single file mode - it will create a segment map for the named file. |

There are settings at the top of each script file that you can change.  They are explained in the comments.

As of Dec 7, the script now creates a unique temp directory with each run, so it is possible to run more than one process of the script concurrently.  The only limits to the number of processes you can run simultaneously are processor power and memory (each process that uses the mapper will consume 12GB).

## Settings

The settings are found at the top of each of the script files.

### drp Settings

#### ALLOWLOSS

The more invasive the method used by the script, the more likely that the runtime of the video will change.  Usually, it ends up slightly shorter due to dropped frames, but occasionally it will also run slightly longer, so this setting really refers to runtime change, not just loss.  You can't change the number of frames and the frame rate and have the duration come out the same every time.  In most cases, the difference is small enough that it does not matter, but a big enough change will result in audio that is out of sync.  Videos that change by more than ALLOWLOSS will be renamed to indicate the time difference.  Also, segments that change by more than this amount may be retried with an alternate method to see if it produces a better result.  This is a percentage, so the default value allows a change of 2.5 seconds per hour of video.  That might seem like a lot, but that much difference is only ever likely to result on short segments, not on the full runtime of the video.  Default is 0.07.

#### AUTOMAP

If enabled (1), runs the segment mapper in batch mode.  If disabled (0), you will need to run smap on mixed files manually.  Default is enabled.  Disable if you prefer to run smap and drp separately.  When enabled, you should run drp with sudo so that it can set up and tear down the memory filesystem.

#### CONVPAL

If you feed the script PAL video, it will slow it from 25 to 24 FPS, hopefully restoring the original runtime and audio pitch.  The script will output an audio MKV with the default track stretched 4% to match up with the slower frame rate.  You will need to add this track to the final MKV after you upscale.  If you have alternate audio or sub tracks that need to be matched to the slower frame rate, you can use the strech function in MKVToolnix with a value of 1.042708289.

Set to 1 to enable PAL conversion or 0 to disable it.  Default is enabled (1).

#### DEBUG and DEBUGSLOW

These are primarily for my use, but you might find them helpful for troubleshooting on occasion.  These mostly just report entering and leaving functions and the current values of variables.  DEBUGSLOW adds a wait at the end of loops.  Disabled (0) by default.

#### FROUTS

Most of the time, when using the mixed method, the script will output at either 60 or 120FPS.  You might want to try other frame rates.  For example, if a video is mostly 24FPS but has some higher frame rate material at the beginning or end, you might want to try it at 24FPS.  You can set FROUTS to a space delimited list of rates that you want to be output.  For example, if you want to try every major frame rate, use FROUTS=(24 30 48 60 120).  The default is (24 30 60 120), as 48 is only useful in relatively rare cases.  You can also set this to OFF and the script will output only the frame rate that is the lowest common denominator.

#### HTCTHRESH

Sets the percentage threshhold above which a video is considered to be hard telecined.  The default setting is 2.5.  Settings between 1 and 25 are probably reasonable and might give good results with some sources.

#### INTLTHRESH

Sets the percentage threshhold above which a video is condidered to be fully interlaced.  The default setting is 50.  Settings between 40 and 80 are probably reasonable and might give good results with some sources.

#### KEEPTEMP

Enable (1) this setting to stop the script from deleting temp files at the end of the run.  This can be helpful when troubleshooting, but you will have to remember to clean up the temp files, since they consume a lot of disk space.  Default is disabled (0).

#### MAPPER

Specify the location of the segment mapper script.  The default is the smap script that comes with drp, but you could potentially modify your own mapper to meet different requirements.  All that it needs to do is accept a file name as input and output a segment map file.  Default is "$(dirname "$0")/smap", which is a file named smap in the same directory as drp.

#### MERGEMKV

By default, the script outputs a file containing only the video track, as this seems to work better with TVAI sometimes.  If you enable merging, then drp will also output a copy that is merged with the original MKV, which allows you to compare the two video tracks and make sure that the audio syncs up.  OUTPUTJOIN must be enabled (1) for this to work.  Default is enabled (1).

#### OUTDIR

This is the directory that you want the script to use for its output and working directories.  There is no longer a default setting, due to the complexities of sometimes needing to run the script with sudo.  Instead, if you have not set the directory, the script will prompt you and exit.

OUTDIR can also be overridden with the command line switch -o.  So, "-o /home/johndoe", for example, will use the specified directory for output.

#### OUTPUTJOIN

If enabled, the script will output a rejoined output file when the mixed process was used (the input file is divided into segments for processing and then those segments are rejoined into a single MKV file for output).  This can be used in combination with OUTPUTSEGS.  Default is enabled (1)

#### OUTPUTSEGS

If enabled, the script will output the processed segments when the mixed process was used.  You might want to output the segments if you would like to upscale them separately.  This can be used in combination with OUTPUTJOIN.  Default is disabled (0).

#### RATELOCK

RATELOCK determines how far the frame rate can diverge from the nominal rate and still be treated by the script as having that nominal frame rate.  The default is 0.0001, which translates, for example, into a film rate range of ~23.974 to ~23.978.  Videos that fall outside this range will be converted with the mixed process, which is very slow.  The default is very conservative.  If you don't care about dropping frames from bumpers, titles, and credit sequences, then you could increase this to 0.01 and still get good results in most cases.

#### STRICT

STRICT mode will send all videos that match 30FPS, even within the RATELOCK allowance, through the mixed method.  The script cannot reliably determine whether a 30FPS video consists of a mix of telecined and interlaced content without running it through the mapper.  If you want to make sure that interlaced and hard telecined segments get processed separately, leave this enabled (1), which is the default.  If you want faster processing and aren't particular, disable it.  Most of the time, the script will appropriately process mixed interlaced and telecined content, but when it gets really chopped up and there is a mixture of TFF and BFF material, it is better to separate segments.  The only downside to STRICT is that the script is much slower.

#### WRITELOGS

Set to 1 to enable logging or 0 to disable it.  Logs are written to 00DRP/LOGS.  Default is enabled (1).

### smap Settings

#### DEBUG and DEBUGSLOW

These are primarily for my use, but you might find them helpful for troubleshooting on occasion.  These mostly just report entering and leaving functions and the current values of variables.  DEBUGSLOW adds a wait at the end of loops.  Disabled (0) by default.

#### KEEPRAMDISK

If enabled (1), then the memory filesystem will not be unmounted at the end of the run.  This is helpful if you are troubleshooting and want to look at the working files.  Default is disabled (0).

#### MINSEG

Segments shorter than MINSEG will be ignored.  As with SEGSIZE, I no longer recommend changing this setting.  You are more likely to make problems than solve them.  Default is 12.

#### OUTDIR

This is the directory that you want the script to use for its output and working directories.  Default is your home directory (~).

#### RAMDISKSIZE

This is the maximum size of the memory filesystem.  The script will calculate the right size for the video you are working on, so the RAM disk will not exceed this size, but it might be smaller.  If the working file exceeds this size, then it will be processed on the regular filesystem instead.  Default value is 16 (GB), which will generally work for runtimes up to 90 minutes.  If you have lots of RAM and longer videos, increase this setting.

#### SEGSIZE

How finely should the script slice the video for analysis?  The default time of 0.1 effectively instructs ffmpeg to make the smallest possible segments.  I no longer recommend changing this setting.  You are more likely to make problems than solve them if you change this value.

#### USERAMDISK

If enabled (1), which is now the default, the script will create a RAMDISKSIZE tmpfs memory filesystem for the mapper to use.  By default, this will consume up to 16GB of RAM for the duration that the script runs.  Because this script writes out thousands of files per analysis, using a memory filesystem seems like a good idea.  Writing lots of small files over and over might accelerate disk wear.  If you have less than 24GB of RAM, the RAMDISK will be automatically disabled, regardless of the setting.

If the script crashes or you break out of it, you will need to manually unmount the filesystem, which you can do with the following command:

sudo umount ~/00DRP/*/segments

(This command assumes that you have not changed OUTDIR.)
