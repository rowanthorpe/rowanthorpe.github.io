# Firefox without PulseAudio in Debian

### TL;DR

Download and run [my automation script](https://github.com/rowanthorpe/build-jack-fox) if you want to compile
your own Firefox to run under Jack instead of PulseAudio on Debian (or a derivative distro like Ubuntu), but don't
have the experience or free-time to navigate the process yourself. I considered making a pre-compiled binary
available whenever I compile it for myself, but would only do so if the builds are reproducible, and at least a
couple of other commenters compile and verify the binary's hash each time (you should *never* trust a random
binary from a random website, without at least some kind of independent consensus). Also, I don't want to risk any
legal issues because I accidentally compiled in something I shouldn't have without some disclaimers, etc. The best
solution would be if Debian maintainers would just add `--enable-jack` to the default build, but I suspect they
would be reluctant to do so when upstream don't. Of course the usual disclaimer applies - use at your own risk,
don't hold me responsible if it does something you don't expect, etc. It is a relatively short script so you
might be able to understand most of what it does by glimpsing at it before you run it. Anyway it is just building
`.deb`s with an added flag and added dependency, so it shouldn't really be able to do much damage beyond what the
standard package does anyway. To use the script edit the top few lines to your liking, run it with the `--help`
flag to see options, then run it. You may need to interact with it a few times (password if needed for sudo, etc)
but the overwhelming majority of the time is spent non-interactively compiling Firefox so you can abandon post
for a few hours once that is running. The firefox source-package is retrieved based on the `deb-src` lines in
your `/etc/apt/sources.list...` file(s), so be sure you have those in place - see the `apt-get` manpage (under
the `source` sub-command) for details.

### The long version

I spent a few years typing 8-bit assembler into an old Amstrad (and saving it to cassettes!) in the mid-80s, and
later intermittently shoulder-surfed while my friend hacked undocumented "X-modes" and did crazy stuff in Pascal
on his 386, but then other priorities/career took over and computers effectively disappeared from my life until
the end of the 90s. Then in '99 I bought a used Pentium laptop from a colleague (I was constantly touring
professionally - the only option was a laptop), inserted a free [Redhat Linux](https://www.redhat.com) cover-CD
from a computing magazine, the install crashed (X couldn't handle the proprietary graphics card), and I spent
every spare moment I had during the next 6 months getting to the point of having a functioning desktop (on my own,
having never used a commandline before, with only sporadic access to dialup internet). Yes, that was painful, and
I'm sure there could have been better approaches, but I wouldn't trade that time for anything. This may seem
irrelevant but I'll refer back to it below.

    "When you lose, don’t lose the lesson."
     - Dalai Lama

Fast-forward through a few years of touring (and falling asleep while teaching myself at my laptop in hotel-rooms
when colleagues were out at bars), and once I'd stopped travelling and had a consistent home and
free-time in '04, I used [Gentoo Linux](https://www.gentoo.org) for all my weirder operating-system and kernel
experiments, and a few forays with [Beyond Linux From Scratch](http://www.linuxfromscratch.org/blfs), but
even then - and especially now - I use(d) [Debian GNU/Linux](https://www.debian.org) for all my "real-life" work
because I never had personal access to a compile-cluster (or much free-time) for constant re-tweaking. These days
I *only* have time for "real-life" stuff so I effectively spend 100% of my time in Debian. As I strive to be an
"optimalist" ([with](http://www.escapingthe9to5.com/optimalism/optimalism)
[software](http://bighealthyme.com/perfectionist-to-optimalist)
[in particular](https://www.leadershipnow.com/leadingblog/2009/04/are_you_a_perfectionist_or_an.html)), I have
ended up with a pretty unique system-configuration on my laptop - "Frankendesktop" would be a good term for it.
This is built up out of deliberate decisions over many years, often involving knowingly treading the road less
travelled. The inevitable side-effect of choosing to be an outlier instead of embedding myself in the herd is
being first to feel the brunt of (get T-boned by) new non-backwards-compatible executive-decisions, especially
the low-level ones. A good recent example is when I had to yield to the slow-but-relentless zombie-style invasion
of everything and anything by [systemd](https://freedesktop.org/wiki/Software/systemd), as it does its best to
turn GNU/Linux into a Windows-wannabe. Despite its list of opaque monoculture-assumptions - and inevitable
crashes on unusual systems like mine - I did begrudgingly give in because I found myself losing too much time
fitting square-pegs into round-holes which developers and maintainers were no longer willing to do, but I did so
with the note-to-self that as soon as is feasible I will sidestep onto [Devuan](https://devuan.org), while
waiting for [GuixSD](https://www.gnu.org/software/guix) to reach full prime-time stability. I remember at the
time I lost a couple of evenings shuffling everything, and one of the real killers was
[PulseAudio](https://www.freedesktop.org/wiki/Software/PulseAudio) (also a flagship project of the head developer
of systemd, hmm). At the time it was *impossible* to avoid PulseAudio (and a bunch of other things) when running
under systemd, so I begrudgingly installed them too. From the moment PulseAudio was on my system I was seeing
jitter, freezes, and even the occasional crash - always as obscure as hell, impossible to reliably reproduce and
hopelessly undiagnosable. About a year(?) later I found I was able to shuffle things back enough to remove
PulseAudio, while only losing one or two obscure packages with hard-deps which I almost never used anyway. The
moment PA was off my system stability returned to normal (i.e. "solid as a rock"), except for the few times when
systemd decided to overwrite my customizations on upgrade, even the customizations I had properly entered in
`/usr/local`, linking from `/etc` and so on [sigh]. I call that kind of software "bully code", and still believe
that the worst way to deal with a bully is to be an "enabler", but then again you also have to choose your battles
when you want to have time for anything else in your life. I should add at this point that instabilities from PA
and systemd were the extremely rare anomalies despite my updating my systems against Debian Sid (`unstable`) and
even including some packages from `experimental`. The key is actually reading the reportbug output and
occasionally holding off updating something until a "grave" bug is fixed.

The only other moment of instability was a brief one very recently when I had to temporarily reinstall PulseAudio
to test something. At that same time I had also decided to return to [Firefox](https://www.mozilla.org/firefox)
after a few years of using [Chromium](https://www.chromium.org) for performance and stability reasons, enticed
back by all the "massive rewrite, new engine" hype around Firefox 57. I was pleasantly surprised, and tentatively,
progressively migrated back to it. Then last week I had three random desktop-crashes three days in a row, losing
edits I was working on all three times, so after relaxing and regaining enough of my sanity to think straight, I
drilled into system logs to diagnose. It was bizarrely obscure, but after nearly a day (yes, a *day*) I isolated
that it was (drumroll...) PulseAudio, which I had forgotten to re-uninstall. I uninstalled it and then discovered
Firefox had no sound because Mozilla had decided back in version (52?) to deprecate the direct alsa-driver, and
later to remove it entirely. After having had such a nice-but-brief born-again Firefox honeymoon, I was really
annoyed to think I would have to rewind back to Chromium *again*, so I scoured the message-boards and
issue-trackers for answers. That was soul-destroying stuff. So many angry, impotent (yet justified) power-users,
and so many angry and too-overwhelmed-to-please-everyone (yet justified) developers - butting heads in so many
echo-chambers. One positive takeaway at least was that I understood the developers had decided to abandon direct
Alsa (in-kernel) support and just support the simpler API of the PulseAudio sound-*server*, so they could
dedicate their efforts to stuff which feels less futile than effectively maintaining their own in-app
sound-server. Anyway, after the X-th hour of scrolling through people's rants I discovered
[this blog-post](http://www.zamaudio.com/?p=1580) from an external developer who had contributed support for the
[Jack](http://www.jackaudio.org/) sound-*server* but Mozilla didn't want to compile it in by default. I had
interacted with Jack a few times when using various dependent sound-editing tools for soundtracks for
performances back in the day, and had found it to be totally solid even then, and that I'd only ever scratched
the surface of its capabilities (it is a "pro-tool"). I installed jackd and build-deps for Firefox, downloaded
the source-package for Firefox and tried compiling it with custom flags. After a few false starts: **[A]**
Firefox no longer accepts `--disable-pulseaudio` even if you enable an alternative, and **[B]** you have to
install `libjack-jackd2-dev` (or `libjack-dev` for Jack v1) in order for the Firefox code to include
`jack/jack.h`, and after waiting about 2 hours for the final compile to finish, it worked. I've been listening
to [Nine Inch Nails live on Youtube](https://www.youtube.com/watch?v=LBC3NXnN8y4) while typing this, without any
glitches, and more importantly without any crashes, and no PulseAudio running.

I know this doesn't solve the problem for people who want Firefox without an external sound-server at all because
e.g. they are trying to squeeze it onto a device like a phone where every kB counts, but it is worth considering
that part of the Firefox devs' complaints seem to be that maintaining support for Alsa means they end up
internalizing quite a bit of "sound-server" functionality into the browser instead, so it is worth considering
how much you are actually gaining in the end these days, even if you manage to avoid an external sound-server.
Also, I found it an interesting object-lesson seeing so many people (not the ones just mentioned) complaining
about being forced into installing PulseAudio (but who weren't particularly averse to the idea of a sound-server
per se, or of compiling their own browser occasionally), that they were too preoccupied with that frustration and
yearning for the status quo to find out that there is a perfectly reasonable alternative (which then also means
they have all the power-tools of Jack at their disposal for more than just the browser). This brings me back to
the seemingly irrelevant bit of ancient history I mentioned at the top of this post: it plugs in nicely to a
quote which I've had as my email-signature for a long time, and which things like the proprietary-GPU-laden
laptop of '99 taught me:

    "There is a great difference between worry and concern. A worried
     person sees a problem, and a concerned person solves a problem."
     - Harold Stephens

I know most users won't have all of **[A]** the patience, **[B]** the frustration-fuelled-motivation, and **[C]**
the experience with compiling/configuration, so they won't go through what I did to get a working solution without
PulseAudio. For this reason I wrote a script to automate the build-process. Be warned, the compile took 1.5
hours on my powerful quad-core laptop, so expect it to take about that long, or even up to 4 hours if on a
slow/loaded computer.

**NB:** I haven't set this up as a proper blog yet, so I apologise for the lack of a commenting system, etc. Email
me (find my email address via my github profile @rowanthorpe) if you feel strongly about something, and I will add
notes here (with attribution) if in your email you give me permission to do so.

[Go back to main page]({% link index.md %}).
