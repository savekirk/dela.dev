---
title: "Hacking ExoPlayer Download Service"
date: 2023-06-03T14:44:51+03:00
description: "Use Android ExoPlayer download service and manager to download files that can be used outside ExoPlayer."
categories: [software engineering]
tags: [android,media]
draft: false
---
{{< svg "exoplayer_downloader.svg" >}}

## Introduction

Android [ExoPlayer](https://developer.android.com/guide/topics/media/exoplayer) has an in-built functionality for [downloading media files](https://developer.android.com/guide/topics/media/exoplayer/downloading-media). This is intended to be used by ExoPlayer for offline playback, as indicated in the [documentation](https://developer.android.com/guide/topics/media/exoplayer/downloading-media#playing-downloaded-content).
> Note: It's important that you do not try and read files directly from the download directory. Instead, use ExoPlayer library classes as described below.

Recently, I was working on a task that required downloading and processing of video files. I explored how to leverage the download service and manager that came with ExoPlayer.

## Downloading your media

Most of ExoPlayer's default components can be replaced with our own custom implementations. We'll make changes to the default components to achieve what we want. 

### Custom downloader

[Downloader](https://developer.android.com/reference/kotlin/androidx/media3/exoplayer/offline/Downloader) is responsible for downloading and removing content. ExoPlayer is bundled with some default implementations of the downloader. [ProgressiveDownloader](https://developer.android.com/reference/androidx/media3/exoplayer/offline/ProgressiveDownloader) is the closest to what we want, except we need to make a simple [change](https://github.com/androidx/media/blob/2fc189d6a40f116bd54da69ab9a065219f6973e7/libraries/exoplayer/src/main/java/androidx/media3/exoplayer/offline/ProgressiveDownloader.java#LL81C48-L81C48) to make it work for us. 

ProgressiveDownloader [fragments](https://github.com/androidx/media/blob/release/libraries/datasource/src/main/java/androidx/media3/datasource/DataSpec.java#L282) the downloaded data into multiple cache files, that is useful when playing the media whiles downloading it. In our situation, we want the data in a single cache file. Unfortunately, we can't extend the ProgressiveDownloader, so we can copy the code and make that single change. 

You can read through the code [here](https://gist.github.com/savekirk/a5a0eccbb805f64ae6170fbca6c61893#file-customprogressivedownloader-kt)

### Custom download factory 

To use our downloader, we need to inject into a [DownloaderFactory](https://developer.android.com/reference/kotlin/androidx/media3/exoplayer/offline/DownloaderFactory).

{{< gist savekirk a5a0eccbb805f64ae6170fbca6c61893 CustomDownloaderFactory.kt >}}

### Downloader manager

Finally, we can create our [DownloadManager](https://developer.android.com/reference/kotlin/androidx/media3/exoplayer/offline/DownloadManager) that will use our custom components.

{{< gist savekirk a5a0eccbb805f64ae6170fbca6c61893 download_manager.kt >}}

### Rewriting downloaded file

ExoPlayer downloader uses **.exo** file extension to persist the cached media. We will add a download listener that listens to the download completed state and then change the format of the file.
In the code snippet below, we are expecting an **.mp4** file. 

{{< gist savekirk a5a0eccbb805f64ae6170fbca6c61893 download_listener.kt >}}