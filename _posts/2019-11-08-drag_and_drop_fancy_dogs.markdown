---
layout: post
title:      "Drag & drop fancy dogs"
date:       2019-11-08 17:23:06 +0000
permalink:  drag_and_drop_fancy_dogs
---


For this project, I took a more lighthearted approach and made an app where you can dress up dogs with different accessories. This required a way to drag images of dog accessories on top of an image of a dog, and "drop" them there so it looks like you dressed the dog. 

To create this drag and drop functionality, I wrote three Javascript functions: 

1. This allows the parent div to have items "dropped" into it:
```
function allowDrop(ev) {
    ev.preventDefault();
}
```

2. This stores the id of the image. We'll use that data to grab the element later and append it to the new location.
```  
function drag(ev) {
    ev.dataTransfer.setData("text", ev.target.id);
}
```

3. This defines what happens when an image is "dropped" into the new location. The "data" variable is set to the dropped image's id. Next, if the image was dropped onto the main image of the dog (or onto an image that was already dropped on to the dog), add it to the container for selected accesories. Otherwise, put it back in the container for the accessories that haven't been selected.
``` 
function drop(ev) {
    ev.preventDefault();
    let data = ev.dataTransfer.getData("text");
    if (ev.target.id === "accessory-selected" || ev.target.parentElement.id === "accessory-selected" ) {
        document.getElementById("accessory-selected").appendChild(document.getElementById(data));
    } else {
        document.getElementById("accessory").appendChild(document.getElementById(data));
    }
}
```

Next, I added these functions to the html elements. The `accessory` div is where unselected accessories are stored. The `ondragover` and `ondrop` properties are set to `allowDrop()` and `drop()` so images can be dragged into that div. The images are set to be `draggable`, and the `drag()` function happens when a drag event is started.

```
<div id="accessory" ondrop="drop(event)" ondragover="allowDrop(event)">
    <img class="card" src="./images/monacle.png" draggable="true" ondragstart="drag(event)" id="monacle">
    <img class="card" src="./images/glasses.png" draggable="true" ondragstart="drag(event)" id="glasses">
    ...
</div>										
```

The `accessory-selected` div stores the accessories that overlap the main dog image. I also added the `ondrop` and `ondragover` properties to this div, so images can be dragged to and from this div as well.

```
<div id="accessory-selected" ondrop="drop(event)" ondragover="allowDrop(event)"></div>
```

