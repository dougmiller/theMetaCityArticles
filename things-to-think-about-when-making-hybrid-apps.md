# Things to think about when making hybrid apps

## Specifically Cordova and Ionic

Developing hybrid mobile apps can seem like a nice solution to developing applications for multiple platforms and for the most part it is. There are however a couple of things that you should be aware of which will make your life easier.

1. The developer ecosystem is not quite as mature as desktop environments.
Plugins
Building

2. The software running on mobile devices is not quite as mature as their desktop equilivants
I am looking at you iOS. Storage of large data is the latest issue I ran into with 
Error reporting (white screen of death)

3. Security/app lifecycles policy differs per vendor
The way that each vendor handles sleep, wake, suspend etc state for their devices means that you may have to put in special cases for each one. The latest issue I ran into was GPS reportings.


## Things you should probably be doing

### Have a fast feedback loop
Develop on the desktop with a live reload server going. This will catch the most egrrarious of errors you make as well as have access to the console for fast (and early) reporting of errors. After every significant build (say a branch check in) deploy to a device. This last step is super helpful as the error reporting from devices is not very helpful (white screen of death, black screen of death and not captuing console output until late in the startup of the app). The sooner you can get the app onto a real device with real testing, the better. This is especially helpful if you are usign features that may or may not work as intended.


### Optimise your workflow
My development workflow generally goes as following:
Have a live reload server running in a terminal. You can set this to also watch the SASS/Less files or run a watcher for that in a separate terminal.
Run the app in Chrome(ium) with the mobile device inspector turned on. This will be your primary feedback window.
Have the source opened in your favourite editor/IDE (I use IntelliJ WebStorm for this). This is obviously where you will edit the source and make the changes needed. Because of the live reload server that is on, any changes you make in here will cause the browser to reload the page (losing/resetting the state).
Have a second/different browser window open for docs etc.
Having a second monitor with the app browser in it reducing the context switching needed for fast iterations.
Have a device (or two) plugged in that you can deploy to quickly (see the button box for an example of some more context switching savings) for fast testing on devices.

