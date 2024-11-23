DVD Rip Prep (drp) is a bash script that attemps to prepare an MKV file produced
from a DVD by MakeMKV and prepare it for upscaling.  It will probably not be of 
any use with other video files.  It won't do you much good with video extracted
from a Blu Ray or 4K or even with files that were extracted from a DVD but later
edited.

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
are used to take 24 FPS video and make it play at 30 FPS or 25 FPS.  In North
America, we used telecine to make 24 FPS content play like 30 FPS.  In Europe,
since 24 and 25 FPS was so close, they usually just played 24 FPS content at
25 FPS.  The video and audio play about 4% faster, but nobody really noticed.

Telecine is the process of adding 20% more frames to video in order to make it
play at 30 FPS instead of 24 FPS.  This is done by inserting an extra frame in
every set of five, but the extra frame is special.  Telecine takes the fourth
frame and splits it into two interlaced frames.  The original video consisted
of four progressive frames: PPPP.  The telecined version becomes PPPII.  This
accomplishes the job of getting the video to play at 30 FPS, but it introduces
a motion irregularity, because those two interlaced frames are copies of the
same frame.  This wasn't as noticeable on CRTs, but it is easier to spot on
modern screens.  We're so used to it that you might never notice, but there is
a way that you can make it worse and more noticeable and that is by running a
deinterlace on the telecined video.  That takes the PPPII and turns it into
PPPPP.  There are now two full, duplicate frames and your display won't be
aware of it.  The solution is to remove the telecine instead of deinterlacing.
But, before you can do that, you need to know if your video is telecined.

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
tends to be soft telecined.  It is those years in between that are a real
problem.  From around 1980 to 2005, video was often shot on one format and
edited on another.  The original bits and the edited bits were then spliced
back together.  Often, this will result in combinations of soft telecine
and hard telecine or even full interlacing.  This is really hard to deal
with automatically.  If you inverse telecine soft telecined frames, then you
drop 20% of the frames.  If you deinterlace telecined content, then you make
the interlaced frames progressive and turn the original 24 FPS video into
30 FPS with a stutter.  So, when the techniques are mixed in a video, if you
apply a single method to reverse them, you will fix some of the video but 
break the rest.

So, the first function of drp is to look at MKVs and decide if they are soft
telecined, hard telecined, interlaced, or some horrifying combination of 
these techniques (which is, unfortunately, common).  Originally, this was my
only goal for the script.  I wanted something that would analyze the videos
and give me a report so that I could then apply the appropriate techniques to
recover fully progressive video for upscaling.  As these things tend to go, I
then wondered if I could make the script actually do all of the work and fix
the video.

As it stands, drp can do these things pretty well:

* Convert 30FPS soft telecined video to 24FPS progressive video
* Convert 30FPS hard telecined video to 24FPS progressive video
* Convert 30FPS interlaced video to 30FPS progressive video
* Convert 25FPS PAL (interlaced or progressive) to 24FPS progressive video

These conversions should give you a file with a consistent frame rate and no
introduced stutters that can be processed in TVAI without resorting to an 
interlaced method.
