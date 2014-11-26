---
layout: post
category : pages
tags : [Java, IOT, Tinkerforge, Testing]
disqusid : mockingtinkerforge
---

A couple of days have passed since the TinkerForge-API has been published on maven central. But what can you do with it if there is no hardware around. This was my situation
as i wanted to try it out and play with the weatherstation from jaxenter (<a href="http://jaxenter.de/Internet-of-Things-mit-Java-8-TinkerForge-Teil-5-171242">weatherstation</a>).<br/>
So what can you do now? You can either mock the hardware sensor and struggle with the protocol of the sensors or you can mock the software sensor. The latter approach seems easier to me. So lets start.<br/>

If you only want to have a single value request (e.g. getTemperature()) the task is easy. You only have to override the method on sensor instantiation.
{% highlight java linenos %}
new BrickletTemperature("dV6", new IPConnection()){
    @Override
    public short getTemperature() throws TimeoutException, NotConnectedException {
           return 42;
    }
};
{% endhighlight %}

The more interesting task is to hack the callback listeners so that you continuously get measured values. Ok, this would be no real data but it should be sufficient for
testing and playing. To hack the sensor we need to dive a little bit into the internals of the sensors. If you add a listener to a sensor it is put into an internal list which is
processed if a specific callback event (for example for temperature) is fired. In this process the changed value is passed.<br/>
So the task is to create a thread which fires a specific callback event on a sensor.

{% highlight java linenos %}
//Creates Standard-BrickletTemperature with injected IPConnection
BrickletTemperature brickletTemperature = new BrickletTemperature("kjh6", ipConnection);

//Determine the callback event constants (note: callbackreached listeners are ignored)
List<Integer> ints = new ArrayList<>();
    Field[] declaredFields = deviceClass.getDeclaredFields();
    for (Field declaredField : declaredFields) {
    String fieldName = declaredField.getName();
    if (fieldName.startsWith("CALLBACK_") && !fieldName.endsWith("REACHED")) {
    declaredField.setAccessible(true);
    int callbackIndex = declaredField.getInt(device);
    ints.add(callbackIndex);
    }
    }

    //Creates a Value generator for the bricklet
    try {
    createBrickletMock(ipConnection, brickletTemperature, (byte) 200, ints);
    } catch (NoSuchMethodException e) {e.printStackTrace();}
{% endhighlight %}
The callback events of each sensor are mapped to a constant in the sensor class which starts with CALLBACK (callback reached events have CALLBACK_<name>_REACHED) so that you can use reflection to read the values.
{% highlight java linenos %}
static <Bricklet extends Device> void startCallbackListenerThread(IPConnection ipcon, Device bricklet, byte startValue, List<Integer> callbackIndizes) throws NoSuchMethodException {
        Class<IPConnection> ipConnectionClass = IPConnection.class;
        Method callDeviceListener = ipConnectionClass.getDeclaredMethod("callDeviceListener", Device.class, byte.class, byte[].class);
        callDeviceListener.setAccessible(true);

        //start thread for each callback event
        for (int callbackIndex : callbackIndizes) {
            new Thread(() -> {

                try {

                    Random random = new Random();
                    while (true) {

                        //Generates values -1, 0 or 1
                        int randomDiff = random.nextInt(3) - 1;

                        //Invoke on device
                        callDeviceListener.invoke(ipcon, bricklet, (byte) callbackIndex, new byte[]{0, 0, 0, 0, 0, 0, 0, 0, (byte) (startValue + randomDiff), 0});


                        //wait 5s
                        Thread.sleep(THREAD_SLEEP_MILLIS);
                    }
                } catch (IllegalAccessException | InvocationTargetException | InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }{% endhighlight %}
