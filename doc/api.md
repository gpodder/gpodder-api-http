#gPodder HTTP-API Documentation

## Response Format:
All responses have 3 top level keys:

* **status**: HTTP status code
* **msg**: Error message or additional information 
* **response**: Data returned by the requested function (can be empty)

The data in `response` is usually a PodcastDict or EpisodeDict which are intended to be used as a patch for the local database. It only contains the requested or changed values.

###Common data formats:
**Podcast**:

    {
      "auto_archive_episodes": false, 
      "cover_url": "http://www.example.com/podcast/podcast.jpg", 
      "description": "Example Podcast", 
      "download_folder": "Podcast",
      "download_strategy": 0, 
      "http_etag": "\"111111111111111111111111111\"",
      "http_last_modified": "Wed, 26 Mar 2014 22:53:32 GMT", 
      "id": 1, 
      "link": "http://www.example.com", 
      "pause_subscription": false, 
      "payment_url": null, 
      "section": "audio", 
      "title": "Podcast", 
      "url": "http://www.example.com/podcast/podcast.xml"
    }

**Episode**:

    {
      "archive": false, 
      "current_position": 0, 
      "description": "First episode of our new podcast", 
      "download_filename": null, 
      "download_progress": 0.0, 
      "downloading": false, 
      "file_size": 4326269, 
      "guid": "http://example.com/podcast/episode1.mp3", 
      "id": 1, 
      "is_new": true, 
      "last_playback": 0, 
      "link": "http://example.com/podcast/episode1.mp3", 
      "mime_type": "audio/mp3", 
      "payment_url": null, 
      "podcast_id": 2, 
      "published": 1394139600, 
      "state": 0, 
      "title": "Podcast - Episode 1", 
      "total_time": 9001, 
      "url": "http://example.com/podcast/episode1.mp3"
    }

**PodcastDict**:

    {
      "msg": "OK", 
      "response": {
        "1": <PodcastChannel>,
        "2": <PodcastChannel>,
        "3": <PodcastChannel>,
        { ... }
      }, 
      "status": 200
    }

**EpisodeDict**:

    {
      "msg": "OK",
      "response": {
        "1": <PodcastEpisode>,
        "2": <PodcastEpisode>,
        "3": <PodcastEpisode>,
        { ... }
      },
      "status": 200
    }

**Empty Response**:
An empty response is returned by methods which only execute task to check if errors occured.

    {
      "msg": "OK", 
      "response": "", 
      "status": 200
    }

## Argument Format:
All arguments are passed using http queries. A plus indicates that multiple values may be passed, seperated using ",". **Example**: "id=1,2,3,5,7".  
Common Arguments:

* **id** - Podcast or episode ID, depends on the category of the method
* **url** - Podcast url, should be url encoded
* **title** - Podcast title, should be url encoded
* **prop** - Properties to be returned by the episodes method
* **podcast** - Podcast ID for episode methods
* **key** - Config key
* **value** - Config value


## Podcast Methods:

###/api/podcasts

**Description**: Returns a list of all or the requested podcast channels  
**Arguments**: [id+], [prop]  
**Example**: `http://localhost:5000/api/podcasts/`  
**Response**: PodcastDict of all podcasts

###/api/auth
**Description**: Returns the authentication data  
**Arguments**: id+  
**Example**: `http://localhost:5000/api/auth/?id=3`  
**Response**: PodcastDict with the "auth_username" and "auth_password" property only

###/api/subscribe
**Description**: Subscribes to the given url and optionally sets the title  
**Arguments**: url, title  
**Example**: `http://localhost:5000/api/subscribe/?url=http%3A%2F%2Fexample.com%2Fpodcast%2Fpodcast.xml&title=Podcast`  
**Response**: PodcastDict

###/api/unsubscribe
**Description**: Unsubscribes a podcast  
**Arguments**: id+  
**Example**: `http://localhost:5000/api/unsubscribe/?id=3,4`  
**Response**: Empty Response

###/api/enable
**Description**: Tries to enable the given podcasts and returns the enabled and skipped (already enabled) podcasts  
**Arguments**: id+  
**Example**: `http://localhost:5000/api/enable/?id=1,2`  
**Response**: PodcastDict with the "pause_subscription" property only

###/api/disable
**Description**: Tries to disable the given podcast and returns the disabled and skipped (already disabled) podcasts  
**Arguments**: id+  
**Example**: `http://localhost:5000/api/disable/?id=1,2`  
**Response**: PodcastDict with the "pause_subscription" property only

###/api/update
**Description**: Updates the given podcast or all podcasts  
**Arguments**: [id+]  
**Example**: `http://localhost:5000/api/update/`  
**Response**:

    {
      "msg": "OK", 
      "response": {
        "failed": [], 
        "paused": [], 
        "unchanged": [
          2
        ], 
        "updated": [
          1
        ]
      }, 
      "status": 200
    }

###/api/scan
**Description**: Scans the podcast directory for downloaded episodes, it's recommended to call /api/episodes afterwards  
**Arguments**: [id+]  
**Example**: `http://localhost:5000/api/scan/?id=1`  
**Response**: Empty response

###/api/rewrite
**Description**: Changes the url of an existing podcast  
**Arguments**: id, url  
**Example**: `http://localhost:5000/api/rewrite/?id=4&url=http%3A%2F%2Fexample.com%2Fnewurl%2Fpodcast.xml`  
**Response**: PodcastDict with the "url" property only

###/api/rename
**Description**: Changes the name of an existing podcast  
**Arguments**: id, title  
**Example**: `http://localhost:5000/api/rename/?id=4&title=Podcast1`  
**Response**: PodcastDict with the "title" property only


##Episode Methods:

###/api/episodes
**Description**: Returns the properties for single episodes, all episodes of a podcast or all episodes in the database.  
**Arguments**: [id+], [podcast+], [prop+]  
**Examples**:
`http://localhost:5000/api/episodes/?id=101,102&podcast=1,2&prop=title,url,total_time,is_new`
`http://localhost:5000/api/episodes/`  
**Response**: EpisodesDict with the requested properties

###/api/download
**Description**: Downloads the given episodes  
**Arguments**: [id+]  
**Example**: `http://localhost:5000/api/download/?id=7`  
**Response**: EpisodesDict with the "downloading" property only

###/api/cancel
**Description**: Cancels a download task  
**Arguments**: id+  
**Example**: `http://localhost:5000/api/cancel/?id=8,9`  
**Response**: EpisodesDict with the "state" property only

###/api/remove
**Description**: Deletes the given episodes  
**Arguments**: id+  
**Example**: `http://localhost:5000/api/remove/?id=6,7`  
**Response**: Empty response

###/api/new
**Description**: Marks the given episodes as new  
**Arguments**: id+  
**Example**: `http://localhost:5000/api/new/?id=3,4`  
**Response**: EpisodesDict with the "is_new" property only

###/api/old
**Description**: Marks the given episode as old  
**Arguments**: id+  
**Example**: `http://localhost:5000/api/old/?id=3,4`  
**Response**: EpisodesDict with the "is_new" property only

###/api/played
**Description**: Sets the playback position of an episode
**Arguments**: id, position
**Example**: `http://localhost:5000/api/played/?id=1&position=1392`
**Response**: EpisodeDict with the "total_time", "current_position" and "last_playback" properties only

##Other methods:

###/api/registry
**Description**: Dumps the registry  
**Arguments**: none  
**Example**: `http://localhost:5000/api/registry`  
**Response**:

    {
      "msg": "OK", 
      "response": {
        "content_type": {
          "description": "Resolve the content type (audio, video) of an episode", 
          "plugins": {
            "vimeo_resolve_content_type": "gpodder.plugins.vimeo", 
            "youtube_resolve_content_type": "gpodder.plugins.youtube"
          }
        }, 
        "cover_art": {
          "description": "Resolve the real cover art URL of an episode", 
          "plugins": {
            "youtube_resolve_cover_art": "gpodder.plugins.youtube"
          }
        }, 
        "download_url": {
          "description": "Resolve the real download URL of an episode", 
          "plugins": {
            "vimeo_resolve_download_url": "gpodder.plugins.vimeo", 
            "youtube_resolve_download_url": "gpodder.plugins.youtube"
          }
        }, 
        "episode_basename": {
          "description": "Resolve a good, unique download filename for an episode", 
          "plugins": {
            "vimeo_resolve_episode_basename": "gpodder.plugins.vimeo", 
            "youtube_resolve_episode_basename": "gpodder.plugins.youtube"
          }
        }, 
        "fallback_feed_handler": {
          "description": "Handle parsing of a feed (catch-all)", 
          "plugins": {
            "podcast_parser_handler": "gpodder.plugins.podcast"
          }
        }, 
        "feed_handler": {
          "description": "Handle parsing of a feed", 
          "plugins": {
            "itunes_feed_handler": "gpodder.plugins.itunes", 
            "soundcloud_fav_feed_handler": "gpodder.plugins.soundcloud", 
            "soundcloud_feed_handler": "gpodder.plugins.soundcloud", 
            "vimeo_feed_handler": "gpodder.plugins.vimeo", 
            "youtube_feed_handler": "gpodder.plugins.youtube"
          }
        }, 
        "podcast_title": {
          "description": "Resolve a good title for a podcast", 
          "plugins": {
            "vimeo_resolve_podcast_title": "gpodder.plugins.vimeo", 
            "youtube_resolve_podcast_title": "gpodder.plugins.youtube"
          }
        }, 
        "url_shortcut": {
          "description": "Expand shortcuts when adding a new URL", 
          "plugins": {
            "podcast_resolve_url_shortcut": "gpodder.plugins.podcast", 
            "soundcloud_resolve_url_shortcut": "gpodder.plugins.soundcloud", 
            "youtube_resolve_url_shortcut": "gpodder.plugins.youtube"
          }
        }
      }, 
      "status": 200
    }

###/api/version
**Description**: Returns the currently installed and latest version of gPodder  
**Arguments**: none  
**Example**: `http://localhost:5000/api/version/`  
**Response**:

    {
      "msg": "OK", 
      "response": {
        "latestdate": "2014-03-08", 
        "latestversion": "3.6.1", 
        "thisdate": "2014-03-29", 
        "thisversion": "4.1.0", 
        "url": "http://gpodder.org/"
      }, 
      "status": 200
    }

###/api/config
**Description**: Dumps the config  
**Arguments**: none  
**Example**: `http://localhost:5000/api/config/`  
**Response**:

    {
      "msg": "OK", 
      "response": {
        "auto.cleanup.days": 7, 
        "auto.cleanup.played": false, 
        "auto.cleanup.unfinished": true, 
        "auto.cleanup.unplayed": false, 
        "auto.retries": 3, 
        "auto.update.enabled": false, 
        "auto.update.frequency": 20, 
        "limit.bandwidth.enabled": false, 
        "limit.bandwidth.kbps": 500.0, 
        "limit.downloads.concurrent": 1, 
        "limit.downloads.enabled": true, 
        "limit.episodes": 200, 
        "plugins.youtube.preferred_fmt_id": 18, 
        "plugins.youtube.preferred_fmt_ids": [], 
        "ui.cli.colors": true
      }, 
      "status": 200
    }

###/api/set
**Description**: Sets a config value  
**Arguments**: key, value  
**Example**: `http://localhost:5000/api/set/?key=limit.bandwidth.kbps&value=1000.0`  
**Response**: same as /api/config

###/api/save
**Description**: Saves the config and db  
**Arguments**: none  
**Example**: `http://localhost:5000/api/save`  
**Response**: Empty response

###/api/importurl
**Description**: Imports an opml file from the given url  
**Arguments**: url  
**Example**: `http://localhost:5000/api/importurl/?url=http%3A%2F%2Fexample.com%2Fpodcast%2Fpodcasts.opml`  
**Response**: Empty response

###/api/importfile
**Description**: Imports an uploaded opml file  
**Arguments**: none, you have to upload the file using a push request  
**Example**: `curl -X POST -d @podcasts.opml http://localhost:5000/api/importfile`  
**Response**: Empty response

###/api/export
**Description**: Exports all subscriptions as an opml file  
**Arguments**: none  
**Example**: `http://localhost:5000/api/export/`  
**Response**: opml content, no json response!

###/files/episode_id
**Description**: Method for accessing the downloaded file of an episode  
**Arguments**: none, use the episode_id argument in the url  
**Example**: `http://localhost:5000/files/12`  
**Response**: Media file of the requested episode, no json response!
