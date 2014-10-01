# KazooCon demo app
The goal of this document is to assist you in following the "Build your first Monster app" talk

### Launch the Monster
First of all, go at the URL where your Monster-UI is hosted, and check that it actually renders something... It should display the main banner, but an empty white content...

### Copy and paste of the skeleton app
The content isn't filled because the UI is trying to start the `demo` app, which you currently don't have on your system... And that's the goal of this demo!

So the first thing to do is to go in your Monster-UI folder, then go in the `apps` folder. You need to have a `demo` folder in order for Monster-UI to load your app properly. To do so, you'll copy the `skeleton` folder available at `%monsterDir%/apps/skeleton` and paste it under the `apps` folder. Rename the skeleton copy folder to `demo` and finally reload the Monster-UI website: it should load the app and say Welcome!

Once this is done, you're running the demo app! But let's try to customize it and use some of the Monster-UI features now :)

At this point, wait until the presenter asks you to copy and paste this code before playing with it!

### HTML
`/apps/demo/views/layout.html`
```html
<div id="demo_wrapper">
	<div class="demo-content row-fluid">
		<div class="span8 offset2">
			<div class="span12">
				<div class="well title-container">
					<h1>{{ i18n.demo.welcome }}</h1>
				</div>

				<div class="row-fluid">
					<div class="span4">
						<h4>{{ i18n.demo.listDevices }}</h4>
						<ul class="list-devices">
							{{#each registeredDevices}}
								<li class="device-item" data-id="{{ id }}">
									{{ name }}
								</li>
							{{/each}}
						</ul>
					</div>

					<div class="span8">
						<h4>{{ i18n.demo.listEvents }}<button class="btn btn-primary" type="button" id="clearEvents">{{ i18n.demo.clearEvents }}</button></h4> 
						<table class="table table-condensed list-events">
							<thead>
								<tr>
									<th>{{ i18n.demo.table.eventType }}</th>
									<th>{{ i18n.demo.table.caller }}</th>
									<th>{{ i18n.demo.table.callee }}</th>
									<th>{{ i18n.demo.table.time }}</th>
								</tr>
							</thead>
							<tbody>
								<tr class="no-events">
									<td colspan="4">{{ i18n.demo.table.noEvents }}</td>
								</tr>
							</tbody>
						</table>
					</div>
				</div>
			</div>
		</div>
	</div>
</div>
```

### I18n
`/apps/demo/i18n/en-US.json`
```json
{
	"demo": {
		"welcome": "Welcome to our KazooCon Demo App",
		"listDevices": "Registered Devices",
		"listEvents": "List of live Events",
		"clearEvents": "Clear Events",
		"events": {
			"CHANNEL_CREATE": "Placing call...",
			"CHANNEL_ANSWER": "Call picked up...",
			"CHANNEL_DESTROY": "Hangup"
		},
		"table": {
			"caller": "Caller",
			"callee": "Callee",
			"eventType": "Event Type",
			"time": "Time",
			"noEvents": "Nothing happened for now..."
		}
	}
}
```

### Sockets events
inside `/apps/demo/app.js`
```javascript
bindEvents: function(template, globalData) {
	var self = this,
		addEvent = function(data) {
			console.log(data);
			var formattedEvent = self.formatEvent(data),
				eventTemplate = monster.template(self, 'event', formattedEvent);

			if(formattedEvent.extra.deviceId && formattedEvent.extra.deviceId in globalData.registeredDevices) {
				monster.ui.fade(template.find('.device-item[data-id="'+ formattedEvent.extra.deviceId +'"]'));
			}

			template.find('.list-events tbody').prepend(eventTemplate);
		};

	// subscribe to call events
	monster.socket.emit("subscribe", { account_id: self.accountId, auth_token: self.authToken, binding: "call.CHANNEL_CREATE.*"});
	monster.socket.emit("subscribe", { account_id: self.accountId, auth_token: self.authToken, binding: "call.CHANNEL_ANSWER.*"});
	monster.socket.emit("subscribe", { account_id: self.accountId, auth_token: self.authToken, binding: "call.CHANNEL_DESTROY.*"});

	// Bind some js code to the reception of call events
	monster.socket.on("CHANNEL_CREATE", function (data) {
		addEvent(data);
	});

	monster.socket.on("CHANNEL_ANSWER", function (data) {
		addEvent(data);
	});

	monster.socket.on("CHANNEL_DESTROY", function (data) {
		addEvent(data);
	});

	template.find('#clearEvents').on('click', function() {
		template.find('.table tbody tr:not(.no-events)').remove();
	});
},
```

### Formatting Event data
inside `/apps/demo/app.js`
```javascript
// Formatting data
formatEvent: function(data) {
	var self = this,
		formattedData = data;

	formattedData.extra = {};

	formattedData.extra.to = data['To'].substr(0, data['To'].indexOf('@'));
	formattedData.extra.friendlyEvent = self.i18n.active().demo.events[data['Event-Name']];
	formattedData.extra.classEvent = data['Event-Name'] === 'CHANNEL_CREATE' ? 'info' : (data['Event-Name'] === 'CHANNEL_ANSWER' ? 'success' : 'error');

	if('Custom-Channel-Vars' in data && 'Authorizing-Type' in data['Custom-Channel-Vars'] && data['Custom-Channel-Vars']['Authorizing-Type'] === 'device') {
		formattedData.extra.deviceId = data['Custom-Channel-Vars']['Authorizing-ID'];
	}

	return formattedData;
}
```