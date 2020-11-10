# Nats

1. NatsProtoo

   1. NewNatsProtoo

      ```go
      type NatsProtoo struct {
      	emission.Emitter
      	nc                 *nats.Conn
      	mutex              *sync.Mutex
      	subj               string
      	closed             bool
      	requestListener    map[string]RequestFunc
      	broadcastListeners map[string][]BroadCastFunc
      }	
      ```

   2. Set NatsProtoo.closed true

   3. Create Nats Connect Options (slice)

      ```go
      // Option is a function on the options for a connection.
      type Option func(*Options) error
      ```

   4. Invoking **setupConnOptions()** to set up all args

   5. Attempting to connect to the NATS system, using the **Address** provided and the **Options** created

   6. Setting up NATS CloseHandler

   7. Setting up Emitter

      ```go
      // Emitter ...
      type Emitter struct {
      	// Mutex to prevent race conditions within the Emitter.
      	*sync.Mutex
      	// Map of event to a slice of listener function's reflect Values.
      	events map[interface{}][]reflect.Value
      	// Optional RecoveryListener to call when a panic occurs.
      	recoverer RecoveryListener
      	// Maximum listeners for debugging potential memory leaks.
      	maxListeners int
      	// Map used to remove Listeners wrapped in a Once func
      	onces map[reflect.Value]reflect.Value
      }
      ```

   8. Setting up Mutex, requestListener, broadcastListeners

2. Broadcaster

   1. ```go
      type Broadcaster struct {
      	emission.Emitter
      	subj string
      	np   *NatsProtoo
      }
      ```

   2. Setting the Emitter as the NatsProtoo's Emitter and Setting the NatsProtoo as the one, which previously created

   3. Setting <u>EventListener</u> OnClose and OnError to the the Broadcaster's NatsProtoo.