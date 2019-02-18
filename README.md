# Bootstrap Tourist

Quick and easy way to build your product tours with Bootstrap Popovers for Bootstrap 3 and 4.

## About Bootstrap Tourist
Bootstrap Tourist (called "Tourist" from here) is a fork of Bootstrap Tour, a plugin to create product tours.

The original Bootstrap Tour was written in coffeescript, and had a number of open feature and bug fix requests in the github repo. Bootstrap Tourist is an in-progress effort to move Bootstrap Tour to native ES6, fix some issues and add some requested features. You can read more about why Bootstrap Tourist exists, and why it's not a github fork anymore, here: https://github.com/sorich87/bootstrap-tour/issues/713

This repo has been created to give an easy way to update and track the revisions to the code, rather than filling up the original Tour forum.


Tourist automatically works with Bootstrap 3 and 4, however the "standalone" non-Bootstrap version is not available

## Getting started with Bootstrap Tourist
Simply include bootstrap-tourist.js and bootstrap-tourist.css into your page. A minified version is not provided because the entire purpose of this repo is to enable fixes, features and native port to ES6. If you are uncomfortable with this, please use the original Bootstrap Tour!

```html
<link href="bootstrap-tourist.css" rel="stylesheet">
....
<script src="bootstrap-tourist.js"></script>
```

Next, set up and start your tour with some steps:
```javascript
var tour = new Tour({
						framework: "bootstrap3",	// or "bootstrap4" depending on your version of bootstrap
						steps:
						[
							{
							element: "#my-element",
							title: "Title of my step",
							content: "Content of my step"
							},
							{
							element: "#my-other-element",
							title: "Title of my step",
							content: "Content of my step"
							}
						]
					});

// Start the tour - note, no call to .init() required
tour.start();
```

## Switching from Tour to Tourist
If you already have a working tour using Bootstrap Tour, and you want to move to Tourist (because it has some fixes etc), perform the following steps:

1. Copy over the Tourist CSS and JS, include them instead of Bootstrap Tour
2. If you are using Bootstrap 4, add the following option to your initialization code:
```framework: "bootstrap4"```
3. Remove the call to tour.init() - this is not required
4. Add a call to tour.start() to start the tour, and optionally add a call to tour.restart() to force restart the tour


## Basic Demo and Documentation
Because Tourist is based entirely on the original Bootstrap Tour, please see the demos and API documentation at the Bootstrap Tour website: [http://bootstraptour.com/api](http://bootstraptour.com/api)


# Documentation note
In bootstrap-tourist.js, you will find comprehensive documentation at the very top of the file. This is the most up to date and relevant docs, please look here first. The docs below in this readme are here for ease of reference only.


## Added features & fixes: Summary
The entire purpose of Tourist is to add features and fixes, and migrate to native ES6. Here is a list of all additional features and fixes in Tourist (compared to original Tour). All original Tour options are still available, unless superseded by the documentation below:

 1. onNext/onPrevious - prevent auto-move to next step, allow .goTo
 2. *** Do not call Tour.init *** - fixed tours with hidden elements on page reload
 3. Dynamically determine step element by function
 4. Only continue tour when reflex element is clicked using reflexOnly
 5. Call onElementUnavailable if step element is missing
 6. Scroll flicker/continual step reload fixed
 7. Magic progress bar and progress text, plus options to customize per step
 8. Prevent user interaction with element using preventInteraction
 9. Wait for arbitrary DOM element to be visible before showing tour step/crapping out due to missing element, using delayOnElement
 10. Handle bootstrap modal dialogs better - autodetect modals or children of modals, and call onModalHidden to handle when user dismisses modal without following tour steps
 11. Automagically fixes drawing issues with Bootstrap Selectpicker (https://github.com/snapappointments/bootstrap-select/)
 12. Call onPreviouslyEnded if tour.start() is called for a tour that has previously ended (see docs)
 13. Switch between Bootstrap 3 or 4 (popover template) automatically using tour options


## Added features & fixes: Documentation
Full feature documentation below. These supersede any features or structure required by Bootstrap Tour.


### Control flow from onNext() / onPrevious() options:
Returning false from onNext/onPrevious handler will prevent Tour from automatically moving to the next/previous step.
Tour flow methods (Tour.goTo etc) now also work correctly in onNext/onPrevious.
Option is available per step or globally:

```
var tourSteps = [
					{
						element: "#inputBanana",
						title: "Bananas!",
						content: "Bananas are yellow, except when they're not",
						onNext: function(tour){
							if($('#inputBanana').val() !== "banana")
							{
								// no banana? highlight the banana field
								$('#inputBanana').css("background-color", "red");
								// do not jump to the next tour step!
								return false;
							}
						}
					}
				];

var Tour=new Tour({
					steps: tourSteps,
					onNext: function(tour)
							{
								if(someVar = true)
								{
									// force the tour to jump to slide 3
									tour.goTo(3);
									// Prevent default move to next step - important!
									return false;
								}
							}
				});
```

### Do not call Tour.init if you previously used Bootstrap Tour
When setting up Tour, do not call Tour.init().
Call Tour.start() to start/resume the Tour from previous step
Call Tour.restart() to always start Tour from first step

Tour.init() was a redundant method that caused conflict with hidden Tour elements.


### Dynamically determine element by function
Step "element:" option allows element to be determined programmatically. Return a jquery object.
The following is possible:
```
var tourSteps = [
					{
						element: function() { return $(document).find("...something..."); },
						title: "Dynamic",
						content: "Element found by function"
					},
					{
						element: "#static",
						title: "Static",
						content: "Element found by static ID"
					}
				];
```


### Only continue tour when reflex element is clicked
Use step option reflexOnly in conjunction with step option reflex to automagically hide the "next" button in the tour, and only continue when the user clicks the element:
```
var tourSteps = [
					{
						element: "#myButton",
						reflex: true,
						reflexOnly: true,
						title: "Click it",
						content: "Click to continue, or you're stuck"
					}
				];
```


### Call function when element is missing
If the element specified in the step (static or dynamically determined as per feature #3), onElementUnavailable is called.
Function signature: function(tour, stepNumber) {}
Option is available at global and per step levels.

```
function tourBroken(tour, stepNumber)
{
	alert("Uhoh, tour element is done broke on step number " + stepNumber);
}

var tourSteps = [
					{
						element: "#btnMagic",
						onElementUnavailable: tourBroken,
						title: "Hold my beer",
						content: "now watch this"
					}
				];
```


### Scroll flicker / continue reload fixed
Original Tour constantly reloaded the current tour step on scroll & similar events. This produced flickering, constant reloads and therefore constant calls to all the step function calls.
This is now fixed. Scrolling the browser window does not cause the tour step to reload.

IMPORTANT: orphan steps are stuck to the center of the screen. However steps linked to elements ALWAYS stay stuck to their element, even if user scrolls the element & tour popover
			off the screen. This is my personal preference, as original functionality of tour step moving with the scroll even when the element was off the viewport seemed strange.



### Progress bar & progress text:
Use the following options globally or per step to show tour progress:
showProgressBar - shows a bootstrap progress bar for tour progress at the top of the tour content
showProgressText - shows a textual progress (N/X, i.e.: 1/24 for slide 1 of 24) in the tour title

```
var tourSteps = [
					{
						element: "#inputBanana",
						title: "Bananas!",
						content: "Bananas are yellow, except when they're not",
					},
					{
						element: "#inputOranges",
						title: "Oranges!",
						content: "Oranges are not bananas",
						showProgressBar: false,	// don't show the progress bar on this step only
						showProgressText: false, // don't show the progress text on this step only
					}
				];
				
var Tour=new Tour({
					steps: tourSteps,
					showProgressBar: true, // default show progress bar
					showProgressText: true, // default show progress text
				});
```


### Customize the progressbar/progress text:
In conjunction with previous, provide the following functions globally or per step to draw your own progressbar/progress text:

```
getProgressBarHTML(percent)
getProgressTextHTML(stepNumber, percent, stepCount)
```

These will be called when each step is shown, with the appropriate percentage/step number etc passed to your function. Return an HTML string of a "drawn" progress bar/progress text
which will be directly inserted into the tour step.

Example:
```
var tourSteps = [
					{
						element: "#inputBanana",
						title: "Bananas!",
						content: "Bananas are yellow, except when they're not",
					},
					{
						element: "#inputOranges",
						title: "Oranges!",
						content: "Oranges are not bananas",
						getProgressBarHTML:	function(percent)
											{
												// override the global progress bar function for this step
												return '<div>You be ' + percent + ' of the way through!</div>';
											}
					}
				];
var Tour=new Tour({
					steps: tourSteps,
					showProgressBar: true, // default show progress bar
					showProgressText: true, // default show progress text
					getProgressBarHTML: 	function(percent)
											{
												// default progress bar for all steps. Return valid HTML to draw the progress bar you want
												return '<div class="progress"><div class="progress-bar progress-bar-striped" role="progressbar" style="width: ' + percent + '%;"></div></div>';
											},
					getProgressTextHTML: 	function(stepNumber, percent, stepCount)
											{
												// default progress text for all steps
												return 'Slide ' + stepNumber + "/" + stepCount;
											},

				});
```



### Prevent interaction with element
Sometimes you want to highlight a DOM element (button, input field) for a tour step, but don't want the user to be able to interact with it.
Use preventInteraction to stop the user touching the element:

```
var tourSteps = [
					{
						element: "#btnMCHammer",
						preventInteraction: true,
						title: "Hammer Time",
						content: "You can't touch this"
					}
				];
```


### Wait for an element to appear before continuing tour
Sometimes a tour step element might not be immediately ready because of transition effects etc. This is a specific issue with bootstrap select, which is relatively slow to show the selectpicker
dropdown after clicking.
Use delayOnElement to instruct Tour to wait for **ANY** element to appear before showing the step (or crapping out due to missing element). Yes this means the tour step element can be one DOM
element, but the delay will wait for a completely separate DOM element to appear. This is really useful for hidden divs etc.
Use in conjunction with onElementUnavailable for robust tour step handling.

delayOnElement is an object with the following:
```
				delayOnElement: {
									delayElement: "#waitForMe", // the element to wait to become visible, or the string literal "element" to use the step element
									maxDelay: 2000, // optional milliseconds to wait/timeout for the element, before crapping out. If maxDelay is not specified, this is 2000ms by default
								}
```

Example:
```
var tourSteps = [
					{
						element: "#btnPrettyTransition",
						delayOnElement:	{
											delayElement: "element" // use string literal "element" to wait for this step's element, i.e.: #btnPrettyTransition
										},
						title: "Ages",
						content: "This button takes ages to appear"
					},
					{
						element: "#inputUnrelated",
						delayOnElement:	{
											delayElement: "#divStuff" // wait until DOM element "divStuff" is visible before showing this tour step against DOM element "inputUnrelated"
										},
						title: "Waiting",
						content: "This input is nice, but you only see this step when the other div appears"
					},
					{
						element: "#btnDontForgetThis",
						delayOnElement:	{
											delayElement: "element", // use string literal "element" to wait for this step's element, i.e.: #btnDontForgetThis
											maxDelay: 5000	// wait 5 seconds for it to appear before timing out
										},
						title: "Cool",
						content: "Remember the onElementUnavailable option!",
						onElementUnavailable: 	function(tour, stepNumber)
												{
													// This will be called if btnDontForgetThis is not visible after 5 seconds
													console.log("Well that went badly wrong");
												}
					},
				];
```


### Trigger when modal closes
If tour element is a modal, or is a DOM element inside a modal, the element can disappear "at random" if the user dismisses the dialog.
In this case, onModalHidden global and per step function is called. Only functional when step is not an orphan.
This is useful if a tour includes a step that launches a modal, and the tour requires the user to take some actions inside the modal before OK'ing it and moving to the next
tour step.

- Return (int) step number to immediately move to that step
- Return exactly false to not change tour state in any way - this is useful if you need to reshow the modal because some validation failed
- Return anything else to move to the next step

element === Bootstrap modal, or element parent === bootstrap modal is automatically detected.

```
var Tour=new Tour({
					steps: tourSteps,
					onModalHidden: 	function(tour, stepNumber)
									{
										console.log("Well damn, this step's element was a modal, or inside a modal, and the modal just done got dismissed y'all. Moving to step 3.");

										// move to step number 3
										return 3;
									},
				});


var Tour=new Tour({
					steps: tourSteps,
					onModalHidden: 	function(tour, stepNumber)
									{
										if(validateSomeModalContent() == false)
										{
											// The validation failed, user dismissed modal without properly taking actions.
											// Show the modal again
											showModalAgain();

											// Instruct tour to stay on same step
											return false;
										}
										else
										{
											// Content was valid. Return null or do nothing to instruct tour to continue to next step
										}
									},
				});
```


### Handle Dialogs and BootstrapDialog plugin better (https://nakupanda.github.io/bootstrap3-dialog/)
Plugin makes creating dialogs very easy, but it does some extra stuff to the dialogs and dynamically creates/destroys them. This
causes issues with plugins that want to include a modal dialog in the steps using this plugin.

To use Tour to highlight an element in a dialog, just use the element ID as you would for any normal step. The dialog will be automatically
detected and handled.

To use Tour to highlight an entire dialog, set the step element to the dialog div. Tour will automatically realize this is a dialog, and
shift the element to use the modal-content div inside the dialog. This makes life friendly, because you can do this:

```
<div class="modal" id="myModal" role="dialog">
	<div class="modal-dialog">
		<div class="modal-content">
		...blah...
		</div>
	</div>
</div>
```

Then use element: myModal in the Tour.


FOR BOOTSTRAPDIALOG PLUGIN: this plugin creates random UUIDs for the dialog DOM ID. You need to fix the ID to something you know. Do this:

```
dlg = new BootstrapDialog.confirm({
									....all the options...
								});

// BootstrapDialog gives a random GUID ID for dialog. Give it a proper one
$objModal = dlg.getModal();
$objModal.attr("id", "myModal");
dlg.setId("myModal");
```

Now you can use element: myModal in the tour, even when the dialog is created by BootstrapDialog plugin.



###	Fix conflict with Bootstrap Selectpicker: https://github.com/snapappointments/bootstrap-select/
Selectpicker draws a custom select. Tour now automagically finds and adjusts the selectpicker dropdown so that it appears correctly within the tour



### Call onPreviouslyEnded if tour.start() is called for a tour that has previously ended
See the following github issue: https://github.com/sorich87/bootstrap-tour/issues/720
Original behavior for a tour that had previously ended was to call onStart() callback, and then abort without calling onEnd(). This has been altered so
that calling start() on a tour that has previously ended (cookie step set to end etc) will now ONLY call onPreviouslyEnded().

This restores the functionality that allows app JS to simply call tour.start() on page load, and the Tour will now only call onStart() / onEnd() when
the tour really is started or ended.

```
var Tour=new Tour({
					steps: [ ..... ],
					onPreviouslyEnded: 	function(tour)
										{
											console.log("Looks like this tour has already ended");
										},
				});

Tour.start();
```


### Switch between Bootstrap 3 or 4 (popover template) automatically using tour options, or use a custom template
With thanks to this thread: https://github.com/sorich87/bootstrap-tour/pull/643

Tour is compatible with bootstrap 3 and 4 if the right template is used for the popover. To select the correct template, use the "framework" global option.

```
var Tour=new Tour({
					steps: tourSteps,
					template: null,			// template option is null by default, but MUST BE SET TO NULL to use the framework option
					framework: "bootstrap4" // can be string literal "bootstrap3" or "bootstrap4"
				});
```

To use a custom template, use the "template" global option:

```
var Tour=new Tour({
					steps: tourSteps,
					template: '<div class="popover" role="tooltip">....blah....</div>'
				});
```

Review the following logic:
- If neither template nor framework options are specified, bootstrap3 compatibility is used by default
- If template == null, framework option is used
- If template != null, template is always used
- If template == null, and framework option is not literal "bootstrap3" or "bootstrap4", error will occur

To add additional templates, search the code for "PLACEHOLDER: TEMPLATES LOCATION". This will take you to an array that contains the templates, simply edit
or add as required.



## Contributing
Feel free to contribute with pull requests, bug reports or enhancement suggestions.


## License

Code licensed under the [MIT license](https://opensource.org/licenses/MIT).
Documentation licensed under [CC BY 3.0](http://creativecommons.org/licenses/by/3.0/).
