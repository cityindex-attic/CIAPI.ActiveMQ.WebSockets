#CIAPI.ActiveMQ websockets

##Server

###Local install:
download and unzip activemq 5.4.2 to a folder (we will assume that be for ACTIVEMQ_HOME for the rest of the documentation)

####Enable websocket connector

Open the file
$ACTIVEMQ_HOME/conf/activemq.xml

Locate the transportConnectors element and add the following 

    <transportConnector name="websocket" uri="ws://0.0.0.0:80"/>

This will expose the websocket interface on port 80.

Copy the camel-freemarker & freemarker jars to $ACTIVEMQ_HOME/lib folder

Copy the stock-quote.json and news-quote.json to the folder $ACTIVEMQ_HOME/conf/templates

####Set up camel routes to publish messages

Open the file

$ACTIVEMQ_HOME/conf/camel.xml

Add the following inside the camelContext element

    <route id="stock-quote">
    	<from uri="timer:stock-trigger?period=1000"/>
    	<to uri="freemarker:templates/stock-quote.json"/>
    	<to uri="activemq:topic:stock-quote"/>	
    </route>
    
    <route id="news-quote">
    	<from uri="timer:news-trigger?period=1000"/>
    	<to uri="freemarker:templates/news-quote.json"/>
    	<to uri="activemq:topic:news-quote"/>	
    </route>


The above routes  setup a timer that fires every second which then invokes the freemarker component to load 
and process a template. The output of it is sent to the specified topic.


The final version of activemq.xml & camel.xml are also included along with this documentation.

####Start activemq

$ACTIVEMQ_HOME/bin/activemq start

If having trouble starting activemq; try $ACTIVEMQ_HOME/bin/activemq console for more detail

To start / stop the routes (to enable / disable triggering background publishing) go the camel page
http://localhost:8161/camel/routes

And then invoke the start / stop for the specific route (stock-quote / news-quote)

##Client
###CS sample client - 
https://github.com/cityindex/CIAPI.CS/tree/master/src/StreamingClient/Websocket

Can be used as:

        [Test, Category("DependsOnExternalResource")]
        public void CanConnectToExternal()
        {
        		_logger.InfoFormat("Ready to subscribe");
        		var stompMessages = new List<StompMessage>();
        		using (var stomp = new StompOverWebsocketConnection(
        			new Uri("ws://==ActiveMQUrl==:80")))
        		{
        			stomp.Connect("", ""); 
        			stomp.Subscribe("/topic/mock.news");
        			for (var i = 0; i < 3; i++)
        			{
        				stompMessages.Add(stomp.WaitForMessage());
        			}
        			stomp.Unsubscribe("/topic/mock.news");
        		}
        
        		Assert.AreEqual(3, stompMessages.Count);
        }

###Javascript sample client (works in Chrome; other browsers not so much)
See Client.Chrome.html for full sample

    var socket, client, 
    host = "ws://ec2-50-17-7-70.compute-1.amazonaws.com", 
    login = '', 
    passcode = '', 
    destination = '/topic/mock.news';
    
    $(document).ready(function() {
    	log("Initialising websocket connection to " + host);
    	$('#ws_host').text(host);
    
    	client = Stomp.client(host);
    
    	// this allows to display debug logs directly on the web page
    	client.debug = function(str) {
    		log(str);
    	};
    	// the client is notified when it is connected to the server.
    	var onconnect = function(frame) {
    		debug("connected to Stomp");
    
    		client.subscribe(destination, function(message) {
    			log(message);
    		});
    	};
    	client.connect(login, passcode, onconnect);
    
    });