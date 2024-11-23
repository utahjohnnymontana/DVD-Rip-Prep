DVD Rip Prep (drp) is a bash script that attemps to prepare an MKV file produced
from a DVD by MakeMKV and prepare it for upscaling.  It will probably not be of 
any use with other video files.  It won't do you much good with video extracted
from a Blu Ray or 4K or even with files that were extracted from a DVD but later
edited.

Before you read further, this is a hobby project.  I am not a video professional
and this is as much a learning project for me as anything.  I don't really know
what I am doing.  The script is just built around ffmpeg and I am probably not 
even making full use of its capabilities.

Background

To understand why this exists, you need to know a few things about video, DVDs,
and upscaling.  Until recently, most Movies and TV shows were shot on film, at 
a frame rate of 24 FPS (actually 23.976025 or 24000/1001).  The rate at which TV
screens refresh is tied to the cycle rate of the power they use.  In North
America, this was 30 FPS (29.97003 or 30000/1001) and in Europe it was 25 FPS.
The North American standard is known as NTSC and the European standard is PAL.
There are other standards as well.  Screens are now somewhat more flexible than
in the days of tube TVs, but we still live in a world where the standards were
set in the past.  The problem here is that the rate at which the video was
recorded and the rate at which is is displayed are different.  Several methods
are used to make 24 FPS video play at 30 FPS or 25 FPS.  In North America, we 
used telecine to make 24 FPS content play like 30 FPS.  In Europe, since 24 
and 25 FPS are so close, they usually just played 24 FPS content at 25 FPS.  
The video and audio play about 4% faster, but nobody really noticed.

Telecine is the process of adding 20% more frames to video in order to make it
play at 30 FPS instead of 24 FPS.  This is done by inserting an extra frame 
after every set of four, but the extra frame is special.  Telecine takes the 
fourth frame and splits it into two interlaced frames.  The original video 
consisted of four progressive frames: PPPP.  The telecined version becomes 
PPP(II).  This accomplishes the job of getting the video to play at 30 FPS, but 
it introduces a motion irregularity, because those two interlaced frames are 
copies of the same image.  This wasn't as noticeable on CRTs, but it is easier 
to spot on modern screens.  We're so used to it that you might never notice, 
but there is a way that you can make it worse and more noticeable and that is 
by running a deinterlace on the telecined video.  That takes the PPP(II) and 
turns it into PPP(PP).  There are now two full, duplicate frames and your 
display won't be aware of it.  The solution is to remove the telecine instead 
of deinterlacing. But, before you can do that, you need to know if your video 
is telecined.

There are also two forms of telecine: soft and hard.  In soft telecine, the
video is actually stored as progressive content on the DVD, but the player is
intructed to add in the interlaced frames.  If you know this, you can just
extract the progressive frames and move ahead.  In hard telecine, the video
is stored in telecined format, so the interlaced frames actually exist on the
disk and the player just reads them off like anything else.  To remove hard
telecine, you need to recombine those interlaced pairs, hopefully without
altering anything else.  This process is known as inverse telecine.

You will find these techniques used in different places.  Almost all North
American DVDs of movies are either soft or hard telecined and that's it.
European DVDs of movies are usually just progressive content played at the
slightly faster 25 FPS.  You can just slow it down to 24 FPS and the job is
done.  TV is where it gets complicated.  TV made before the 1980s tends to
be just like film, either soft or hard telecined.  TV made after about 2005
is usually soft telecined.  It is those years in between that are a real
problem.  From around 1980 to 2005, video was often shot on one format and
edited on another.  The original bits and the edited bits were then spliced
back together.  Often, this will result in combinations of soft telecine
and hard telecine or even full interlacing.  This is really hard to deal
with automatically.  If you inverse telecine soft telecined frames, then you
drop 20% of the frames.  If you deinterlace telecined content, then you make
the interlaced frames progressive and turn the original 24 FPS video into
30 FPS with a stutter.  When the techniques are mixed in a video, if you
apply a single method to reverse them, you will fix some of the video but 
break the rest.

Most people who are interested in this script are probably intending to
upscale video with Topaz Video AI, which is the reason why I wrote it. TVAI 
can deal with the easy cases itself, as long as you know how to tell it 
what to do.  TVAI does not recognize soft telecined content, which you can 
see when you load the video.  It will see the original frame rate as 29.97 
and will output the same.  So, you are better off reencoding soft telecined 
videos before importing to TVAI.  TVAI can perform inverse telecine for 
hard telecined video using the setting found under Input/Edit, but this will 
wreck anything that isn't actually hard telecined.  And TVAI can deinterlace 
using the very same function that I use in the script, but the problem is 
that there are very few instances of movies or TV on DVD that are actually 
interlaced.  And Topaz has nothing that will deal with the problem of mixed 
methods in the video.  You might not notice the problems introduced by 
misapplications of deinterlacing.  TVAI output often looks so much better 
that it is easy to overlook the small problems that it can introduce.  I had 
upscaled hundreds of videos before I started to notice stutters and artifacts 
related to improper deinterlacing.  I spent a lot of time fiddling around with 
TVAI settings to no great success because the problem was that I was feeding 
it defective video.

So, the first function of drp is to look at MKVs and decide if they are soft
telecined, hard telecined, interlaced, or some horrifying combination of 
these techniques (which is, unfortunately, common).  Originally, this was my
only goal for the script.  I wanted something that would analyze the videos
and give me a report so that I could then apply the appropriate techniques to
recover fully progressive video for upscaling.  As these things tend to go, I
then wondered if I could make the script actually do all of the work and fix
the video.

Features

The most basic feature of the script is to diagnose what kind of structure
a DVD rip has.  You can use the script in dry run mode, which will just
output a report in the same directory that you run it from that lists the
diagnosis for each file.  To use this feature, just change to a directory
containing your MKV files and run drp -d.

The script can also handle progressive conversions for all but the mixed
methods case.  It can do the following pretty reliably:

* Convert 30FPS soft telecined video to 24FPS progressive video  
* Convert 30FPS hard telecined video to 24FPS progressive video  
* Convert 30FPS interlaced video to 30FPS progressive video  
* Convert 25FPS PAL (interlaced or progressive) to 24FPS progressive video  

These conversions should give you a file with a consistent frame rate, and no
introduced stutters, that can be processed in TVAI without resorting to an 
interlaced method.

There is another feature that is still in progress, but works in many cases:

* Convert mixed video to 24FPS progressive video using a user created
  segment map.

The previous version of the script had a one pass technique for mixed content 
and sometimes it even worked, but it was destructive and often resulted in 
stutters.  I have discontinued that method.

I am working on several approaches for an automatic solution, but I don't 
even know all of the right questions yet, much less the answers.  Because of 
that, I have introduced a new process that requires a human to find the 
segments in the video.  That allows me to move forward without solving the 
entire problem first.

So, if you have a mixed content video, the script will now look for a segment 
map file, which is just a series of times in a text file.  If you have a file 
named my-video.mkv, you will create a file named my-video.times in the same 
directory.  Scan through the video and look for places where titles, credits, 
or computer graphics were added.  Get the start and end times for each such 
segment and add them to the times file in this format:

start,00:02:00  
00:02:00,00:04:00  
00:04:00,00:48:00  
00:48:00,end  

This is an example of a very common pattern in TV shows that have overlayed 
titles and credits but no CGI effects.  This would look like a two minute 
intro, followed by two minutes of overlayed titles and major credits, followed 
by the 44 minute body of the episode, followed by the end credits.  You have 
to account for the full run of the video - no gaps.  Start and end are special 
times, since the script can get the first and last frame number automatically.  
You should try to get as close as possible to the time where the segment 
changes.  If you are lucky, your videos will only have a few segments, but I 
have tested the script successfully with as many as sixteen.

If you have identified the segments correctly, the script will apply the 
appropriate technqiue to each segment and output a rejoined video that still 
syncs up with the audio.  If you got it wrong, then the audio will be out of 
sync.  That is a sign that you either need to tighten up the segment times or 
look for an additional segment (maybe a special effects sequence) that you 
missed.

If this sounds like too much work, I don't blame you.  This is still 
primitive and slow.  It is a lot less work than trying to do this all manually 
though.  If you have a good eye for finding the seams in your videos, then you 
can do a lot with it.  If enough people use this, I can imagine them swapping 
their times files so that others don't have to start from scratch.  Of course,
maybe I will find an automatic solution soon enough to make that unnecessary.  
Only time will tell.

The script now creates a temp directory named 00DRP that contains log files 
and subdirectories for the output of all the intermediate steps.  If you find 
that a video didn't complete, check the log files and you are bound to discover 
that one of the segments needs adjustment.

By default, the final output of the script will be both a "rejoined" MKV that 
contains only the video and a "merged" MKV that add the source MKV file back in.  
You should use the rejoined file for your TVAI upscaling.  The merged file is 
primarily there so that you can check it and make sure that the audio still 
syncs.

You can run:
drp					  | batch mode  
drp -d				| dry run batch mode  
drp file			| single file  
drp -d file		| single file dry run  

There are settings at the top of the script file that you can change.  They 
are explained in the comments.

The script requires a system that runs a bash shell (linux/unix/mac or Windows 
with the linux subsystem) and ffmpeg, mediainfo, and mkvtoolnix must be 
installed.

This is way more complicated than it was originally, has overall poor error 
handling, and not much in the way of sanity checking, so it is pretty fragile.  
If you give it anything unexpected, it is likely to break in a fairly ugly 
fashion.  Don't expect too much.  Any kind of feedback is welcome.  I will do 
my best to fix problems, but this is just a little hobby project for me, so it 
might not happen that fast.
