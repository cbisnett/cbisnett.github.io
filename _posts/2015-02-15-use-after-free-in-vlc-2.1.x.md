---
layout: news_item
author: cbisnett
categories: [fuzzing]
title: Use-After-Free in VLC 2.1.x
---

**tldr;** I found a vulnerability in VLC while creating a training course on fuzzing.  I reported it to the VLC maintainers but they declined to fix it.  I contend it's a security vulnerability.  Here is the evidence, you decide.

I've been meaning to write this post for a while.  A year and a half ago I started developing a [training course](/2015/02/10/blackhat-usa-2015-training-announced) to teach a vulnerability discovery technique called fuzzing.  As part of the course I wanted students to gain first hand knowledge and find a vulnerability in a "real" deployed application.  I feel that this helps validate the technique and prove it's usefulness.

I started browsing recent CVE reports looking for a vulnerability that I could try and find using a fuzzer.  This is not as easy as it seems because there are two factors I had to keep in mind: first I wanted the application to be something that people actually use and second it needs to be a bug which can be fuzzed by a beginner in a day or two.  Yeah I know, it's unlikely I'll find something I can use.  I took a quick look and nothing popped out at me.

I did however, find a target: [VLC](http://www.videolan.org/vlc/index.html).  Since VLC is a media player it has to parse complicated audio and video formats from untrusted sources.  Fuzzing a file can be as simple as replacing a percentage of bytes of a file.

After writing a simple Python script to replace random bytes in input files I got started fuzzing.  It didn't take long (a couple hundred iterations) for the fuzzer to start finding crashes.  This is the details of one of them.

#### The Details
Opening the video will render corrupted playback for a few seconds before the window resizes and shows a black background with the VLC icon for a few seconds before VLC crashes and closes.

As I said before, I found this vulnerability in the fall of 2013 and at that time the current version was 2.1.4.  I've tried building newer sources but after many hours of fighting trying to get a successful build, I've given up and the following code excerpts will be from the official 2.1.4 source release.

I reported this bug to the VLC maintainers but they declined to fix the vulnerability and instead downplayed it since the bug doesn't affect the 2.2.x or 3.x branches.  While it is true that it doesn't affect the current 2.2.0 or 3.0.0 nightlies at the time of publishing, the 2.2.x branch was vulnerable when I reported it.  From my perspective that doesn't seem to matter much since the download page still serves up the 2.1.x binaries which are vulnerable.

The full [bug](https://trac.videolan.org/vlc/ticket/12754) report contains a complete [AddressSanitizer](https://code.google.com/p/address-sanitizer/wiki/AddressSanitizer) output with symbols resolved.  AddressSanitizer reported this crash as a [Use-After-Free](http://cwe.mitre.org/data/definitions/416.html) vulnerability.  This is a stack trace for the faulting instruction:
{% highlight bash %}
=================================================================
==5667== ERROR: AddressSanitizer: heap-use-after-free on address
0x602a0005ea48 at pc 0x7f008f7971f7 bp 0x7f0077e74280 sp 0x7f0077e74278
WRITE of size 8 at 0x602a0005ea48 thread T36
    #0 0x7f008f7971f6 in vlc_atomic_sub vlc/src/../include/vlc_atomic.h:340
    #1 0x7f008f7618cc in vout_ReleasePicture vlc/src/video_output/video_output.c:436
    #2 0x7f008f6fc75d in vout_unlink_picture vlc/src/input/decoder.c:2435
    #3 0x7f008f704c97 in decoder_UnlinkPicture vlc/src/input/decoder.c:206
    #4 0x7f006e156a22 in ffmpeg_ReleaseFrameBuf vlc/modules/codec/avcodec/video.c:1081
    #5 0x7f006d56a433 in compat_free_buffer libav/libavcodec/utils.c:563
    #6 0x7f006ce52d6e in av_buffer_unref libav/libavutil/buffer.c:115
    #7 0x7f006d56a465 in compat_release_buffer libav/libavcodec/utils.c:570
    #8 0x7f006ce52d6e in av_buffer_unref libav/libavutil/buffer.c:115
    #9 0x7f006ce58a82 in av_frame_unref libav/libavutil/frame.c:285 (discriminator 2)
    #10 0x7f006d56c948 in unrefcount_frame libav/libavcodec/utils.c:1414
    #11 0x7f006d56cd0e in avcodec_decode_video2 libav/libavcodec/utils.c:1496
    <snip>
{% endhighlight %}

Looking at the stack trace it's pretty clear this has something to do with releasing a picture instance.  This is the structure of a `picture_t`:
{% highlight c++ %}
struct picture_t
{
    video_frame_format_t format;
    plane_t         p[PICTURE_PLANE_MAX];     /**< description of the planes */
    int             i_planes;                /**< number of allocated planes */
    mtime_t         date;                                  /**< display date */
    bool            b_force;
    bool            b_progressive;          /**< is it a progressive frame ? */
    bool            b_top_field_first;             /**< which field is first */
    unsigned int    i_nb_fields;                  /**< # of displayed fields */
    picture_sys_t * p_sys;
    struct
    {
        vlc_atomic_t refcount;
        void (*pf_destroy)( picture_t * );
        picture_gc_sys_t *p_sys;
    } gc;
    struct picture_t *p_next;
};
{% endhighlight %}

Based on the functions in the stack trace and the faulting function `vlc_atomic_sub`, I assumed I could narrow this down to the refcount member since it's the only atomic field in the structure.  A little work with GDB proved this to be correct:
{% highlight bash %}
(gdb) p &((struct picture_t*)0)->gc.refcount
$1 = (vlc_atomic_t *) 0x128
(gdb) p sizeof(struct picture_t)
$2 = 328
{% endhighlight %}

This aligns with the AddressSanitizer output which reported the faulting instruction attempted to write 8 bytes at offset 296 or 0x128.  This isn't the instruction which attempted the write but it's related.  I'll explain more in a minute.

This also explains the maintainers [comment](https://trac.videolan.org/vlc/ticket/12754#comment:7) that this causes an assertion to fail in the 2.1.x branch.  While I didn't hit any assertion (due to my build settings) I expect the assertion that the maintainers are referring to is an assertion in `PictureDestroy`.  This function is defined in `src/misc/picture.c` and is assigned to the `pf_destroy` field of the garbage collection structure when the `picture_t` structure is created.
{% highlight c %}
static void PictureDestroy( picture_t *p_picture )
{
    assert( p_picture &&
            vlc_atomic_get( &p_picture->gc.refcount ) == 0 );

    vlc_free( p_picture->gc.p_sys );
    free( p_picture->p_sys );
    free( p_picture );
}
{% endhighlight %}

The developer is asserting that they are indeed freeing an instance whose reference count is zero, meaning there are no other references which still require access to this memory.  A violation of this assertion means one of two things: the instance is being freed while references to it's memory still exist, or that this instance has already been freed in which case the reference count will be negative.

In this case I know this to be the second possibility since AddressSanitizer tracks all allocations and frees.  Looking at the AddressSanitizer output we can see the stack trace for the free also ends in `PictureDestroy`.  I verified this by running the sample through GDB.  You can see from the following output that at the time the `vlc_atomic_sub` function is executed the reference count field is zero:
{% highlight bash %}
Breakpoint 2, __asan_report_error (pc=140737298543095, bp=140736845898368, sp=140736845898360, 
    addr=105733505280584, is_write=true, access_size=8)
    at ../../../../libsanitizer/asan/asan_report.cc:628
628                          uptr addr, bool is_write, uptr access_size) {
(gdb) up
#1  0x00007ffff5053857 in __asan::__asan_report_store8 (addr=addr@entry=105733505280584)
    at ../../../../libsanitizer/asan/asan_rtl.cc:234
234 ASAN_REPORT_ERROR(store, true, 8)
(gdb) up
#2  0x00007ffff4afb1f7 in vlc_atomic_sub (v=1, atom=0x602a0005ea48) at ../include/vlc_atomic.h:340
340     return atomic_fetch_sub (&atom->u, v) - v;
(gdb) p *atom
$1 = {u = 0}
(gdb)
{% endhighlight %}

This `atomic_fetch_sub` call contains the instruction which reads the value at `atom->u` subtracts 1 and writes it back to memory.  Writing the new value back to memory is what triggers the AddressSanitizer crash since it's writing to memory that was freed and has not yet been re-allocated.

We now have a `picture_t` structure which has been freed twice.  Without AddressSanitizer this will crash due to heap corruption resulting from the double free.

#### Exploitation

I want to start this section by stating that I haven't spent much time analyzing the exploitability of this vulnerability but for the sake of argument I'm going to walk through why I believe this vulnerability can be successfully exploited.

The sheer fact that the garbage collection structure contains a function pointer which will be invoked to free the instance is a huge step in the right direction.  If an attacker is able to cause an allocation after the first free and before the second, they may be able to direct execution to a ROP gadget and pivot the stack to attacker controlled data.  From that point it's simply a matter of chaining ROP gadgets to call `mprotect` and return to the newly executable memory.

#### Conclusion

Yes I understand that I just glossed over some very technical and often difficult to overcome challenges but this blog post isn't about exploiting this vulnerability merely challenging it's existence.  I contend this is a security vulnerability based on the details provided above.

If you would like to have a look for yourself you can find the [sample](https://www.dropbox.com/s/dlq06ugc0oturmu/1084463834816444598.mov?dl=0) in my Dropbox.  I'd be interested to know if anyone is able to turn this into a PoC or potentially a Metasploit module.  Good luck and happy hunting.