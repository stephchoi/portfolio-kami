---
layout: post
title: BlocJams
thumbnail-path: 'img/bloc_jams_home.png'
short-description: BlocJams is an interactive and responsive digital music player like Spotify with a simplified music library.
---

{:.center}
![]({{ site.baseurl }}/img/bloc_jams_home.png)

## Explanation
BlocJams is a a music streaming application originaly built with HTML, CSS, JavaScript and JQuery and refactored with AngularJS. It's user friendly, browser-responsive, and has a small music library that you can play tracks from.

# Features

The collection template displays all of the albums currently in the database of the application with album art, album title, artist name, and release date.

![]({{ site.baseurl }}/img/bloc_jams_colection.png)

When you click on the cover art, the page is redirected to the album template. This holds the entire list of songs within the album as well as the fixed playback bar on the bottom of the screen, where you can play, pause, skip, and adjust the volume of the song.

Hovering over the song number on the album converts the number into a play button. Using different Angular directives such as `ng-mouseover` and `ng-mouseleave`,, I could adjust small changes like this much more concisely than if I had tried to animate with plain JavaScript.

This:
{% highlight javascript %}
var onHover = function(event) {
  var songNumberCell = $(this).find('.song-item-number');
  var songNumber = parseInt(songNumberCell.attr('data-song-number'));

  if (songNumber !== currentlyPlayingSongNumber) {
    songNumberCell.html(playButtonTemplate);
  }
};
{% endhighlight %}

 becomes this:
 {% highlight html %}
 <tr class="album-view-song-item" ng-mouseover="hovered = true" ng-mouseleave="hovered = false" ng-repeat="song in album.albumData.songs">
 {% endhighlight %}

![]({{ site.baseurl }}/img/bloc_jams_album.png)

For music playback, I'm using the [Buzz music library](http://buzz.jaysalvat.com/) to stream the audio files in our library. The song durations are marked in seconds rather than the standard minutes and seconds (X:XX) that most people are used to. What used to need an algorithm is done super easily with `buzz.toTimer(seconds)` which takes the number of seconds and converts the value into a minutes and seconds format.  

The translucent player bar at the bottom also contains two seek bars: one that adjusts the volume and one that reflects the current place in the song.

{% highlight javascript %}
var calculatePercent = function(seekBar, event) {
  var offsetX = event.pageX - seekBar.offset().left;
  var seekBarWidth = seekBar.width();
  var offsetXPercent = offsetX / seekBarWidth;
  offsetXPercent = Math.max(0, offsetXPercent);
  offsetXPercent = Math.min(1, offsetXPercent);
  return offsetXPercent;
};
{% endhighlight %}

Setting and updating both seek bars are based on the ratio of the thumb's position on the bar to the full length of the bar and converts it to a percentage from 0 to 100.

{% highlight javascript %}
scope.onClickSeekBar = function(event) {
  var percent = calculatePercent(seekBar, event);
  scope.value = percent * scope.max;
  notifyOnChange(scope.value);
};

scope.trackThumb = function() {
  $document.bind('mousemove.thumb', function(event) {
    var percent = calculatePercent(seekBar, event);
    scope.$apply( function(){
      scope.value = percent * scope.max;
      notifyOnChange(scope.value);
    });
  });
{% endhighlight %}

If you click any position or drag the thumb to a random place on either seek bar, the application calculates the new percentages and updates the new volume or changes the place in the song currently playing.

{% highlight javascript %}
currentBuzzObject.bind('timeupdate', function() {
  $rootScope.$apply(function(){
    SongPlayer.currentTime = currentBuzzObject.getTime();
  });
});

SongPlayer.setCurrentTime = function(time) {
  if (currentBuzzObject) {
    currentBuzzObject.setTime(time);
  }
};
{% endhighlight %}


# Shortcomings

Obviously this application doesn't quite have the breadth of music that any user would want to have in their music streaming application. It has an Angular framework but no real database/ back-end support to make it a widely used and musically varied application.

However, as a whole there's a potential to build on the framework set. Currently the application only has one album within it's collection that's shown repeatedly in this format in order to mimic how a multi-album collection would look like. This way, if in the future we'd like to add more albums to the database, we won't have to worry about formatting the collections page, the wireframe for it is there.

 # Conclusions

 In general, I did notice how much easier it was to bind events using Angular than it was just using JavaScript and jQuery. The structure of the entire webpage/application with Angular was much more complex than that of the page without it. It makes a lot of sense because Angular sets up the frame for a much more complex application.

As a whole, the application is solid. It does what it needs to with what it has. And there's potential for future expansion if desired. Practically if I were to expand way beyond tens of albums, there would obviously need to have a much better method for displaying the music collection, perhaps allowing for searches of album titles or authors.  There are some bugs that still need fixing, for example after you can't drag the seek bar more than once in a row. But for a first project I would say it is rather successful.
