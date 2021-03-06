Simple GUI toolkit with go
------------------------------

Install
===========================

You will need to install gtk-dev files.

`go get github.com/zozor/hgui`


The idea
===========================

This is a gui toolkit that relies on HTML, CSS, and javascript. But as a user of this toolkit you do not have to worry about that
although it helps understanding it.

The compiled program will, when you run it, start a server at 127.0.0.1:randomport, and connect to it with gtk-webkit.

To make a simple interface, no knowledge of html, javascript or css is needed. For styling some CSS knowledge is needed, but not much.

Here is a simple program. An advanced one can be found in examples


	package main

	import "github.com/zozor/hgui"

	func main() {
		label := hgui.NewLabel("Simple label")
		input := hgui.NewTextinput("", hgui.TextType_Text)
		hgui.Topframe.Add(
			input, 
			hgui.NewButton("New Text!", nil, func() {
				label.SetValue(input.Value())
				label.SetStyle(hgui.Style{"color":"blue"})
			}),
			hgui.Html("<br/>"), 
			label,
		)
		hgui.StartServer(800, 600, "Simple program!")
	}

How it works
===========================
We start with server for clarity

	StartServer(width, height, title)

Starts a http server, creates a window with gtk, adds webkit to that window and make webkit connect to the server.

The two first lines in main() creates two widgets for use.

	hgui.Topframe.Add(...HTMLer)

Adds widgets between `<body></body>` in the outputtet html. We first add `input` to the body.
Then we make a button widget with an onclick event. `HTMLer` is an 

	type interface HTMLer {
		HTML() string
	}

When this button is created, it puts the function in a `map[id.onclick]func()`, puts javascript on the button in webkit
`onclick="callhandler(id.onclick)"`. An ajax query is then sent, when button is clicked, with the ID to call the function specified in the map.

	label.SetValue(value)

This sends a javascript event to webkit. All events are send through a bufferede channel with some javascript code to run and a reply channel.
This can also be done using the function `SendEvent(javascript, replychannel)`. But the widgets do this for you.

This event channel is emptied by webkit 100 times a second, and runs the javascript inside them in the order they came in.

	input.GetValue()

This will use the replychannel, the SetX methods has a nil reply channel. Events that require a reply have to include the variable `reply` in the javascript. So the events look like this `SendEvent("reply = ...", replychannel)`. Webkit runs the javascript, and returns a `String(reply)` to
the return channel. In the package's various Value() methods, it usually looks like this

	reply := make(chan string)
	events <- Event("reply = ...", reply)
	return <-reply

And ofcourse, jquery is part of the package, making everything extremely easy.

Features
===========================
#### Widgets

- Frames / Container
- Tables
- Radio and Check boxes
- Fieldset
- Labels
- Text input
- Textarea
- Selectform
- Buttons
- Links
- Images
- Modal Dialogs
- Gauge

#### Other
- Styling with css
- Raw javascript
- Resources (it actually does not allow anything else)
- Everything that can be made in html/css/javascript can be used here. Making it somewhat more powerfull than GTK?.

Contribution - Help me out!
===============================
If you are badass to css, javascript and html, please contribute, as we need more widgets.
If you do not want to work with the internals of the package, you can just send the html/css/js and i will make it happen.

If you got any ideas for widgets, API change, design or whatever, please speak your mind.

All help and contribution is appriciated :)

Issues
===========================
Gtk sucks. It spams my face with errors.

I can remove the 100 requests per second and deal with events on time if i could get this to work

	gchar* k = "some code;";
	webkit_web_view_execute_script(WEBKIT_WEB_VIEW(webview), k);

This call i supposed to run the script in `k` in webkit. I could make it run `GetEvents()`, which is now being called
with `setInterval()` to fetch the events. But it crashes all the time. And it also seems that `GetEvents` or 
jquery object is not part of the scope.. I do not know what to do here.. I hate GTK.