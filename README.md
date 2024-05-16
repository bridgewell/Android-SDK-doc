# Documentation Guide

Bridgewell SDK for Android.  <br><br>

<b>Version: 1.0</b> 

## Revision History
| Date      | Version | Description     | Author |
| :---        |    :----   |     :--- |       :--- |
| 20/02/2024      | 1.0        |  Create new  | Hoang Chung |



<u><b>Note</b></u>: All integration examples are written in Kotlin. If you’re using Java please convert them to corresponding Java code

<br>

# Requirements

1. Android 7.0 (SDK version 24) or newer
2. Mobile phone must have Google play service 
3. Android Studio version 2023.3.1 or newer
4. Gradle version 8.2 or newer

# How to build project?

1. Clone this repository to your computer
2. Make sure JAVA_HOME is added to your environment
3. Open command line and run this command
   ```
   gradlew clean :bwmobile:assembleRelease
   ```
# General setup
### 1. SDK Integration

#### a. Maven
```
dependencies {
    ////
    
    // Bw Mobile SDK 
    implementation ''
}
```
#### b. File .aar
Or you can import the .aar file to your project then dependency it into the app. 

To download precompile file, You can go to Actions tab -> Choose commit you want to download -> Go to Artifacs -> Download the file

How to import .aar file from github actions to Android Studio project?

1. Copy file .aar file to libs folder. Example: /Path/to/your/project/app/libs/bwmobile.aar
2. In build.gradle file of your application add this code to dependencies
   ```
   implementation(files("libs/bwmobile.aar"))
   ```
3. Sync project with Gradle file 

### 2. Setup SDK
#### a. Set BW Server
Once you have a Bw server, you will add them to BW mobile. For example, if you’re using the Rubicon Server.
```
BWMobile.getInstance().setAccountId("YOUR_ACCOUNT_ID")
BWMobile.getInstance().setHostServer(HostServer.RUBICON)
```
Or you can pass other values like ```Host.APPNEXUS``` and If you have opted to host your own Bw Server solution you will need to store the url to the server in your app. Make sure that your URL points to the /openrtb2/auction endpoint.

```
BWMobile.getInstance().setHostServer(HostServer.Custom("YOUR_CUSTOM_HOST_SERVER"))
```

#### b. Initialize SDK
After you have an account id and host server. You should initialize BW Sdk like this
```
BWMobile.getInstance().initialize(
   context,
   object : OnInitializationListener {


       override fun onSuccess(hasWarningPBS: Boolean) {   
       }


       override fun onFailed(msg: String) {
       }
   }
)

```

During the initialization, SDK creates internal classes and performs the health check request to the /status endpoint. If you use a custom host you should provide a custom status endpoint as well:

```
BWMobile.getInstance().setEndPoint("YOUR_END_POINT")
```

quest to the /status endpoint. If you use a custom host you should provide a custom status endpoint 

## 3. Update your Android manifest
### a. Declare some permissions
Before you start, you need to integrate the SDK by updating your Android manifest.

```
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
<uses-permission android:name="com.google.android.gms.permission.AD_ID"/>

```
``` ACCESS_COARSE_LOCATION ``` and ``` ACCESS_FINE_LOCATION ``` will automatically allow the device to send user location for targeting, which can help increase revenue by increasing the value of impressions to buyers.

## 4. Account ID
String containing the BW Server account ID.
```
BWMobile.getInstance().setAccountId("YOUR_ACCOUNT_ID")
val accountId = BWMobile.getInstance().getAccountId()
```

## 5. Host
Object containing configuration for your Bw Server host with which the Bw SDK will communicate. Choose from the system-defined Bw Server hosts or define your own custom Bw Server host
```
BWMobile.getInstance().setHostServer(host)
```

`host` should be `HostServer.RUBICON`, `HostServer.RUBICON` or `HostServer.Custom("YOUR_CUSTOM_HOST_SERVER")`

## 6. Timeout
The Bw timeout set in milliseconds, will return control to the ad server SDK to fetch an ad once the expiration period is achieved. Because Bw SDK solicits bids from Bw Server in one payload, setting Bw timeout too low can stymie all demand resulting in a potential negative revenue impact.
```
BWMobile.getInstance().setTimeoutMilliseconds(30_000)
val timeout = BWMobile.getInstance().getTimeoutMilliseconds()
```

## 7. Creative Factory Timeout
Indicates how long each creative has to load before it is considered a failure.
```
BWMobile.getInstance().setCreativeFactoryTimeout(8_000)
val creativeTimeout = BWMobile.getInstance().getCreativeFactoryTimeout()
```
The creativeFactoryTimeoutPreRenderContent controls the timeout for video and interstitial ads. The creativeFactoryTimeout is used for HTML banner ads.

## 8. ShareGeoLocation
If this flag is True AND the app collects the user’s geographical location data, BW Mobile will send the user’s geographical location data to BW Server. If this flag is False OR the app does not collect the user’s geographical location data, BW Mobile will not populate any user geographical location information in the call to BW Server.

```
BWMobile.getInstance().setShareGeoLocation(true)
val isShareGeoLocation = BWMobile.getInstance().getShareGeoLocation()
```
<br><br>

# Ad setup
## 1. Custom Bidding Integration
You can use BW SDK to monetize your app with a custom ad server or even without it. Use the `InAppApi` to obtain the targeting keywords for following usage with the custom ad server and display the winning bid without the primary ad server and its SDK.

### a. Display banner
Sample
```
// Create an instance of InAppApi
private var adApi: InAppApi = InAppApi()


// perform to fetch and load ad
adApi.createDisplayBannerAd(
   context = context,
   model = DisplayBannerModel(
       configId = "prebid-demo-mraid-expand-1-part",
       width = 300,
       height = 250,
       refreshTimeSeconds = 35_000
   ),
   viewContainer = viewContainer,
   listener = object: BannerAdListener {
       override fun onAdStartLoad(bannerView: BannerView?) {}


       override fun onAdLoaded(bannerView: BannerView?) {}


       override fun onAdDisplayed(bannerView: BannerView?) {}


       override fun onAdFailed(bannerView: BannerView?, exception: AdException?) {}


       override fun onAdClicked(bannerView: BannerView?) {}


       override fun onAdClosed(bannerView: BannerView?) {}


   }
)
```
<br>

You also need pass some necessary parameters to createDisplayBannerAd() function
| Parameter      | Explain | Detail     |
| :---        |    :----   |          :--- |
| context      | Instance of Context        |  -  |
| model   | Include necessary informations to generate the ad | `ConfigId`: an ID of a Stored Impression on the Bw server  <br>  `refreshTimeSeconds`: refresh time for each fetchDemand call in milisecond <br> `width`: width of the ad <br>  `Height`: height of the ad  |
| viewContainer | which view will be add the ad into | - |
| listener | register a callback as interface BannerAdListener to listener | `onAdStartLoad` <br> `onAdLoaded` <br> `onAdDisplayed` <br> `onAdFailed` <br> `onAdClicked` <br> `onAdClosed` |

<br>

### b. Display video
Sample:
```
// Create an instance of InAppApi
private var inAppApi: InAppApi = InAppApi()

inAppApi.createDisplayBannerAd(
    this,
    model = DisplayVideoModel(
        configId = CONFIG_ID,
        width = WIDTH,
        height = HEIGHT,
        refreshTimeSeconds = refreshTimeSeconds
    ),
    viewContainer = adWrapperView,
    listener = object: BannerAdListener {
        override fun onAdStartLoad(bannerView: BannerView?) {}

        override fun onAdLoaded(bannerView: BannerView?) {}

        override fun onAdDisplayed(bannerView: BannerView?) {}

        override fun onAdFailed(bannerView: BannerView?, exception: AdException?) {}

        override fun onAdClicked(bannerView: BannerView?) {}

        override fun onAdClosed(bannerView: BannerView?) {}

    }
)


```
| Parameter      | Explain | Detail     |
| :---           | :----   | :---       |
| context        | Instance of Context | - |
| model        | object DisplayVideoModel, include necessary informations to generate the ad | `ConfigId`: an ID of a Stored Impression on the Bw server  <br>  `refreshTimeSeconds`: refresh time for each fetchDemand call in milisecond <br> `width`: width of the ad <br>  `Height`: height of the ad   |
| viewContainer        | which view will be add the ad into | - |
| listener        | register a callback as interface BannerAdListener to listener | `onAdStartLoad` <br> `onAdLoaded` <br> `onAdDisplayed` <br> `onAdFailed` <br> `onAdClicked` <br> `onAdClosed` |


### c. Interstitial banner
Sample:
```
// Create an instance of InAppApi
private var adApi: InAppApi = InAppApi()


// perform to fetch and load ad
adApi.createInterstitialBannerAd(
   this,
   model = InterstitialBannerModel(
       configId = "prebid-demo-display-interstitial-320-480",
   ),
   listener = object: InterstitialAdListener {
       override fun onAdLoaded(interstitialAdUnit: InterstitialAdUnit?) {}


       override fun onAdDisplayed(interstitialAdUnit: InterstitialAdUnit?) {}


       override fun onAdFailed(interstitialAdUnit: InterstitialAdUnit?, e: AdException?) {}


       override fun onAdClicked(interstitialAdUnit: InterstitialAdUnit?) {}


       override fun onAdClosed(interstitialAdUnit: InterstitialAdUnit?) {}
   }
)
```

| Parameter      | Explain | Detail     |
| :---        |    :----   |         :--- |
| context      | Instance of Context        |  -  |
| model   | Include necessary informations to generate the ad | `ConfigId`: an ID of a Stored Impression on the Bw server  <br>  `refreshTimeSeconds`: refresh time for each fetchDemand call in milisecond <br> `width`: width of the ad <br>  `Height`: height of the ad  |
| viewContainer | which view will be add the ad into | - |
| listener | register a callback as interface BannerAdListener to listener | - |


### d. Interstital video
Sample:
```
// Create an instance of InAppApi
private var adApi: InAppApi = InAppApi()

inAppApi.createInterstitialVideoAd(
    context = this,
    model = InterstitialBannerModel(
        configId = CONFIG_ID
    ),
    listener = object: InterstitialAdListener {
        override fun onAdLoaded(interstitialAdUnit: InterstitialAdUnit?) {}

        override fun onAdDisplayed(interstitialAdUnit: InterstitialAdUnit?) {}

        override fun onAdFailed(interstitialAdUnit: InterstitialAdUnit?, e: AdException?) {}

        override fun onAdClicked(interstitialAdUnit: InterstitialAdUnit?) {}

        override fun onAdClosed(interstitialAdUnit: InterstitialAdUnit?) {}
    }
)
```

| Parameter      | Explain | Detail     |
| :---        |    :----   |         :--- |
| context      | Instance of Context        |  -  |
| model   | object InterstitialBannerModel, include necessary informations to generate the ad | `ConfigId`: an ID of a Stored Impression on the Bw server  <br>  `minWidthPerc`: [optional] width of the ad <br>  `minHeightPerc`: [optional] height of the ad  |
| listener | register a callback as interface InterstitialAdListener to listener | - |

### e. Native
#### Sample:

```
// Create an instance of InAppApi
private var adApi: InAppApi = InAppApi()

// Create native ad
inAppApi.createNativeAd(
    model =
        InAppNativeModel(
            configId = CONFIG_ID,
            configModel = getConfigNativeModel(),
        ),
    onInflateNativeAd = {
        inflateView(it)
    },
)
```
<br>

#### Params detail

| Parameter      | Explain | Detail     |
| :---        |    :----   |         :--- |
| model      | object `InAppNativeModel` include ad information       |  `configId`: an ID of a Stored Impression on the Bw server<br> `configModel`: An object declares config information about the ad   |
| onInflateNativeAd | callback function to use inflate view when the ad is already available | - |
<br>

You also can find the sample to declare config model in the following code:
```
private fun getConfigNativeModel(): InAppNativeConfigModel {
    val config =
        InAppNativeConfigModel(
            contextType = NativeAdUnit.CONTEXT_TYPE.SOCIAL_CENTRIC,
            placementType = NativeAdUnit.PLACEMENTTYPE.CONTENT_FEED,
            contextSubType = NativeAdUnit.CONTEXTSUBTYPE.GENERAL_SOCIAL,
            eventTracker = NativeEventTracker.EVENT_TYPE.IMPRESSION,
            trackerMethods =
                arrayListOf(
                    NativeEventTracker.EVENT_TRACKING_METHOD.IMAGE,
                    NativeEventTracker.EVENT_TRACKING_METHOD.JS,
                ),
            titleAsset =
                NativeTitleAsset().apply {
                    setLength(90)
                    isRequired = true
                },
            iconAsset =
                NativeImageAsset(20, 20, 20, 20).apply {
                    imageType = NativeImageAsset.IMAGE_TYPE.ICON
                    isRequired = true
                },
            imageAsset =
                NativeImageAsset(200, 200, 200, 200).apply {
                    imageType = NativeImageAsset.IMAGE_TYPE.MAIN
                    isRequired = true
                },
            dataAsset =
                NativeDataAsset().apply {
                    len = 90
                    dataType = NativeDataAsset.DATA_TYPE.SPONSORED
                    isRequired = true
                },
            bodyAsset =
                NativeDataAsset().apply {
                    dataType = NativeDataAsset.DATA_TYPE.DESC
                    isRequired = true
                },
            ctaAsset =
                NativeDataAsset().apply {
                    dataType = NativeDataAsset.DATA_TYPE.CTATEXT
                    isRequired = true
                },
        )
    return config
}
```
<br>
Then the ad already available, let to inflate into your view

```
private fun inflateView(ad: PrebidNativeAd) {
    val nativeContainer = LinearLayout(this)
    nativeContainer.orientation = LinearLayout.VERTICAL
    val iconAndTitle = LinearLayout(this)
    iconAndTitle.orientation = LinearLayout.HORIZONTAL

    val icon = ImageView(this)
    icon.layoutParams = LinearLayout.LayoutParams(160, 160)
    ImageUtils.download(ad.iconUrl, icon)
    iconAndTitle.addView(icon)

    val title = TextView(this)
    title.textSize = 20f
    title.text = ad.title
    iconAndTitle.addView(title)
    nativeContainer.addView(iconAndTitle)

    val image = ImageView(this)
    image.layoutParams =
        LinearLayout.LayoutParams(
            ViewGroup.LayoutParams.MATCH_PARENT,
            ViewGroup.LayoutParams.WRAP_CONTENT,
        )
    ImageUtils.download(ad.imageUrl, image)
    nativeContainer.addView(image)

    val description = TextView(this)
    description.textSize = 18f
    description.text = ad.description
    nativeContainer.addView(description)

    val cta = Button(this)
    cta.text = ad.callToAction
    nativeContainer.addView(cta)

    adWrapperView.addView(nativeContainer)

    ad.registerView(
        adWrapperView,
        listOf(icon, title, image, description, cta),
        object : PrebidNativeAdEventListener {
            override fun onAdClicked() {}

            override fun onAdImpression() {}

            override fun onAdExpired() {}
        },
    )
}
```

<u>Note</u>: For more detail of the configuration of native impls, please check this documentation: https://www.iab.com/wp-content/uploads/2018/03/OpenRTB-Native-Ads-Specification-Final-1.2.pdf 

## 2. Updating ...

<br><br>

# About

Copyright 2019 Bridgewell | All Rights Reserved
