---
template: post
title:  "How to get your Top tracks (or whatever info) out of your Spotify account"
date: 2022-02-13
tags: ["spotify", "python"]
author: Ander Granado
---

I'm heavily into web scraping, I know, but this time it's something a bit different. It's not your typical web scraping; it's a small example of how to extract information about your own data. You can get some very interesting insights from the data we generate daily. For instance, in my case, I use Spotify daily to listen to music. I never leave home without headphones, and I even wear them in the bathroom, so I generate a certain amount of data while using Spotify.

## Downloading Your Information

Many applications allow users to download the information they generate. For most European citizens, it's worth mentioning that practically all applications enable this, as it's a right granted by [Article 15](https://support.spotify.com/es/article/gdpr-article-15-information/) of the European Union's GDPR. This means that nearly all online services let you download a copy of your data, at least in this part of the world.

This is interesting because it lets you extract information that the applications and services have about you but don't display through their interfaces. For instance, in the case of Spotify, you can download your [main data](https://support.spotify.com/es/article/data-rights-and-privacy-settings/), including things like your playback history for the last year. It's important to note that in Spotify's case, the data you get with their tool for downloading data isn't complete, but you can request absolutely everything by [contacting them directly](mailto:privacy@spotify.com).

This way, something that can't be seen within the Spotify app can be done through the data they allow you to download. Typically, this kind of data comes in formats like JSON or XML to interact with it programmatically. Sometimes, they even provide options for the format, as is the case with Instagram, where you can download a version in HTML (useful for having a backup) or in JSON (if you want to work with the data).

## Extracting Most Listened Content from Spotify's Information

Getting back to Spotify, you can extract a lot of information from this data. In my case, as a starting point to tinker with the data, I've simply programmed a script to see what my top listened songs and artists are. Some might think that Spotify already provides this in their annual "wrapped" summaries or in some generated playlists. In part, this is true, but it's given to you "their way." Meaning, you might know your Top 5 most listened-to artists, but not the Top 10, or how many times you've listened to a certain artist, or how much you've listened to an artist that's not in your top list, etc. The power of having all the data in your hands is that you can **extract the information you want and how you want it**.

In this case, I've tried with basic data, which has limited information. If you request the copy with all the information, you can obtain a lot more data. In Spotify's [documentation](https://support.spotify.com/es/article/understanding-my-data/), you can see exactly what data they offer in both cases. For the case of basic data, Spotify provides a zip file with a handful of JSON files.

![Spotify Data](/static/images/spotify_data.png "Basic data provided by Spotify")

Among the files provided by Spotify, the ones relevant for this PoC are named `StreamingHistory[x].json`. These files contain the playback history. The information regarding the song is quite limited but enough to identify it. The file has a format like the following:

```json
[
  {
    "endTime" : "2021-02-05 11:10",
    "artistName" : "Hüsker Dü",
    "trackName" : "Don't Want to Know If You Are Lonely",
    "msPlayed" : 212426
  },
  {
    "endTime" : "2021-02-05 12:16",
    "artistName" : "Queens of the Stone Age",
    "trackName" : "Go With The Flow",
    "msPlayed" : 4890
  },
  {
    "endTime" : "2021-02-05 12:23",
    "artistName" : "Queens of the Stone Age",
    "trackName" : "Go With The Flow",
    "msPlayed" : 190646
  },
  {
    "endTime" : "2021-02-05 12:24",
    "artistName" : "Queens of the Stone Age",
    "trackName" : "Make It Wit Chu",
    "msPlayed" : 19403
  },
```

And so on infinitely. It provides the timestamp, song name, artist, and playback time. This last piece is interesting because you can set a minimum threshold to consider a song as listened, for example. You can also play around with the timestamp to see when you've listened to more or less music. The possibilities are vast.

With this, the script I've created to get the top most listened songs and artists is as follows:

`gist:ander94lakx/63776dacd6e986d935b9f01fff755921#spotify_top_songs_and_artists.py`

I don't think it's worth going into detail about the code since it's quite simple, and most of it is self-explanatory. It's basically a function that performs a series of steps that can be summarized as follows:

1. Manage an argument to configure the size of the Top.
2. Read the files and load JSON information into dictionaries.
3. Perform calculations on the loaded information.
4. Sort, prepare, and display the top listened songs and artists.

Certainly, there might be many ways to improve it. You could unify the part where you fetch the data with the part where you calculate the playbacks, but I wanted to keep it separate to make it easier to understand. Additionally, my Python skills aren't perfect, and some things with sets (among many others) could probably be done better.

## Conclusions

Having access to the data you generate in an application enables you to extract a lot of information. Applications and services know this and use the data to extract insights for their benefit. At the very least, since they process user information, they should allow users to do the same.

Moreover, this allows you to extend the capabilities offered by the applications themselves. In Spotify's case, you could analyze the music you listen to, how you listen to it, or how your listening habits have changed over time – things Spotify doesn't offer its users. Similarly, you can do more of the same with other applications. The ability to obtain your own information is the first step to being able to handle it in your own way.

There are surely a thousand examples of interesting cases where this type of application-generated information can be used. If you're reading this, I invite you to download your information from the applications you use, take a look, and think about the interesting uses you can make of it. Information is power, and users should have that power too.

Don't stop downloading what belongs to you!

Happy hacking!