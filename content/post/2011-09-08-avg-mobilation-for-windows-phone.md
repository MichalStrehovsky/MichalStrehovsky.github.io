---
title: AVG Mobilation for Windows Phone
date: 2011-09-08T06:38:41+00:00
tags:
  - reversing
---
A new app hit the Windows Phone marketplace today that claims to keep your device safe from malware. I immediately became interested in it because:

  1. I don&#8217;t know of any malware for the Windows Phone.
  2. Even if there was malware that misuses some kind of hole in the Windows Phone security model, this app wouldn&#8217;t be able to catch it because of phone&#8217;s application model (unless the app itself uses some kind of security hole).
  3. After installing it, it claimed it offers real time protection that would suggest it&#8217;s capable of running in the background.

I would consider it a joke app, if it didn&#8217;t come from a well-known antivirus company. (Spoiler: It actually is a joke app, but the joke is on the antivirus company.)

# A look inside

To satisfy my curiosity, I downloaded the XAP file of the app with [Marketplace Browser and Downloader for Windows Phone 7][1] and opened it with Reflector. Surprise, surprise, this app was ported from Android (or at least that&#8217;s what *Droid namespace names suggest). Funny how the game has changed and instead of porting antivirus software from a Microsoft operating system to Linux, people started doing it the other way around.

The scanning UI is concentrated in the `DroidSecurityPOC.Scan` class and gets invoked in the `OnNavigatedTo` method. The `OnNavigatedTo` method is actually the first nugget:

```csharp
protected override void OnNavigatedTo(NavigationEventArgs e)
    {
        // uninteresting code removed

        // do the actual scanning, synchronously (we are scared of threads...)
        (Application.Current as App).malwareCollection.ScanContainingMedia(this.library);

        // simulate work in the UI even though the scanning is already completed at this point
        // people will love this
        this.StartScan();
    }
```

The StartScan method looks at the number of files to scan, divides 5 seconds with that number and starts a timer to update the &#8220;currently scanned&#8221; file name in the UI. Scanning will always take 5+ seconds to complete (closer to 5 seconds if you have few files to scan) and most of the time will be spent waiting for the next timer event to fire. Because all the &#8220;scanning&#8221; already happened in the ScanContainingMedia method, long time before the UI was first updated.

# The scanning algorithm

The `DroidSecurityPOC.Data.MalwareCollection` class is where the hilarity starts. The `ScanContainingMedia` method is where all the &#8220;scanning&#8221; happens. It&#8217;s split up in 2 parts: scanning your picture library and scanning your music library. The method doesn&#8217;t look at anything else (but that&#8217;s not much of a surprise given a marketplace application really cannot access anything else).

At this point, I was still giving the app a chance. Maybe it&#8217;s scanning for damaged files that can trigger known exploits in music players or picture viewers. All my hopes disappeared when I looked at the code:

```csharp
private void ScanContainingMedia(PictureCollection mediaFileCollectiont)
{
    // uninteresting code removed
    // malwareGroup contains a list of known "malware"

    // for each picture in the library
    foreach (Picture picture in mediaFileCollectiont)
    {
        // for each known malware (because HashSet is overrated)
        foreach (string str in malwareGroup.MalwareGroupList)
        {
            // compare malware name with current file name (!!!!!!!)
            // NOTE: We call ToLower() on each string to allocate a new string
            // and never cache the result. This way the garbage collector will
            // be busy picking up redundant trash and we can have some fun time
            // with his daughter.
            // Also, String.Equals(s1, s2, StringComparison.OrdinalIgnoreCase)
            // is for pussies.
            if (str.ToLower() == picture.Name.ToLower())
            {
                // uninteresting code - add malware to a collection of "effected malware"
            }
        }
    }
}
```

Basically, this code couldn&#8217;t be less bothered about the file contents. It only looks at the file name and if it matches the predicate, boom, it&#8217;s jailed. No questions asked. Do not pass Go. Do not collect $200.  
The list of &#8220;dangerous file names&#8221; is downloaded from a web service and Rafael Rivera can [show you][2] the current &#8220;definition file&#8221;.

The code also contains an unused method that hints at a future update that will actually look at the file contents, but the method makes me really scared:

```csharp
public bool ScanEicar(Picture picture)
{
    Stream image = picture.GetImage();
    image.Position = 0L;
    while ((image.Position + 0x44L) &lt;= image.Length)
    {
        // the garbage collector still doesn't seem to be busy enough, so
        // let's allocate an array in a tight loop
        byte[] buffer = new byte[70];
        image.Read(buffer, 0, 0x44);

        // BLAM! Potentially triple the amount of allocated memory by allocating
        // a string with the contents of the buffer. Note each character
        // in a string takes up 2 bytes.
        // Except Convert.ToString will actually return string "System.Byte[]"
        // for each and every call. What the author probably wanted
        // is Encoding.ASCII.GetString().
        if (Convert.ToString(buffer).Contains(@"X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*"))
        {
            image.Close();
            return true;
        }

        // Scanning fail: the call to image.Read() already moved the position
        // by 0x44 bytes. What the author probably wanted to do is
        // image.Position -= 0x43, but if he did that, the while loop would
        // run for each byte in the file, allocating about 210 MB from the heap
        // for a 1 MB file, so the algorithm is probably better off this way.
        image.Position += 1L;
    }
    image.Close();
    return false;
}
```

Everything (including the release date) hints at this being some kind of a summer intern project at AVG (if it's not, it's very disturbing). But AVG, c'mon. Interns do all kinds of wonky stuff. You really don't need to ship all of it...

 [1]: http://mktwp7.codeplex.com/
 [2]: http://www.withinwindows.com/2011/09/07/the-only-time-youll-see-avg-security-suite-warn-you-about-malware-on-windows-phone-7/