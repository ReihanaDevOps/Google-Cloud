PS C:\Users\Reihana\Documents\Reihana\Bloom\bloomworld\shop-service> docker logs bloomworld-shop-service
◇ injected env (0) from .env // tip: ⌁ auth for agents [www.vestauth.com]
Shop Service running on port 3000
Database connection failed: Error: connect ECONNREFUSED 127.0.0.1:5433
    at TCPConnectWrap.afterConnect [as oncomplete] (node:net:1638:16) {
  errno: -111,
  code: 'ECONNREFUSED',
  syscall: 'connect',
  address: '127.0.0.1',
  port: 5433
}
PS C:\Users\Reihana\Documents\Reihana\Bloom\bloomworld\shop-service> docker network create bloomworld-network
a847eace6d1474b94474caea677075935f3af79ceca684dc966d5ff012c0d901
PS C:\Users\Reihana\Documents\Reihana\Bloom\bloomworld\shop-service> docker network connect bloomworld-network bloomworld-postgres
PS C:\Users\Reihana\Documents\Reihana\Bloom\bloomworld\shop-service> docker stop bloomworld-shop-service
bloomworld-shop-service
PS C:\Users\Reihana\Documents\Reihana\Bloom\bloomworld\shop-service> docker rm bloomworld-shop-service
bloomworld-shop-service
PS C:\Users\Reihana\Documents\Reihana\Bloom\bloomworld\shop-service> docker run --name bloomworld-shop-service `
>>   --network bloomworld-network `
>>   --env-file .env `
>>   -p 3000:3000 `
>>   -d bloomworld-shop-service
4fe822289cc172ff6f726fdbea21ead11eab46fca91301f81a38a15190fae5b5
PS C:\Users\Reihana\Documents\Reihana\Bloom\bloomworld\shop-service> docker logs bloomworld-shop-service
◇ injected env (0) from .env // tip: ◈ secrets for agents [www.dotenvx.com]
Shop Service running on port 3000
Database connected: { now: 2026-09-07T06:51:34.647Z }
PS C:\Users\Reihana\Documents\Reihana\Bloom\bloomworld\shop-service> 
