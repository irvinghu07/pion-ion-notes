# PION/ION

### ISLB: **（docker-compose defined that ISLB depends on ETCD, NATS AND Redis）**

1. 使用 net/http **ListenAndServe**(port, nil=>Handler)

   - **DefaultServeMux** would be used if the Handler is nil

2. 创建 discovery.**serviceNode**(Etcd Addr,  dc)

   - 创建ServiceRegistry， 并设置新的etcd到该ServiceRegistry
     	etcd, _ = newEtcd(endpoints)
       	registry.etcd = etcd

3. 将islb注册自身 **serviceNode**.RegisterNode

   - RegisterServiceNode
     - 分配接口IP：node.Info["ip"] = GetInterfaceIP
     - **Etcd** keep [etcd:存储大规模分布式系统的配置信息](https://blog.csdn.net/weixin_40161254/article/details/88659185?utm_source=app)
       - Grant **lease**
       - Put K/V：将serviceNode的路径Path和serviceNode的信息Info存储到etcd中。
       - KeepAlive： KeepAlive keeps the given **lease** alive forever. 

4. Redis Config

5. get**RPC**Chennel 

   - **Remote Procedure Call** (**RPC**) is a protocol that one program can use to request a service from a program located in another computer on a network without having to understand the network's details. **RPC** is used to call other processes on the remote systems like a local system.

6. getEventChannel

   - An **EventChannel** is a broadcast queue of **events**. **Events** may be any type that implements Send + Sync + 'static . Typically, **EventChannel** s are inserted as resources in the World .

7. **ISLB init:** (The Initializtion of ISLB depends on Redis, Nats and Etcd); ./etcd.md)

8. 

   - ```go
     func Init(dcID, nodeID, rpcID, eventID string, redisCfg db.Config, etcd []string, natsURL string) {
     	dc = dcID
     	nid = nodeID
     	redis = db.NewRedis(redisCfg)
     	protoo = nprotoo.NewNatsProtoo(natsURL)
     	broadcaster = protoo.NewBroadcaster(eventID)
     	services = make(map[string]discovery.Node)
     	handleRequest(rpcID)
     	WatchAllStreams()		
     ```

   - Checking the Initialization source code for:   [NATS.md](./Nats.md), [Redis.md](./Redis.md) and [etcd.md](./etcd.md)

   - 使用一个map：services记录所有的服务

     ```go
     services = make(map[string]discovery.Node)
     ```

   - ISLB: handleRequest

     ```go
     func handleRequest(rpcID string) {
     	log.Infof("handleRequest: rpcID => [%v]", rpcID)
     
     	protoo.OnRequest(rpcID, func(request nprotoo.Request, accept nprotoo.RespondFunc, reject nprotoo.RejectFunc) {
     		go func(request nprotoo.Request, accept nprotoo.RespondFunc, reject nprotoo.RejectFunc) {
     			method := request.Method
     			msg := request.Data
     			log.Infof("method => %s", method)
     
     			var result interface{}
     			err := util.NewNpError(400, fmt.Sprintf("Unkown method [%s]", method))
     
     			switch method {
     			case proto.IslbFindService:
     				var msgData proto.FindServiceParams
     				if err = msg.Unmarshal(&msgData); err == nil {
     					result, err = findServiceNode(msgData)
     				}
     			case proto.IslbOnStreamAdd:
     				var msgData proto.StreamAddMsg
     				if err = msg.Unmarshal(&msgData); err == nil {
     					result, err = streamAdd(msgData)
     				}
     			case proto.IslbOnStreamRemove:
     				var msgData proto.StreamRemoveMsg
     				if err = msg.Unmarshal(&msgData); err == nil {
     					result, err = streamRemove(msgData)
     				}
     			case proto.IslbGetPubs:
     				var msgData proto.RoomInfo
     				if err = msg.Unmarshal(&msgData); err == nil {
     					result, err = getPubs(msgData)
     				}
     			case proto.IslbClientOnJoin:
     				var msgData proto.JoinMsg
     				if err = msg.Unmarshal(&msgData); err == nil {
     					result, err = clientJoin(msgData)
     				}
     			case proto.IslbClientOnLeave:
     				var msgData proto.RoomInfo
     				if err = msg.Unmarshal(&msgData); err == nil {
     					result, err = clientLeave(msgData)
     				}
     			case proto.IslbGetMediaInfo:
     				var msgData proto.MediaInfo
     				if err = msg.Unmarshal(&msgData); err == nil {
     					result, err = getMediaInfo(msgData)
     				}
     			// case proto.IslbRelay:
     			// 	result, err = relay(data)
     			// case proto.IslbUnrelay:
     			// 	result, err = unRelay(data)
     			case proto.IslbOnBroadcast:
     				var msgData proto.BroadcastMsg
     				if err = msg.Unmarshal(&msgData); err == nil {
     					result, err = broadcast(msgData)
     				}
     			}
     
     			if err != nil {
     				reject(err.Code, err.Reason)
     			} else {
     				accept(result)
     			}
     		}(request, accept, reject)
     	})
     }
     ```

     - TODO

     - IslbOnStreamAdd:

       - Parse JSON from msg and deserialize it to msgData

       - Build Redis user hash key using dc + roomID + userID

       - Build Redis media hash key using MediaInfo + dc

       - 

       - Set K/V in Redis

         - ```go
           err = redis.HSetTTL(mkey, field, value, redisLongKeyTTL)
           err = redis.HSetTTL(mkey, descField, data.Description, redisLongKeyTTL)
           ```

       - Get all fields from Redis

         ```go
         fields := redis.HGetAll(ukey)
         ```

       - broadcaster.Say()

       - watchStream(mediaKey)

     - watchAllStreams: Fetch all streams on mediaKey and for each mediaKey,  **Redis OP watch** their streams respectively.

       - watchStream: Emit an Go routine to Watch for "del" or "expire" Redis OP and remove Stream.

9. Initialize **discovery.ServiceWatcher()**

10. WatchServiceNode:

   - Get all nodes from ServiceWatcher.ServiceRegistry
   - Get from serviceWatcher.NodesMap
   - serviceWatcher.GetNodesByID()
     - Setting all nodes state UP
     - **etcd watch** the node
     - Setting the node into the NodesMap