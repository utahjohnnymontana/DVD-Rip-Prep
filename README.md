# DVD Rip Prep

DVD Rip Prep (drp) is a bash script that attemps to prepare an MKV file produced from a DVD by MakeMKV for upscaling.  It might also work with rips made with software, but I have only tested MakeMKV rips.  It will probably not be of any use with other video files.  It won't do you much good with video extracted from a Blu Ray or 4K or even with files that were extracted from a DVD but later edited.

Before you read further, this is a hobby project.  I am not a video professional and this is as much a learning project for me as anything.  I don't really know what I am doing.  The script is just built around ffmpeg and I am probably not even making full use of its capabilities.  So, keep your expectations minimal and perhaps you will be pleasantly surprised.

## Background

To understand why this exists, you need to know a few things about video, DVDs, and upscaling.  Until recently, most movies and TV shows were shot on film, at a frame rate of 24 FPS (actually 23.976025 or 24000/1001).  The rate at which TV screens refresh is tied to the cycle rate of the power they use.  In North America, this was 30 FPS (29.97003 or 30000/1001) and in Europe it was 25 FPS.  The North American standard is known as NTSC and the European standard is PAL.  There are other standards as well.  Screens are now somewhat more flexible than in the days of tube TVs, but we still live in a world where the standards were set in the past.  The problem here is that the rate at which the video was recorded and the rate at which is is displayed are different.  Several methods are used to make 24 FPS video play at 30 FPS or 25 FPS.  In North America, we used telecine to make 24 FPS content play like 30 FPS.  In Europe, since 24 and 25 FPS are so close, they usually just played 24 FPS content at 25 FPS. The video and audio play about 4% faster, but nobody really noticed.

### Telecine

Telecine is the process of adding 20% more frames to video in order to make it play at 30 FPS instead of 24 FPS.  This is done by inserting an extra frame after every set of four, but the extra frame is special.  Telecine takes the fourth frame and splits it into two interlaced frames.  The original video consisted of four progressive frames: PPPP.  The telecined version becomes PPP(II).  This accomplishes the job of getting the video to play at 30 FPS, but it introduces a motion irregularity, because those two interlaced frames are copies of the same image.  This wasn't as noticeable on CRTs, but it is easier to spot on modern screens.  We're so used to it that you might never notice, but there is a way that you can make it worse and more noticeable and that is by running a deinterlace on the telecined video.  That takes the PPP(II) and turns it into PPP(PP).  There are now two full, duplicate frames and your display won't be aware of it.  The solution is to remove the telecine instead 
of deinterlacing. But, before you can do that, you need to know if your video is telecined.

There are also two forms of telecine: soft and hard.  In soft telecine, the video is actually stored as progressive content on the DVD, but the player is intructed to add in the interlaced frames.  If you know this, you can just extract the progressive frames and move ahead.  In hard telecine, the video is stored in telecined format, so the interlaced frames actually exist on the disk and the player just reads them off like anything else.  To remove hard telecine, you need to recombine those interlaced pairs, hopefully without altering anything else.  This process is known as inverse telecine.

### Interlacing

Video can also be fully interlaced.  While telecine uses interlacing, it only interlaces 25% of the original frames.  Full interlacing is a signature of content that was either shot or edited on video.  Filmed content is almost always telecined.  Content that was recorded on a video camera is generally interlaced.  True interlacing means that every frame of video is spread across two interlaced fields.  Deinterlacers can deal with this in two ways: by combining the two fields into a single frame, which preserves the 30 FPS frame rate, or by extrapolating the missing lines in each field, resulting in a doubled frame rate of 60 FPS.

You will find these techniques used in different places.  Almost all North American DVDs of movies are either soft or hard telecined and that's it.  European DVDs of movies are usually just progressive content played at the slightly faster 25 FPS.  You can just slow it down to 24 FPS and the job is done.  TV is where it gets complicated.  TV made before the 1980s tends to be just like film, either soft or hard telecined.  TV made after about 2005 is usually soft telecined.  It is those years in between that are a real problem.  From around 1980 to 2005, video was often shot on one format and edited on another.  The original bits and the edited bits were then spliced back together.  Often, this will result in combinations of soft telecine and hard telecine or even full interlacing.  This is really hard to deal with automatically.  If you inverse telecine soft telecined frames, then you drop 20% of the frames.  If you deinterlace telecined content, then you make the interlaced frames progressive and turn the original 24 FPS video into 30 FPS with a stutter.  When the techniques are mixed in a video, if you apply a single method to reverse them, you will fix some of the video but break the rest.

When telecine is removed from filmed content, the frame rate is restored to 24 FPS.  When fully interlaced content is deinterlaced, then the frame rate is either 30 or 60 FPS.  This is no problem when only one of these techniques is used, but when they are combined, there is no way to get to uniformly progressive frames without either increasing the frame rate of the 24 FPS material or dropping frames from the 30/60 FPS material.  After all, by removing telecine, we have removed the original solution to this rate mismatch.  There is no single solution to this problem.  In many cases, videos consist of something like 95% telecined content and 5% interlaced.  In such cases, the interlacing tends to be in the titles and credits and you might never notice if you dropped 20% of the frames from those sections.  In other cases, the 30 FPS sections might be special effects and dropping frames would be really noticeable.  In that case, the only solution is to increase the overall frame rate of the video to something that can accomodate both rates.  The closest frame rate that is wholly divisible by 24, 30, and 60 is 120.  So, a video that mixes 24 FPS filmed content and 30 FPS video content can be increased to 120 FPS by quintupling the 24 FPS frames and quadrupling the 30 FPS frames.  This maintains the original cadence of all content, although at the cost of greater file size.

### Upscaling

Most people who are interested in this script are probably intending to upscale video with Topaz Video AI or other software, which is the reason why I wrote it. TVAI can deal with the easy cases itself, as long as you know how to tell it what to do.  TVAI does not recognize soft telecined content, which you can see when you load the video.  It will see the original frame rate as 29.97 and will output the same.  So, you are better off reencoding soft telecined videos before importing to TVAI.  TVAI can perform inverse telecine for hard telecined video using the setting found under Input/Edit, but this will wreck anything that isn't actually hard telecined.  And TVAI can deinterlace using the very same function that I use in the script, but the problem is that there are very few instances of movies or TV on DVD that are actually interlaced.  And Topaz has nothing that will deal with the problem of mixed methods in the video.  You might not notice the problems introduced by misapplications of deinterlacing.  TVAI output often looks so much better that it is easy to overlook the small problems that it can introduce.  I had upscaled hundreds of videos before I started to notice stutters and artifacts related to improper deinterlacing.  I spent a lot of time fiddling around with TVAI settings to no great success because the problem was that I was feeding it defective video.

So, the first function of drp is to look at MKVs and decide if they are soft telecined, hard telecined, interlaced, or some horrifying combination of these techniques (which is, unfortunately, common).  Originally, this was my only goal for the script.  I wanted something that would analyze the videos and give me a report so that I could then apply the appropriate techniques to recover fully progressive video for upscaling.  As these things tend to go, I then wondered if I could make the script actually do all of the work and fix
the video.

## Features

### Analysis

The most basic feature of the script is to diagnose what kind of structure a DVD rip has.  You can use the script in dry run mode, which will just output a report in the same directory that you run it from that lists the diagnosis for each file.  To use this feature, just change to a directory containing your MKV files and run drp -d.

The script will output two files: diagreport.txt and maplist.txt.  The diagreport.txt file lists the dignosis for each file.  The maplist.txt file contains a list of files that use mixed methods and need further analysis.  If AUTOMAP is enabled, then the script will continue to analyse and produce segment maps for each of these files.

#### Segment Mapping

If you are lucky, most of the content that you want to upscale will either use a single mode of frame rate manipulation or it will only make brief use of a secondary method that can be overlooked.

But, if your video was made in the 80s, 90s, or early 2000s, there is a good chance that it uses mixed methods.  To fix such a video, the script needs to break it up into segments by frame rate and then work on each segment.  The script uses a segment map file to do this work.

There are two ways to produce a segment map file.  The manual method is to look at the video yourself, identify the places where there appears to be a change between original filmed content and edited content, and make your own map file.  The automatic method uses another script, smap, to try to find the frame rate changes.

##### Manual Mapping

If you have a mixed content video, the script will now look for a segment map file, which is just a series of times in a text file.  If you have a file named my-video.mkv, you will create a file named my-video.smap in the same directory.  Scan through the video and look for places where titles, credits, or computer graphics were added.  Get the start and end times for each such segment and add them to the times file in this format:

start,00:02:00  
00:02:00,00:04:00  
00:04:00,00:48:00  
00:48:00,end  

This is an example of a very common pattern in TV shows that have overlayed titles and credits but no CGI effects.  This would look like a two minute intro, followed by two minutes of overlayed titles and major credits, followed by the 44 minute body of the episode, followed by the end credits.  You have to account for the full run of the video - no gaps.  Start and end are special times, since the script can get the first and last frame number automatically.  You should try to get as close as possible to the time where the segment 
changes.  If you are lucky, your videos will only have a few segments, but I have tested the script successfully with as many as thirty three.

If you have identified the segments correctly, the script will apply the appropriate technqiue to each segment and output a rejoined video that still syncs up with the audio.  If you got it wrong, then the audio will be out of sync.  That is a sign that you either need to tighten up the segment times or look for an additional segment (maybe a special effects sequence) that you missed.

##### Automatic Mapping

It is now also possible for the script to automatically generate segment maps.  This is an experimental procedure and it doesn't always work well.  If you have videos with relatively simple structures, like the example above in the Manual Mapping section, then it will probably be faster to make manual maps than to do it automatically.  However, if your video has many small segments, the automatic mapper is likely to do the job faster.

Drp uses an accessory script, smap, to do the mapping.  If you have AUTOMAP enabled, then the whole process takes place automatically.  If AUTOMAP is disabled, which is the default setting, then you need to run the smap script on each file yourself (smap file).

Smap divides the file into hundreds or thousands of short segments and analyzes each of those to determine where frame rate changes occur in the file.  It then outputs this data as a segment map file.  Once the segment map file has been created, you can run drp normally and it will use the map file.

Automatic mapping is a slow process that generates a lot of small files and can use a lot of disk space.  I'm not sure at what point all these writes become a wear and tear problem for drives, but it seems like a good idea to avoid them if possible, so the smap script has an option to create and use a memory filesystem.  This is controlled by the USERAMDISK setting at the top of the smap script.  It is disabled by default, but if you have at least 16GB of RAM, you might want to enable it.

### Conversions

The script is best at converting single method videos.  It can do the following pretty reliably:

* Convert 30FPS soft telecined video to 24FPS progressive video  
* Convert 30FPS hard telecined video to 24FPS progressive video  
* Convert 30FPS interlaced video to 30FPS progressive video  
* Convert 25FPS PAL (interlaced or progressive) to 24FPS progressive video  

These conversions should give you a file with a consistent frame rate, and no introduced stutters, that can be processed in TVAI without resorting to an interlaced method.

Mixed method conversion is still a work in progress.  It works a lot of the time, but it doesn't work at all for some combinations.  This is a very slow process - the more segments, the longer it takes.  Figure that the processing time for a video with more than 10 segments will be at least twice the runtime of the video, so two hours for every hour of video.  The more segments, the longer it will take, because ffmpeg has to read from the beginning to the segment start every time.

The script now creates a temp directory named 00DRP that contains log files and subdirectories for the output of all the intermediate steps.  If you find that a video didn't complete, check the log files and you are bound to discover that one of the segments needs adjustment.

By default, the final output of the script will be both a "rejoined" MKV that contains only the video and a "merged" MKV that add the source MKV file back in.  You should use the rejoined file for your TVAI upscaling.  The merged file is primarily there so that you can check it and make sure that the audio still syncs.

#### PAL Conversion

As far as I can tell, any content that you find on a PAL format DVD that was originally shot on film is just sped up from 24 to 25 FPS.  Drp will convert this by slowing the video back down to 24 FPS.  It will also output an audio file that is slowed to the same rate so that you can ensure that the audio still syncs up.  Assuming that the DVD contains multiple audio and subtitle tracks, you will need to slow those down to match the video.  I recommend doing this with MKVtoolnix using the stretch option.  Just enter 1.042708289 for the stretch value for each track.  If you don't want PAL conversion, set CONVPAL to 0 in the drp script.

### Installation

The script requires a system that runs a bash shell (linux/unix/mac or Windows with the linux subsystem) and ffmpeg, mediainfo, and mkvtoolnix must be installed.  I have only tested it on Ubuntu Linux, so there might well be minor compatibility problems with other distros or systems.

The two scripts, drp and smap, are just bash scripts and can be installed anywhere that is convenient.  You might just keep them in your home directory.  If you want them to be in path, then the /usr/local/bin directory wouldn't be a bad place to put them.

You can run them through bash, like 'bash drp' or 'bash smap', but if you want to run them directly as commands, then you will need to make them executable:

'chmod +x drp'  
'chmod +x smap'  

### Usage

| Command | Description |
| --- | --- |
| drp | Batch mode - it will try to convert every MKV file in the current directory. |
| drp -d | Dry run batch mode - it will try to diagnose every MKV file and then exit. |
| drp file | Single file mode - it will convert the named file only. |
| drp -d file | Dry run single file - it will diagnose only the named file. |
| smap file | Single file mode - it will create a segment map for the named file. |

There are settings at the top of each script file that you can change.  They are explained in the comments.


