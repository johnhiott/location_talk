# Location

## by: John Hiott

### @johnhiott

^ Hey!  I am John Hiott.  I am a developer at Passport and have been there for around 6 months.  At Passport, we provide mobile solutions for parking and transit.  I work on our ticketing and enforcement application.


---
# Why use location?

- Tracking
- Proximity Alerts
- Locations Aware Content
- Navigation
- And so on....

^ Location is one of the most useful features on a mobile device. I cannot think of many apps that I use frequently that do not used locations.  My many weather apps, run tracker, maps for my commute, twitter, Facebook, Uber.  In one app at Passport, we use location to analyze the routes of parking enforcement officers and the other we use location to find nearby parking.  The list of uses is never ending.

---
# What is location?

6 things make up location

- Lat
- Lng
- Altitude
- Time
- Accuracy
- Provider

^ Lat, Lng, and Altitude is what people typically think of when they think location.  This is not wrong but it is only part of it.  Lat, Lng and Altitude represent the physical location.  However, when writing apps we have to think of location having more than just physical attributes.  A location has a provider (GPS, WiFi, Cell Signal), a time, and an accuracy.

---
# Real Quick

- Provider
    - Network
    - GPS
    - Passive
- Accuracy
    - radius of 68% confidence.

^ More than likely, most of you already know what the location provider is but just in case, the location provider is where the location comes from.  Android has 3-4 location providers, depending on how you want to count them

^ Accuracy is defined as the radius of 68% confidence. In other words, if you draw a circle centered at this location's latitude and longitude, and with a radius equal to the accuracy (in meters), then there is a 68% probability that the true location is inside the circle.

---
# Things to consider

- Accuracy, Time
- Battery Life

^ In a perfect world we would always have the user's most up to date and accurate location.  As we all know, this is not the case. The biggest thing in the way of our perfect little world in the phone's battery.  Typically, the more accurate the location the higher the battery use is.  As developers, we have to find some kind of compromise.  When making these decisions we weigh how important accuracy is to our app and how much battery loss is acceptable for what we are trying to accomplish.

^ If your app has an accurate and current location it is considered fresh, at least in terms of location.  App freshness also covers data but we are talking about location.  In our perfect world, your app should always be fresh.

---
# Android's Location APIs

---
# Getting Location Updates

```xml
<!-- use one or the other -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

```

```java
LocationManager locationManager = (LocationManager) this.getSystemService(Context.LOCATION_SERVICE);

LocationListener locationListener = new LocationListener() {
    public void onLocationChanged(Location location) {
      makeUseOfNewLocation(location);
    }

    ...
  };

locationManager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 0, 0, locationListener);

```

```java
locationManager.removeUpdates(locationListener);
```

^ You might want to start listening for location updates as soon as your application starts, or only after users activate a certain feature. Be aware that long windows of listening for location fixes can consume a lot of battery power, but short periods might not allow for sufficient accuracy.

^ minTime -	minimum time interval between location updates, in milliseconds
^ minDistance - minimum distance between location updates, in meters

---
# Last Known Location

```java
Location lastKnownLocation = locationManager.getLastKnownLocation(locationProvider);

if (isLocationUsable(lastKnownLocation)) {
    doSomethingWithLocation();

}else {
    requestLocationUpdates();
}
```

---
# Using Passive Location

```xml
<!-- Manifest -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
```

```java
//Using Passive Location Provider
public void requestUpdates() {
    locationManager.requestLocationUpdates(LocationManager.PASSIVE_PROVIDER,
                                            minTime, minDistance, locationListener);
}
```

^ You request passive updates the same way you would with any other provider.  You pass requestLocationUpdates the minimum time, minimum distance, the listener/internt, and of course the provider you want to use which of course is the passive provider.

^ When using the passive provider, you first must request permission to receive fine/accurate updates.  The updates may not of needed this permission but because you do not know the provider, you must have this permission.

---
# Passive Location: a step further
## Using Intents

```java
final String locationAction = "com.johnhiott.LOCATION_UPDATE_RECEIVED";
int flags = PendingIntent.FLAG_UPDATE_CURRENT;

Intent intent = new Intent(locationAction);
PendingIntent pendingIntent = PendingIntent.getBroadcast(this, resultCode, intent, flags);

locationManager.requestLocationUpdates(provider, minTime, minDistance, pendingIntent);
```

^ All locations providers allow you to use intents instead of listeners.  This comes in handy when multiple activities or services need location updates.  However, intents are even more useful when using the passive provider.  I will cover why in a slide or two.

---
# Passive Location
## Using Intents

```java
BroadcastReceiver locationReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        Location location (Location)intent.getExtras().get(LocationManager.KEY_LOCATION_CHANGED);
        //Do what ya will
    }
};

IntentFilter locationIntentFilter = new IntentFilter(locationAction);
registerReceiver(locationReceiver, locationIntentFilter);
```

---
# Passive Location: a step further

```xml
<receiver android:name=".locationReceiver" android:enabled="true">
    <intent-filter>
        <action android:name="com.johnhiott.LOCATION_UPDATE_RECEIVED"/>
    </intent-filter>
</receiver>
```

^ If we take the passive provider one step further, we can get location updates without your app even being open.  Let's say you just used Maps to navigate to a new restaurant, with each location update Maps receives, your app gets one too.  So now a user opens your app to check in or get some details about the new place, the last location received is probably close enough to work for your app.

---
# Google Location Services

^ The Android API's seem easy enough.  However, there is a lot to account for.  What if someone turns GPS on or off??  Are you using the best provider that is enabled that meets your criteria????  Using Android's APIs, you have to account for all of this and switch providers when needed.  This can become a headache.  However, Google Play Services provides us with a set of API's designed to help us out with this.

---
# Setup Play Services

Add Play Services to your build.gradle file.

```java
dependencies {
    compile 'com.google.android.gms:play-services:7.0.0'
}
```

---
# Fused Location Provider

- Simple APIs
- Immediately available
- Power-efficiency
- Versatility

^ The Fused Location Provider intelligently manages the underlying location technology and gives you the best location according to your needs.

^ Simple APIS -Lets you specify high-level needs like "high accuracy" or "low power", instead of having to worry about location providers.
^ Immediately available - Gives your apps immediate access to the best, most recent location.
^ Power-efficiency - Minimizes your app's use of power. Based on all incoming location requests and available sensors, fused location provider chooses the most efficient way to meet those needs.
^ Versatility - Meets a wide range of needs, from foreground uses that need highly accurate location to background uses that need periodic location updates with negligible power impact.


---
# Get Last Known Location

```xml
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```

```java
protected synchronized void buildGoogleApiClient() {
    mGoogleApiClient = new GoogleApiClient.Builder(this)
        .addConnectionCallbacks(this)
        .addOnConnectionFailedListener(this)
        .addApi(LocationServices.API)
        .build();
}

public class MainActivity extends ActionBarActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
    @Override
    public void onConnected(Bundle connectionHint) {
        mLastLocation = LocationServices.FusedLocationApi.getLastLocation(mGoogleApiClient);
        if (mLastLocation != null) {
            //use location
        }
    }
}
```

^ the last known location is usually the user's current location... but if for some reason it is not or your app still needs to request location updates, the fused location provider can do that too!
^ for coarse location the API usually returns a location with an accuracy equivalent to about a city block

---
# Request Location Updates

```java
LocationRequest mLocationRequest = new LocationRequest();
mLocationRequest.setInterval(10000);
mLocationRequest.setFastestInterval(5000);
mLocationRequest.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);

LocationServices.FusedLocationApi.requestLocationUpdates(
         mGoogleApiClient, mLocationRequest, this);

@Override
public void onLocationChanged(Location location) {
    //do something awesome
}

```
