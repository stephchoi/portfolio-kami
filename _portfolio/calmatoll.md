---
layout: post
title: Calm Atoll
thumbnail-path: "img/calm-atoll.png"
short-description: Draw on a shared whiteboard while chatting with a friend in real-time.
---

{:.center}
![]({{ site.baseurl }}/img/calm-atoll.png)


## Explanation

Using PeerJS and WebRTC to send data, Calm Atoll let's you chat with real-time audio, send links over the messenger, and draw on a shared whiteboard space. Once connected to the secure PeerJS server, a username is set and displayed for the user to be shared with others. The PeerJS server establishes both a media and data connection between the user's and the peer's username.

Initially I thought of the idea as a video chat with a shared whiteboard space instead of the option of sharing a screen. You could talk to each other and work out problems on the whiteboard space together - like *Draw with Friends* on your computer screen. I added the message chat aspect later on so that the participants could share links with each other for videos or webpages that would be fun.

## Problem Solving

To tackle this daunting task I broke it up into smaller chunks based upon the functionality. I started with the HTML5 canvas whiteboard making event listeners for `mousedown` and `mousemove` that drew the line as it pushed the XY coordinate of each point on it into the `coord` array.

{% highlight javascript %}
canvasX = e.pageX - canvas.offsetLeft;
canvasY = e.pageY - canvas.offsetTop;
context.lineTo(canvasX, canvasY);
coord.push([canvasX, canvasY]);
context.strokeStyle = "#000";
context.stroke();
{% endhighlight %}

These coordinates are what are sent over the WebRTC connection on the `mouseup` event and used to recreate the same image on the other user's screen.

Initially, I had used [Pusher](https://pusher.com/tutorials/webrtc_chat) to create the WebRTC data connection. While it works amazingly for sending data, Pusher doesn't have media streaming, which is necessary for an audio chat application. After a bit of searching I found that [PeerJS](peerjs.com/docs) had media streaming capabilities as well as a data channel connection, so I refactored the Pusher code into Peer code.

With Peer, the server assigns a random username, which you can use to connect to another peer user to create both the data channel and media stream. After the initial switch, the whiteboard data and chat messages went through to the other user. However, the audio call would connect but it wouldn't play the audio of the other side.

It turns out a separate audio playing function was required to playback the received audio stream from a connected peer.

{% highlight javascript %}
function playStream(stream) {
  var audio = $('<audio autoplay />').appendTo('body');
  audio[0].src = (URL || webkitURL || mozURL).createObjectURL(stream);
}
{% endhighlight %}

When placed into the the `call` event,  `playStream` retrieves the audio stream of `requestedPeer`, aka the connected peer. Without this it was no wonder that the audio call wasn't working.

{% highlight javascript %}
navigator.getUserMedia({video:false, audio: true}, function(stream) {
  var options = {
      'constraints': {
          'mandatory': {
              'OfferToReceiveAudio': true,
              'OfferToReceiveVideo': true
          }
      }
  };

  var call = peer.call(requestedPeer, stream, options);
  call.on('stream', function(remoteStream) {
    playStream(remoteStream);
  });
}, function(err) {
  console.log('Failed to get local stream' ,err);
});
{% endhighlight %}

After fixing this issue, I deployed this as a PHP application to Heroku, hoping to share the application with some friends and get their opinions on it. But because the free Peer server library in their documentation is `http` and therefore not SSL certified, trying to open the application through Heroku threw a secure server error.

I searched online to see if anyone else had created a secure Peer server and to my luck user [atskimura](https://github.com/atskimura/peerjs-server-heroku) on github did. So I linked it to create the server instead of building one myself from scratch to save some time and effort.


{% highlight javascript %}
var peer = new Peer({host: 'peerjs-server.herokuapp.com', port: 443, secure: true, key: 'peerjs'})
{% endhighlight %}

## Conclusion

There's not much else to say except that I'm really proud of this work. It was 100% conceived by me as a project and with a little bit of help seen through to what I wanted it to be. To be honest it was a lot of hard work and a lot of small problems not worth mentioning to iron out to get it to this point. And there are still bugs to work out as with any complex application, but it does what it needs to do and I'm really excited to share this with you guys.
