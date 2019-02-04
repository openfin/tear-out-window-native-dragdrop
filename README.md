# Tear Out Window Native Dragdrop Example

## Overview
Tear out windows with native drag and drop on OpenFin

This approach uses native HTML5 drag and drop.  Since drop can happen outside of the window, we need to create a small and transparent child window as drop target.   This child window follows mouse movement so it can capture drop events.  

### Guidelines
1. Directive for draggable component in main window

 ```javascript
angular.module("MyApp").directive("draggableTab", function() {
    return {
        restrict: 'A',
        link: function(scope, el, attrs) {
            el.attr('draggable', 'true');
            el.bind('dragstart', function(event) {
                event.dataTransfer.setData("draggedID", someID);  // pass custom data in drag event
                event.dataTransfer.effectAllowed = "copy";
                $rootScope.$emit('dragdrop', {action:”dragstart” });
            });
            el.bind('drag', function(event) {
                $rootScope.$emit('dragdrop', {action:”dragging” });
            });
            el.bind('dragend', function(event) {
                $rootScope.$emit('dragdrop', {action:”dragend” });
            });
        }
    };
});

 ```

2. Create child window as drop target

 ```javascript
            dropTargetWindow = new fin.desktop.Window({
                name: 'dropTarget',
                url: "dropTarget.html",
                defaultWidth: 20,	// small
                defaultHeight: 20,
                defaultTop: 0,
                defaultLeft: 0,
                frame: false,
                resizable: false,
                autoShow: false,	           // created as hidden
                alwaysOnTop: false,
                showTaskbarIcon: false,   // required so hidden from task bar
                drag: false,
                shadow: false,
                opacity: 0.1	// transparent, kinda
            }, function () {
            });
 ```

3. Manage position of child window

 ```javascript
            $rootScope.$on("'dragdrop'", function (event, eventObj) {
                    if (eventObj.action === "dragstart") {
                        stopPosSync();
                    }
                    else if (eventObj.action === "dragging") {
                        if (!tabDragStop) {
                           startPosSync();
                        }
                        fin.desktop.System.getMousePosition(function (evt) {
                            var y = parseInt(evt.top);
                            var x = parseInt(evt.left);
                            dropTargetWindow.moveWindowTo(x - 10, y - 10);

                            fin.desktop.Window.getCurrent().getBounds(function (bounds) {
                                if (!inBounds(x, y, bounds)) {
			// if mouse is outside of main window, bring the child window in front of all windows so it can capture
			// drop events
                                    dropTargetWindow.bringToFront();  
                                } else {
                                    fin.desktop.Window.getCurrentWindow().bringToFront();
                                }
                            });
                        });
                    }
                    else if (eventObj.action === "dragend") {
                        if (eventObj.jid) {
                            stopPosSync();
                        }
                    }
                    else if (eventObj.action === "drop") {
                            fin.desktop.System.getMousePosition(function (evt) {  // get current mouse position of drop event
                                var y = parseInt(evt.top);
                                var x = parseInt(evt.left);
                                // check if mouse position is outside of bounds of current window
                                fin.desktop.Window.getCurrent().getBounds(function (bounds) {
                                    if (!inBounds(x, y, bounds)) {
                                        var initBounds = {top: y, left: x};
                                       createChildWindow(initBounds);  // create new window at the position of drop event
                                    }  
                                });
                            });
                        }
                }
            });
        },
        function startPosSync() {   // start making dropTargetWindow follow mouse movement
            dropTargetWindow.show();   // show dropTargetWindow
            chatTabDragWindow.resizeTo(20, 20, "top-left");   // make it small
	// refresh position of dropTargetWindow on a timer
            dragTimer = $interval(function() {
                    fin.desktop.System.getMousePosition(function (evt) {
                        var y = parseInt(evt.top);
                        var x = parseInt(evt.left);
                        dropTargetWindow.moveTo(x-10, y-10);
                    });
            }, 300);
        },
        stopPosSync: function() {  // stop tracking mouse position
            if (dragTimer) {
                $interval.cancel(dragTimer);
                dragTimer = undefined;
            }
            dropTargetWindow.hide();  // hide the child window
        },

 ```

4. dropTarget.html

 ```html
        <div class="wrapper" ng-cloak drop-zone>

 ```

5. drop-zone directive in the drop target child window

In order to notify the parent window when drop is detected, this child window needs to have access to rootScope of the  parent window.

 ```javascript
angular.module("MyApp").directive("dropZone", function($rootScope, OpenFinAdapter) {
    return {
        restrict: 'A',
        link: function(scope, el, attrs) {
            el.bind('dragover', function(event) {
                event.preventDefault();
            });
            el.bind('drop', function(event) {
		// notify parent window about the drop event via rootScope of parent window
                    parentRootScope.$emit(“dragdrop”, {action: 'drop'});
                    event.preventDefault();
            });
        }
    };
});

 ```

## Disclaimers
* This is a starter example and intended to demonstrate to app providers a sample of how to approach an implementation. There are potentially other ways to approach it and alternatives could be considered. 
* This is an open source project and all are encouraged to contribute.
* Its possible that the repo is not actively maintained.

## License
MIT


The code in this repository is covered by the included license.

However, if you run this code, it may call on the OpenFin RVM or OpenFin Runtime, which are covered by OpenFin’s Developer, Community, and Enterprise licenses. You can learn more about OpenFin licensing at the links listed below or just email us at support@openfin.co with questions.

https://openfin.co/developer-agreement/ <br/>
https://openfin.co/licensing/

## Support
Please enter an issue in the repo for any questions or problems. 
<br> Alternatively, please contact us at support@openfin.co
