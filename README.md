#A NativeScript plugin for Meteor DDP Client Applications

##Install 

`tns plugin add https://github.com/triniwiz/nativescript-ddp-client`


##Authentication

DDP Authentication is implemented by [triniwiz/nativescript-ddp-login](https://github.com/triniwiz/nativescript-ddp-login)

## Usage

For connection with Meteor server on local machine use:
adb reverse tcp:3000 tcp:3000


```js
var DDPClient = require('nativescript-ddp-client');

var ddpclient = new DDPClient({
  // All properties optional, defaults shown
  host : "localhost",
  port : 3000,
  ssl  : false,
  autoReconnect : true,
  autoReconnectTimer : 500,
  maintainCollections : true,
  ddpVersion : '1',  // ['1', 'pre2', 'pre1'] available
  // uses the SockJs protocol to create the connection
  // this still uses websockets, but allows to get the benefits
  // from projects like meteorhacks:cluster
  // (for load balancing and service discovery)
  // do not use `path` option when you are using useSockJs
  useSockJs: true,
  // Use a full url instead of a set of `host`, `port` and `ssl`
  // do not set `useSockJs` option if `url` is used
  url: 'wss://localhost/websocket'
});

/*
 * Connect to the Meteor Server
 */
ddpclient.connect((error, wasReconnect)=>{
  // If autoReconnect is true, this callback will be invoked each time
  // a server connection is re-established
  if (error) {
    console.log('DDP connection error!');
    return;
  }

  if (wasReconnect) {
    console.log('Reestablishment of a connection.');
  }

  console.log('connected!');

  setTimeout(()=> {
    /*
     * Call a Meteor Method
     */
    ddpclient.call(
      'deletePosts',             // name of Meteor Method being called
      ['foo', 'bar'],            // parameters to send to Meteor Method
      (err, result) =>{   // callback which returns the method call results
        console.log('called function, result: ' + result);
      },
      () =>{              // callback which fires when server has finished
        console.log('updated');  // sending any updated documents as a result of
        console.log(ddpclient.collections.posts);  // calling this method
      }
    );
  }, 3000);


  /*
   * Subscribe to a Meteor Collection
   */
  ddpclient.subscribe(
    'posts',                  // name of Meteor Publish function to subscribe to
    [],                       // any parameters used by the Publish function
    () =>{             // callback when the subscription is complete
      console.log('posts complete:');
      console.log(ddpclient.collections.posts);
    }
  );

  /*
   * Observe a collection.
   */
  var observer = ddpclient.observe("posts");
  observer.added = (id) =>{
    console.log("[ADDED] to " + observer.name + ":  " + id);
  };
  observer.changed = (id, oldFields, clearedFields, newFields) =>{
    console.log("[CHANGED] in " + observer.name + ":  " + id);
    console.log("[CHANGED] old field values: ", oldFields);
    console.log("[CHANGED] cleared fields: ", clearedFields);
    console.log("[CHANGED] new fields: ", newFields);
  };
  observer.removed = (id, oldValue)=> {
    console.log("[REMOVED] in " + observer.name + ":  " + id);
    console.log("[REMOVED] previous value: ", oldValue);
  };
  setTimeout(() =>{ observer.stop() }, 6000);
});

/*
 * Useful for debugging and learning the ddp protocol
 */
ddpclient.on('message',(msg)=> {
  console.log("ddp message: " + msg);
});

/*
 * Close the ddp connection. This will close the socket, removing it
 * from the event-loop, allowing your application to terminate gracefully
 */
ddpclient.close();

/*
 * If you need to do something specific on close or errors.
 * You can also disable autoReconnect and
 * call ddpclient.connect() when you are ready to re-connect.
*/
ddpclient.on('socket-close', (code, message) =>{
  console.log("Close: %s %s", code, message);
});

ddpclient.on('socket-error', (error) =>{
  console.log("Error: %j", error);
});

/*
 * You can access the EJSON object used by ddp.
 */
var oid = new ddpclient.EJSON.ObjectID();
```




