---
layout: post
title: BlocChat
thumbnail-path:
short-description: BlocChat is a chat app with the ability to send, receive and view messages in real time.
---

{:.center}

## Explanation

BlocChat is a chat room application built with Angular, using both Angular UI-Router and UI-Bootstrap, and styled with Bootstrap. While simple in design, it functions as well as any other chatroom should, giving the user the ability to set a username, create a new chatroom, and send messages that appear in the chatroom as soon as they're sent, storing all chatrooms and any messages contained in Google Firebase and updating the database in real-time.

## Problem Solving

Having no prior experience using UI-Bootstrap, using the UI-bootstrap modal directive correctly was a challenge. I had already designed the new chat modal as part of the page to make sure that the code itself would create and push the new chatroom onto Firebase. But when I introduced the UI-Bootstrap modal coding into the app, the page wouldn't load at all. This was because the controller used for the new chatroom modal contained two different controllers as shown in the documentation on the UI-Bootstrap page.

{% highlight javascript %}

  angular
    .module('blocChat')
    .controller('AddRoomCtrl', ['$uibModal', 'Room', '$uibModalInstance', ModalCtrl]);
    .controller('OpenRoom', ['$uibModal', OpenModal()]

{% endhighlight %}

I had not realized that repeating two different controllers in one file would throw such an error when using UI-Router, because I believed that the documentation online was the correct format. After some research online, I understood that you couldn't have two controllers defined on one page and I separated both controllers into two different files.

After this initial problem had been resolved, I realized that the form for creating a new room wouldn't pop-up when the "New Chat" button was clicked. It was not because of the chatroom form and any of its submit functions, as those had all been written and tested before implementing the UI-Bootstrap modal directive at all. I threw in a `console.log()` into the modal opening function to figure out if the snippet of code was actually being called, which it wasn't.

Because I referenced the modal opening function in the new chat modal controller instead of in the home controller, which was linked to the home.html page where the button was located, the function was not actually being called. I attempted to bridge that gap by linking the button itself to the modal controller using ng-controller which threw an Unknown Provider error as pages cannot have two different controllers linked to them. I attempted to remedy this by creating a modal state within the app.js file, which was much more complicated of a task to do then what I had initially thought, especially with the amount of AngularJS, Angular UI-Router, and UI-Boostrap knowledge that I had at the time.

After fiddling around with the state code for twenty minutes, I was beginning to get frustrated. I decided to reset the code to the time before I started making adjustments in order to have a clean slate and approach the problem from a different angle.
In the end I moved the modal opening function into the home controller like so:

{% highlight javascript %}
(function() {
    function HomeCtrl(Room, $uibModal, Message) {
      this.open = function () {
        $uibModal.open({
          templateUrl: '/templates/room_modal.html',
          controller: 'AddRoomCtrl',
          controllerAs: '$ctrl'
        });
      };

    angular
        .module('blocChat')
        .controller('HomeCtrl', ['Room','$uibModal','Message', HomeCtrl]);
})();
});
{% endhighlight %}

This resulted in the successful modal opening.

(img)

## Results

After the initial debacle with UI-Bootstrap, building the rest of the application itself went rather smoothly. The username modal, which defined and kept the user's preferred name using Cookies and would pop-up when the page loaded, was much easier to define and create knowing how to use the UI-Bootstrap modal.

Bootstrap was used as the main CSS library with a couple of changes here and there in order to fit the simple chatroom style that I preferred. There's a sidebar for navigation in off-black that lists the name of the site, "Bloc Chat", and the names of the different chat rooms in white. The whitespace to the sidebar's right is where the messages are shown, with the user ID and the timestamp of when the message is sent. At the bottom of the whitespace, is the message bar where you can type and send messages in real time. After submission, the new message shows up at the bottom of the message list in the chat room.

(img)

## Conclusion

BlocChat was an excellent project to explore and experiment with AngularJS, UI-Router, and UI-Bootstrap. I didn't know what to expect with this project going in, since I was using so many different and new components, like the Firebase database and ng-Cookies. To be honest I never expected to have so much trouble with just the UI-Bootstrap modal; I expected to struggle all around but not so specifically with just UI-Bootstrap.

The documentation for the UI-Bootstrap modal seemed clear and simple but it didn't translate over into my app quite as directly as I thought it would. It made me learn that I should analyze how to incorporate the documented code into my code based on how I want my code to work and not how the code works in the documentation.
