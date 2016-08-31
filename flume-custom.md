# Flume Custom Components


### Source

``` java
public class MySource extends AbstractSource implements Configurable, PollableSource {
  private String myProp;

  @Override
  public void configure(Context context) {
    String myProp = context.getString("myProp", "defaultValue");

    // Process the myProp value (e.g. validation, convert to another type, ...)

    // Store myProp for later retrieval by process() method
    this.myProp = myProp;
  }

  @Override
  public void start() {
    // Initialize the connection to the external client
  }

  @Override
  public void stop () {
    // Disconnect from external client and do any additional cleanup
    // (e.g. releasing resources or nulling-out field values) ..
  }

  @Override
  public Status process() throws EventDeliveryException {
    Status status = null;

    try {
      // This try clause includes whatever Channel/Event operations you want to do

      // Receive new data
      Event e = getSomeData();

      // Store the Event into this Source's associated Channel(s)
      getChannelProcessor().processEvent(e);

      status = Status.READY;
    } catch (Throwable t) {
      // Log exception, handle individual exceptions as needed

      status = Status.BACKOFF;

      // re-throw all Errors
      if (t instanceof Error) {
        throw (Error)t;
      }
    } finally {
      txn.close();
    }
    return status;
  }
}
```

## Sink

```java
public class MySink extends AbstractSink implements Configurable {
  private String myProp;

  @Override
  public void configure(Context context) {
    String myProp = context.getString("myProp", "defaultValue");

    // Process the myProp value (e.g. validation)

    // Store myProp for later retrieval by process() method
    this.myProp = myProp;
  }

  @Override
  public void start() {
    // Initialize the connection to the external repository (e.g. HDFS) that
    // this Sink will forward Events to ..
  }

  @Override
  public void stop () {
    // Disconnect from the external respository and do any
    // additional cleanup (e.g. releasing resources or nulling-out
    // field values) ..
  }

  @Override
  public Status process() throws EventDeliveryException {
    Status status = null;

    // Start transaction
    Channel ch = getChannel();
    Transaction txn = ch.getTransaction();
    txn.begin();
    try {
      // This try clause includes whatever Channel operations you want to do

      Event event = ch.take();

      // Send the Event to the external repository.
      // storeSomeData(e);

      txn.commit();
      status = Status.READY;
    } catch (Throwable t) {
      txn.rollback();

      // Log exception, handle individual exceptions as needed

      status = Status.BACKOFF;

      // re-throw all Errors
      if (t instanceof Error) {
        throw (Error)t;
      }
    }
    return status;
  }
}
```


## Redis Sink

```java
public class RedisSink extends AbstractSink implements Configurable {

  private Logger logger = LoggerFactory.getLogger(this.getClass());

  String redisHost;
  int redisPort;
  String redisKey;

  private Jedis jedis;

  @Override
  public Status process() throws EventDeliveryException {
    // TODO Auto-generated method stub
    Status status = null;
    // Start transaction
    Channel channel = getChannel();
    Transaction transaction = channel.getTransaction();
    transaction.begin();
    try {
      Event event = channel.take();
      Long result = jedis.rpush(redisKey, new String(event.getBody(), "utf-8"));
      if (result > 0) {
        logger.info("lpush : " + new String(event.getBody(), "utf-8")
            + " into " + redisKey + "(" + result + ")");
        transaction.commit();
        status = Status.READY;
      } else {
        logger.error("RPUSH FAILED");
        transaction.rollback();
        status = Status.BACKOFF;
      }
    } catch (Throwable t) {
      transaction.rollback();
      status = Status.BACKOFF;
      if (t instanceof Error) {
        throw (Error) t;
      }
    } finally {
      transaction.close();
    }
    return status;
  }

  @Override
  public void configure(Context context) {
    redisHost = context.getString("redisHost");
    redisPort = context.getInteger("redisPort");
    redisKey = context.getString("redisKey");
  }

  @Override
  public synchronized void start() {
    // TODO Auto-generated method stub
    super.start();
    jedis = new Jedis(redisHost, redisPort);
  }

  @Override
  public synchronized void stop() {
    // TODO Auto-generated method stub
    super.stop();
    jedis.disconnect();
  }

}
```