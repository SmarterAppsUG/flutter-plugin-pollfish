# flutter_pollfish

A plugin for [Flutter](https://flutter.io) that supports rendering surveys using [Pollish SDKs](https://www.pollfish.com/docs/).

*Note*: Pollfish iOS SDK utilizes Apple's Advertising ID (IDFA) to identify and retarget users with Pollfish surveys. As of iOS 14 you should initialize Pollfish Flutter plugin in iOS only if the relevant IDFA permission was granted by the user

## Initializing the plugin

The Pollfish plugin must be initialized with a Pollfish API Key. You can retrieve an API key from Pollfish Dashboard when you [sign up](https://www.pollfish.com/signup/publisher) and create a new app.

```dart
FlutterPollfish.instance.init(apiKey: 'YOUR_API_KEY')
```

### Other Init params (optional)

During initialization you can pass different optional params:

1. **pollfishPosition**: int - TOP_LEFT=0 , BOTTOM_LEFT=1, TOP_RIGHT=2, BOTTOM_RIGHT=3, MIDDLE_LEFT=4, MIDDLE_RIGHT=5 (defines the side of the Pollish panel, and position of Pollish indicator)
2. **indPadding**: int - Sets padding (in dp) from the top or bottom according to Position of the indicator
3. **releaseMode**: bool - Sets Pollfish SDK to Debug or Release mode. Use Developer mode to test your implementation with demo surveys
4. **rewardMode**: bool - Initializes Pollfish in reward mode (used when implementing a Rewarded approach)
5. **requestUUID**: String - Sets a unique id to identify a user. This param will be passed backthrough server-to-server callbacks
5. **offerwallMode**: bool - Sets Pollfish to Offerwall mode

#### Debug Vs Release Mode

You can use Pollfish either in Debug or in Release mode. 
  
* **Debug mode** is used to show to the developer how Pollfish will be shown through an app (useful during development and testing).
* **Release mode** is the mode to be used for a released app (start receiving paid surveys).


#### init Vs custom init

*	**rewardMode = false** is the standard way of using Pollfish in your apps. Using init function with reward mode disabled, enables controlling the behavior of Pollfish in an app from Pollfish panel.

*	**rewardMode = true** ignores Pollfish behavior from Pollfish panel. It always skips showing Pollfish indicator (small Pollfish icon) and always force open Pollfish view to app users. This method is usually used when app developers want to incentivize first somehow their users, before completing surveys to increase completion rates.

For example:

```
FlutterPollfish.instance.init(
     apiKey: 'YOUR_API_KEY',
     pollfishPosition: 5,
     indPadding: 40,
     rewardMode: false,
     releaseMode: true,
     requestUUID: 'USER_INTERNAL_ID',
     offerwallMode: false);
```

## Manually showing or hiding Pollfish panel

During the lifetime of a survey, publisher can manually show or hide survey panel by calling the following functions.

```dart
FlutterPollfish.instance.show();
```

or

```dart
FlutterPollfish.instance.hide();
```

## Listening to Pollish notifications

Publishers can register and receive notifications on different events during the lifetime of a survey

### Pollfish Survey Received notification

Once a survey is received, Pollish Survey Received notification is fired to inform the publisher. The notification includes several info around the survey such as:

- CPA: money to be earned in USD cents
- LOI: length of the survey is minutes
- IR: incidence rate (conversion)
- Survey Class: survey provider (Pollish, Toluna, Cint etc)
- Reward Name: Reward name as specified on Pollfish Dashboard
- Reward Value: Reward value based on exchange rate as specified on Pollfish Dashboard

```dart
FlutterPollfish.instance.setPollfishSurveyReceivedListener(onPollfishSurveyReveived);

void onPollfishSurveyReveived(String result) => setState(() {

    List<String> surveyCharacteristics = result.split(',');

     if (surveyCharacteristics.length >= 6) {
       String _logText =
          'Survey Received: - SurveyInfo with CPA: ${surveyCharacteristics[0]} and IR: ${surveyCharacteristics[1]} and LOI: ${surveyCharacteristics[2]} and SurveyClass: ${surveyCharacteristics[3]} and RewardName: ${surveyCharacteristics[4]}  and RewardValue: ${surveyCharacteristics[5]}';
       
    }
 });

```

### Pollfish Survey Completed notification

Once a survey is completed, Pollish Survey Completed notification is fired to inform the publisher. The notification includes several info around the survey such as:

- CPA: money to be earned in USD cents
- LOI: length of the survey is minutes
- IR: incidence rate (conversion)
- Survey Class: survey provider (Pollish, Toluna, Cint etc)


```dart
FlutterPollfish.instance.setPollfishSurveyCompletedListener(onPollfishSurveyCompleted);

void onPollfishSurveyCompleted(String result) => setState(() {

    List<String> surveyCharacteristics = result.split(',');

     if (surveyCharacteristics.length >= 6) {
       String  _logText =
          'Survey Completed: - SurveyInfo with CPA: ${surveyCharacteristics[0]} and IR: ${surveyCharacteristics[1]} and LOI: ${surveyCharacteristics[2]} and SurveyClass: ${surveyCharacteristics[3]} and RewardName: ${surveyCharacteristics[4]}  and RewardValue: ${surveyCharacteristics[5]}';
       
    }
 });

```

### Pollfish Survey Panel Opened notification

A notification that informs that Pollfish Survey panel opened


```dart
FlutterPollfish.instance.setPollfishOpenedListener(onPollfishOpened);

void onPollfishOpened() => setState(() {

   String _logText = 'Survey Panel Open';
}

```

### Pollfish Survey Panel Closed notification

A notification that informs that Pollfish Survey panel closed


```dart
FlutterPollfish.instance.setPollfishClosedListener(onPollfishClosed);

void onPollfishClosed() => setState(() {

   String _logText = 'Survey Panel Closed';
}

```

### Pollfish Survey Not Available notification

A notification that informs that no survey is available for that user


```dart
FlutterPollfish.instance.setPollfishSurveyNotAvailableListener(onPollfishSurveyNotAvailable);

void onPollfishSurveyNotAvailable() => setState(() {

   String _logText = 'Survey Not Available';
}

```

### Pollfish User Reject Survey notification

A notification that informs that the user rejected the survey


```dart
FlutterPollfish.instance.setPollfishUserRejectedSurveyListener(onPollfishUserRejectedSurvey);

void onPollfishUserRejectedSurvey() => setState(() {

   String _logText = 'User Rejected Survey';
}

```

### Pollfish User Not Eligible notification

A notification that informs that the user got screened out from the survey


```dart
FlutterPollfish.instance.setPollfishUserNotEligibleListener(onPollfishUserNotEligible);

void onPollfishUserNotEligible() => setState(() {

   String _logText = 'User Not Eligible';
}

```


## Following the rewarded and/or the Offerwall approach

An example is provided on [Github](https://github.com/pollfish/flutter-plugin-pollfish) that demonstrates how a publisher can implement the rewarded and/or the Offerwall approach. Publisher has to initialize Pollish in reward mode = true and specify if Offerwall mode should be on. When a survey is received, the publisher can show a custom prompt, to incentivize user to participate to the survey. If the user clicks on the prompt, the publisher can call Pollish show to show the questioanniare. Upon survey completion, the publisher can reward the user.


## Limitations / Minimum Requirements

This is just an initial version of the plugin. There are still some
limitations:

- You cannot pass custom attributes during initialization
- No tests implemented yet
- Minimum iOS is 9.0 and minimum Android version is 21

For other Pollfish products, see
[Pollfish docs](https://www.pollfish.com/docs).


## Getting Started

If you would like to review an example in code please review the [Github project](https://github.com/pollfish/flutter-plugin-pollfish).

