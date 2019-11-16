---
title: "Post: Visualizing Spotify Data with D3 and Postman's new Visuzliation Feature"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - d3
  - spotify
  - postman
  - javascript
---

Postman recently announced support for visualizations that can display response data using virtually any javascript library out there. Well.. damn, what does this mean exactly? Say you have a collection with a few GET requests that fetch information about your AWS cloud infrastructure. The information you fetch may contain CPU/memory utilization, which can then be displayed using a bar chart or line graph. This feature is incredibly useful if you are a visual person like me, and like to get a high-level overview in a nice pretty graph. Not to mention, you can show high-level graphs to management without all the techno jargon. But besides seeing data in a pretty fashion, what could you really use it for? Well, if you are a developer, it may be useful for rapid prototyping of visualizations you plan on placing on a production web server. Since you can import/export collections with ease, you could package up your visualizations and requests. Thanks to Postman's Sandbox JavaScript execution environment, you don't need a server or any backend code to showcase a visualization idea. Simply export your collection and have the receiver import it into their own Postman application.  

In this demo, we are going to showcase the visualization feature and how to get started. The visualization uses a force-directed graph to show the relationship between playlists and their tracks. It also shows tracks that are apart of multiple playlists. This demo does not require you to set anything up within your Spotify Developer account, however, if you'd like to view your own Playlist and tracks data, you can, but it isn't covered in this post. I will be providing the script that gathers the data from the Spotify API and outputs it in a format that D3 will like. 

<!--more-->

Spotify allows a certain portion of your personalized information viewable to the public, granted they have a valid Spotify account. When you create a playlist, it automatically defaults to public. Anyone can query your username to gather information about your public playlists, including the tracks within that playlist and listening habbits. Since all of my playlists are public, we can use the client credentials method which only performs authentication, but not authorization. More information on OATH2 and spotify security [here][auth-flows].



If you don't already have Postman installed, head over to Postman's [download page][postman-download] and install it. It works on most platforms. 

Once installed download the postman collection here: 

Import the postman collection:

![Postman Import](/assets/images/postman-d3-spotify/import.png)

Visualizations are written in pure JavaScript which can be placed in either the 'Pre-request Script' or 'Tests' tab. The methods that are available when writing normal API tests are also available for Visualizations. All code related to the visualization itself needs to be placed inside a JavaScript template literals. Postman then takes that template, runs it as JavaScript code, and displays it in the Visualize (Beta) response tab. 

Note - Using template literals within the template does not work. For example, after defining the width constant within the template, I am unable to reference it using a template string later. Not that big a deal, but if you were copying code directly from something you're already using on a webserver, it's something to be aware of. 

In the Tests tab, you'll notice the `tmpl` variable. This is a string template which will encompous all of our javascript code.

![Postman Import](/assets/images/postman-d3-spotify/tests.png)


Within the template string, access the response data from the Postman request by using the `pm.getData(callback)` method Postman provides:

```
pm.getData(function (err, json) {
    if (err) console.error("Unable to access data in script", err);
    graph = json
    startGraph();
});
```

_Note: We use "${pm.variables.get("spotifyDataUrl")}" to access variables saved within this Postman collection. Don't forget the quotes on the outside because javascript will repalce this literally and run it as javascript code. Adding the quotes ensures that it interprets it as a string. Dynamic variables like `{{someVariable}}` cannot be used in the sandbox._

For those of you who don't know why the callback is needed - Since Javascript is Asynchronous non-blocking for I/O operations, we need to wait for the data to be received before trying to render the callback function, which is our visualization. In other JavaScript code, you may have seen events handled like this using the Async/Await or promise syntax. The callback function in the `pm.getData` method takes in error and response data. After the response is received and if there is no error, it sets the graph variable to the json data received, and then runs the `startGraph()` function, which is the graphing magic. 

Finally, outside of our template we call `pm.visualizer.set(tmpl, pm.response.json());` to take in our template and our response data. This makes the data available to our visuzliation. 

Click `Send` and you should see the response window show up on. Click the `Visualize` tab. 

![Postman Import](/assets/images/postman-d3-spotify/resp.png)

Click on the box that says `Select Playlists...` to start selecting playlists to get them to display on the screen. This graph allows you to zoom in/out using the mouse scroll wheel and pan left/right by clicking, holding, and moving your mouse cursor. 

![Postman Import](/assets/images/postman-d3-spotify/d3-select-playlists.png)

Try selecting two playlists that have songs in common. In this case, the nodes will display green. You can add and remove Playlists on the fly.

![Postman Import](/assets/images/postman-d3-spotify/viz-response.png)

Zooming in and hovering over nodes allows you to view a track or playlist name more clearly. You can also click and drag nodes to move them about. 

![Postman Import](/assets/images/postman-d3-spotify/zoomed-viz.png)

_Note: you can right-click the visualization and use the `inspect visualization` feature that pops up developer tools just like Chrome to do some debugging should you have any issues. Additionally, you can view errors using the Postman console in the bottom left-hand pane._ 

Welp, that's it! What visualization ideas do you have for your data? I think this feature is freaking awesome and will be very useful for rapid prototypes and showing off your work. I'd personally love to see them implement a full screen mode for the Visualize tab. Currently, I think the Window is way too small and should be able to seperate. 

Bonus:

For those interested in collecting their own data (Granted you have public playlists too), set up tokens in your Spotify Developer account. Then head over to my [github repo][d3-spotify-github] and follow the instructions to create an output of your tracks, playlists for d3/Postman to consume. After you have the output file, you can use github gist or something similar to host the json file. 


[postman-download]: https://www.getpostman.com/downloads/

[d3-spotify-github]: https://github.com/kirbocannon/d3-spotify

[auth-flows]: https://developer.spotify.com/documentation/general/guides/authorization-guide/#client-credentials-flow`