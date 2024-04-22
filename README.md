# Method for Borges Typing

This is a high level overview of the transcription method used by Borges Typing.

This a heuristic copy of my field notes from work with Whisper CPP.

## Introduction
[Whisper](https://en.wikipedia.org/wiki/Whisper_(speech_recognition_system)) is
the sleeping tiger from [OpenAI](https://en.wikipedia.org/wiki/OpenAI).  While
[generative AI](https://en.wikipedia.org/wiki/Generative_artificial_intelligence)
chatbots using [LLMs](https://en.wikipedia.org/wiki/Large_language_model) have
taken the attention of the general public by storm during the [AI boom](https://en.wikipedia.org/wiki/AI_boom),
the [Python](https://en.wikipedia.org/wiki/Python_(programming_language))-based
Whisper has a lot of potential to help with speech-to-text tasks.

[`whisper.cpp`](https://github.com/ggerganov/whisper.cpp) is a [C++](https://en.wikipedia.org/wiki/C%2B%2B)
reimplementation of Whisper, which we will refer to as Whisper CPP throughout
this piece.  However, it is currently only available as a
[CLI](https://en.wikipedia.org/wiki/Command-line_interface) tool.  So, finding a
way to let Whisper CPP, even [under the hood](https://en.wiktionary.org/wiki/under_the_hood),
be used more widely would be a [common good](https://en.wikipedia.org/wiki/Common_good).

## Background
Just for reference, here are my research sources that I have used to guide my
decisions for future project work.

* [Moxie Marlinspike](https://en.wikipedia.org/wiki/Moxie_Marlinspike)
    * "End to End Encryption for Everyone" talk on [YouTube](https://www.youtube.com/watch?v=tOMiAeRwpPA) from [Next Generation Threats 2014](https://techworld.event.idg.se/event/ngt-stockholm-2014/)
        * A dress rehearsal for the next talk below in 2015
    * ["Making private communication simple"](https://vimeo.com/124887048) talk from [Webstock](https://en.wikipedia.org/wiki/Webstock) 2015
    * ["The ecosystem is moving"](https://media.ccc.de/v/36c3-11086-the_ecosystem_is_moving) talk from [36C3](https://en.wikipedia.org/wiki/Chaos_Communication_Congress) in 2019
* [Andrew "bunnie" Huang](https://en.wikipedia.org/wiki/Andrew_Huang_(hacker))
    * "Supply Chain Security: 'If I were a Nation State...'" [talk](https://www.bunniestudios.com/blog/2019/supply-chain-security-talk/) in February 2019 (informally, part 1 of 2)
    * "Open Source is Insufficient to Solve Trust Problems in Hardware" [talk](https://www.bunniestudios.com/blog/2019/can-we-build-trustable-hardware/) in December 2019 (informally, part 2 of 2)
        * 36C3 [mirror](https://media.ccc.de/v/36c3-10690-open_source_is_insufficient_to_solve_trust_problems_in_hardware)
* "Qubes OS for Organizational Security Auditing" talk by [Harlo Holmes](https://freedom.press/people/harlo-holmes/) at [HOPE](https://en.wikipedia.org/wiki/Hackers_on_Planet_Earth) 2020
    * Video available on [Internet Archive](https://archive.org/details/hopeconf2020/20200730_1600_QubesOS_for_Organizational_Security_Auditing.mp4) or [YouTube](https://www.youtube.com/watch?v=WHaPTGZReOU)
* ["The Insecurity Industry"](https://edwardsnowden.substack.com/p/ns-oh-god-how-is-this-legal) by [Edward Snowden](https://en.wikipedia.org/wiki/Edward_Snowden), after the [Pegasus Investigation](https://en.wikipedia.org/wiki/Pegasus_Project_(investigation)) into [NSO Group](https://en.wikipedia.org/wiki/NSO_Group)

Here are the main takeaways:

* From Marlinspike, I learned about [usable security](https://en.wikipedia.org/wiki/Usable_security) through [Signal Messenger](https://en.wikipedia.org/wiki/Signal_(messaging_app));
* From Huang, I learned a sustainable open source technology project, from a human effort stance, should be able to be sustained with less than 10 people and not a giant corporation through [Precursor](https://www.crowdsupply.com/sutajio-kosagi/precursor) and [Betrusted](https://betrusted.io/);
* From Holmes, I learned that small organizations also need to pay attention to cybersecurity beyond 2020; and 
* From Snowden, you're completely correct, because of the next point below.

The call to use Rust in new software projects is prescient, given that a White
House [press release](https://www.whitehouse.gov/oncd/briefing-room/2024/02/26/press-release-technical-report/)
from February 2024 named Rust as an example of a memory-safe language in its
[full report](https://www.whitehouse.gov/wp-content/uploads/2024/02/Final-ONCD-Technical-Report.pdf).

I know someone is going to read this and think something among the lines of:
this isn't new information.  However, I warn you to not misconstrue this with a
"My job here is done/But you didn't do anything"
[meme](https://knowyourmeme.com/memes/my-job-here-is-done-but-you-didnt-do-anything)
situation.

### Examples of prior transcription projects using Whisper
These are a few other projects using Whisper (both Python or CPP) in an
accessible way:

* [Stage Whisper](https://github.com/Stage-Whisper/Stage-Whisper)
    * Last mentioned in a 2022 [article](https://www.theverge.com/2022/9/23/23367296/openai-whisper-transcription-speech-recognition-open-source) from _The Verge_
* [MacWhisper](https://goodsnooze.gumroad.com/l/macwhisper), a proprietary MacOS client primarily for Apple Silicon
    * Well, that is what happens when you use the [MIT License](https://goodsnooze.gumroad.com/l/macwhisper)
    * C.f. [Minix](https://en.wikipedia.org/wiki/Minix) and [Intel ME](https://en.wikipedia.org/wiki/Intel_Management_Engine)
* [Vibe](https://github.com/thewh1teagle/vibe), built with Tauri for desktop (Windows, Mac, & Linux)
* [Transcribro](https://github.com/soupslurpr/Transcribro) an Android app distributed through [Accrescent](https://accrescent.app/)
* [OTranscribe](https://otranscribe.com/opensource/), open-source web app

### Examples of existing non-free online transcription services
* [Trint](https://trint.com/), AI-based transcription
* [Rev](https://www.rev.com/)

TL;DR: You trust these services to not keep copies of your audio forever.

## Motivation
There are three broad reasons why I am interested in working with Whisper CPP.

First, I genuinely have trouble discerning what's being said aloud in relatively
modern multimedia (since 2015).  I really resonated with this Vox
[article](https://www.vox.com/videos/23564218/subtitles-sound-downmixing-dialogue-movies-tv)
from January 2023 when its [video](https://www.youtube.com/watch?v=VYJtb2YXae8)
companion was released on YouTube.  However, though I'm not very fluent in
Spanish, somehow I can hear every word enunciated in Spanish music?  (Does
someone want to take me to a [Bad Bunny](https://en.wikipedia.org/wiki/Bad_Bunny)
concert to test this hypothesis?  Psychologist really need to be studying this.)
Joking aside, I would like to increase accessibility in technology overall - and
if I cast my stone into the collective ocean of knowledge to get that effort
started, then I'll be happy.

Second, I do a quite a bit of work with Whisper CPP.  I use it to help me write
meeting minutes, write transcripts for podcasts, or subtitle the occasional
video.  I'd like to share how I use Whisper CPP, since the most I've seen
documented so far is when it's malfunctioning very badly in its GitHub issues.

Lastly, I feel that there must be some way to work on AI in an ethical manner.
Even though I'm not even making commits back to Whisper CPP, I have a
[Hilbert](https://en.wikipedia.org/wiki/David_Hilbert)-like amount of confidence
that working on AI can be done ethically.  Just like how money is ultimately a
tool, AI is also a tool that ultimately is a giant force multiplier of whatever
existing values, virtues, or traits the user already has.  It is up to the user
whether AI will be used in ways that are detrimental or beneficial to society.
We have to forge the beneficial tools in the
[open](https://en.wikipedia.org/wiki/Free_and_open-source_software), because
the general AI tools of dubious nature have already been long in development
before 2020. [Clearview AI](https://en.wikipedia.org/wiki/Clearview_AI) and the
NSA's internal voice recognition tools detailed by The Intercept in
[2018](https://theintercept.com/2018/01/19/voice-recognition-technology-nsa/)
are only two examples that could've been used.

## Purpose
The purpose is to develop a usable GUI for Whisper CPP that anyone, with no
knowledge of the command line, can use.  Many people who I have used Whisper CPP
with don't have the technical knowledge to rebuild the organizational supports
I have built thanks to Whisper CPP - so, basically the [bus factor](https://en.wikipedia.org/wiki/Bus_factor)
is in effect.  Creating a Whisper CPP GUI would help mitigate bus factor
concerns, if not effectively eliminate it once a sustaining collective effort
behind reaches critical mass.

Eventually explicit and actionable requirements will be listed, but basically
the GUI would satisfy these guiding design requirements:
* be written in Rust,
* be cross-platform on desktop (Windows, macOS, and Linux)
    * [APT](https://en.wikipedia.org/wiki/APT_(software))-, [RPM](https://en.wikipedia.org/wiki/RPM_Package_Manager)-, and [Pacman](https://en.wikipedia.org/wiki/Arch_Linux#Pacman)-based distros will be supported when the "first" `1.0.0` version is released
* only make internet connection to download Whisper CPP's LLMs,
* shows in-progress transcription lines while execution is in progress, and
* saves the output in: plain text, SRT, VTT, and `asciinema` cast.

The GUI is not a live text editor: users are free to choose any application to
edit the resulting files.  (However, for your own sanity, don't choose a rich
text editor like Microsoft Word or LibreOffice Write if you have to
significantly rework the text content prior to visual formatting.)  Regarding
[UX](https://en.wikipedia.org/wiki/User_experience), we will follow the
[KISS principle](https://en.wikipedia.org/wiki/KISS_principle).

## Get started with Whisper CPP
### Download Whisper CPP
In a [terminal emulator](https://en.wikipedia.org/wiki/Terminal_emulator)
instance, create a local clone of the Whisper C++ Git repo with:
```
$ git clone https://github.com/ggerganov/whisper.cpp.git
```

Then, navigate into the `whisper.cpp` directory to start using Whisper CPP.

### How to update the application Whisper CPP
After a new [point version](https://en.wikipedia.org/wiki/Software_versioning)
is released, run the following:
```
$ git pull
$ git log  # Scroll through stdout to find the point release
$ git checkout <commit-checksum-in-hexadecimal>  # Or obtain hash in GH releases
```

To revert the repo to the latest commit to prepare for a new update, run:
```
$ git switch <primary-branch-name>  # Usually master or main
```

### How to update each LLM
You only need to redownload each LLM when it has changed.  Reference the Git LFS
([Large File Storage](https://git-lfs.com/)) [repo](https://huggingface.co/ggerganov/whisper.cpp/tree/main)
on [Hugging Face](https://en.wikipedia.org/wiki/Hugging_Face) to check for LLM
updates.

## How to transcribe with Whisper CPP
While in the Whisper CPP directory, convert any input audio file into a 16kHz
[WAV](https://en.wikipedia.org/wiki/WAV) file with [FFmpeg](https://en.wikipedia.org/wiki/FFmpeg):
```
$ ffmpeg -i <input.audio> -ar 16000 <output.wav>
```

Then, run:
```
$ ./main -m models/ggml-large-v2-q5_0.bin -pp -pc -otxt -ovtt -osrt -f <output.wav>
```

(I will further explain the flags used for: confidence colors and the output in
[plain text](https://en.wikipedia.org/wiki/Plain_text), [SRT](https://en.wikipedia.org/wiki/SubRip),
and [VTT](https://en.wikipedia.org/wiki/WebVTT) formats.  I will also discuss
the quantized versions of the `large-v*` LLMs and possibly using the distilled
and/or the `tinydiarize` models in the long-term future.)

### `asciinema` 1-liner
Instead of trying to automate [`asciinema`](https://asciinema.org/), use the
built-in `-c` flag to record 1 command with the `-i` flag (to limit recording
idle time up to 2 seconds):
```
$ asciinema rec -i 2 filename.cast -c "./main -m models/ggml-large-v2-q5_0.bin -pp -pc -otxt -ovtt -osrt -f /path/to/output.wav"
```

After the output files in a separate location, repeat with `large-v1-q5_0`,
`large-v3-q5_0`, or any other Whisper CPP model.

#### Future possibilities

Using `asciinema` to save Whisper CPP's color output has shown me there are many
possibilities through [pipelining](https://en.wikipedia.org/wiki/Pipeline_(Unix))
when using Whisper CPP.  One example is to make a video of Whisper CPP's output
via [VHS](https://github.com/charmbracelet/vhs) from [Charmbracelet](https://charm.sh/).

## End products
There are three broad categories I use Whisper CPP for.

### A) Reminding myself what happened during meetings
From the words of others and not mine, I have basically outdone my predecessors
when it comes to taking notes from board meetings thanks to Whisper CPP.

With the consent of everyone in board meetings, I use the
[Zoom H1n recorder](https://zoomcorp.com/en/us/handheld-recorders/handheld-recorders/h1n-vp-handy-recorder/)
and a [lavalier mic](https://en.wikipedia.org/wiki/Lavalier_microphone) to
record the meeting.  Afterwards, I then use Whisper CPP on the recording.
Lastly, this lets me make very detailed meeting minutes.

Fortunately, I don't have to edit the output from Whisper CPP.  Due to the
lossless quality of the recording setup, Whisper CPP almost always
[diarizes](https://en.wikipedia.org/wiki/Speaker_diarisation), since I am
downscaling the audio quality for Whisper CPP.

### B) Writing a written transcript
This is typically for a podcast, where there is no video - so, I don't have to
synchronize the subtitles.

I don't always have access to the original audio file, so I tend to
"upscale" the audio quality on a lossy audio file.

This is where I have noticed that, along with the audio quality aspect, Whisper
performs worse for people of color; and I'm not talking about new English
language learners.  I mean I'm basically watching the audio version of
[_Coded Bias_](https://en.wikipedia.org/wiki/Coded_Bias) play out right in front
of me when I work with Whisper CPP.  This empirically validates everything that
[Joy Buolamwini](https://en.wikipedia.org/wiki/Joy_Buolamwini) and her
colleagues have to say about discrimination in facial recognition AI, except
this is in speech recognition.

#### Inspirations for podcast transcriptions:

* [_Darknet Diaries_](https://en.wikipedia.org/wiki/Darknet_Diaries)
    * Example: 
* ["How to Fix the Internet"](https://www.eff.org/how-to-fix-the-internet-podcast) by the [EFF](https://en.wikipedia.org/wiki/Electronic_Frontier_Foundation)
    * Example: ["Don't Be Afraid to Poke the Tigers"](https://en.wikipedia.org/wiki/Electronic_Frontier_Foundation) with Andrew "bunnie" Huang

### C) Making synchronized subtitles for videos
This is a bit tough, as I have the least experience with this.

Currently, I know that [FFsubsync](https://github.com/smacke/ffsubsync) is a
Python program that's supposed to help with this, but I somehow could not get it
to work.

So, I used the online [version](https://subsync.online/en/online.html) of
[SubSync](https://github.com/sc0ty/subsync).

(I will really need to look into this in the future.)

## Working sources
I used these to help me figure out the actual programming:

* Stack Overflow [Q\&A](https://stackoverflow.com/questions/10228760/how-do-i-fix-a-git-detached-head) on a detached HEAD in Git
* Documentation of Whisper CPP and `asciinema`

## Thanks
* Russian Bear, my long-time collaborator for assistance with [Git](https://en.wikipedia.org/wiki/Git)
    * A convoluted "pun", based on purposeful cyber misattribution to Russian [APTs](https://en.wikipedia.org/wiki/Advanced_persistent_threat) (such as [Cozy Bear](https://en.wikipedia.org/wiki/Cozy_Bear)), and the general wariness of Russians by the policymakers of the U.S.A. stemming from the [Illegals Program](https://en.wikipedia.org/wiki/Illegals_Program) and the TV show [_The Americans_](https://en.wikipedia.org/wiki/The_Americans)
* The [threat model](https://tillitis.se/products/threat-model/) of [Tillitis](https://tillitis.se/) for inspiration
    * Tillitus is a hardware company that spun off from [Mullvad](https://en.wikipedia.org/wiki/Mullvad) in [2022](https://mullvad.net/en/blog/mullvad-creates-a-hardware-company)
* There are more entities to thank, but we're not quite ready yet

## License
This is licensed under the [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en)
(Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International)
License.

