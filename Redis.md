# Redis: 

## Redis initialized by ISLB during it's own initialization. 

1. NewRedis(c Config) *Redis; Fetching the configuration from Viper to set up the Redis.

2. ```go
   type Redis struct {
   	cluster     *redis.ClusterClient
   	single      *redis.Client
   	clusterMode bool
   	mutex       *sync.Mutex
   }
   ```

3. Check how many addresses provided from Conf 

   1. Single: Initialize Single Client

      1. Setting up Redis Options

         ```go
         &redis.Options{
         				Addr:         c.Addrs[0], // use default Addr
         				Password:     c.Pwd,      // no password set
         				DB:           c.DB,       // use default DB
         				DialTimeout:  3 * time.Second,
         				ReadTimeout:  5 * time.Second,
         				WriteTimeout: 5 * time.Second,
         			})
         ```

      2. Ping the Redis server, handling Errors.

      3. Set Redis Configs by running Do Ops

      4. Turn off clusterMode

      5. Setting up Mutex

   2. ClusterClient: **ClusterClient is a Redis Cluster client representing a pool of zero or more underlying connections. It's safe for concurrent use by multiple goroutines.**

      1. ```go
         &redis.ClusterOptions{
         			Addrs:        c.Addrs,
         			Password:     c.Pwd,
         			DialTimeout:  3 * time.Second,
         			ReadTimeout:  5 * time.Second,
         			WriteTimeout: 5 * time.Second,
         		})
         ```

      2. Set Configuration 

      3. Turn on clusterMode

      4. No Mutex set.

