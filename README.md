# swyft-app-ios-sdk

This project is to be used by a 3rd party iOS application, inorder to integrate with a Swyft Vission Cabinet


### Table of Contents 
- [Setup](#setup)
  - [Request SDK Credentials](#creds)
  - [Installation](#install)
- [Usage](#usage)
  - [SDK Initilization](#init)
  - [User Enrollment](#enroll) 
  - [User Authentication](#auth)
  - [User Update](#userUpdate)
  - [Get User Orders](#orders)
  - [Get User Payment Methods](#pMethods)
  - [Add User Payment Method](#addPMethod)
  - [Update User Payment Method](#updatePMethod)
  - [Set User Default Payment Method](#setDefaultPMethod)
  - [Delete User Payment Method](#removePMethod)
- [Webhooks](#webhooks)
  - [Session Begins](#webhookBegins)
  - [Session Ends](#webhookEnd)

<a name="setup"/>

## Setup 

Before using the Swyft SDK you will need request access credentials and install the library into your project

<a name="creds"/>

### Request SDK credentials 

In order to use the SDK you need to request credentials by supplying swyft with your Applications Bundle ID. The credentials will should be stored in a plist file that you will need to include in your applications build path. 

Sample plist entry
```xml
<key>SWYFT_SDK_AUTH_KEY</key>
<string>fasdfasfef3cevwe3qf3q4fewvq3f34tg4gv4v4v45v</string>
```
<a name="install"/>

### Installation

To include the Swyft SDK in your project, add the following to your Podfile

```javascript
source 'https://github.com/CocoaPods/Specs.git'
source 'https://github.com/swyftstore/swyft-app-ios-sdk-cocoa-pod.git'

target 'YourAppName' do
 # Comment the next line if you don't want to use dynamic frameworks
  use_frameworks!
  # SwyftSdk
  pod 'SwyftSdk', '1.0.0-alpha'
end  
```

Note: we haven't decided if we want to publish our sdk to cocopods or distribute through more closed channels 

<a name="usage"/>

## Usage

Below you will find some code examples for how you can intergate the sdk with your application

<a name="init"/>

### SDK Initilization

The first step when integrating the Swyft SDK with your project is to the initializion method on the SDK. We recommend doing this in your AppDelegate's application method.

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Override point for customization after application launch.

    SwyftSdk.initSDK()

    return true
}
```

<a name="enroll"/>

### Enroll User 

The next step is to enroll your applications user with Swyft. This is used to create a profile for your user on Swyft's Platform. The method returns a swyftId that is used to authenticate the user later on. If the user already exist the SDK blocks the creation of a duplicate user and returns the swyftId successfully.
```swift
let user = SwyftUser(
    email: "user@swyftstore.com",
    firstName: "John",
    lastName: "Smith",
    phoneNumber: "+11234567890")
        
SwyftSdk.enrollUser(user: user, success: { response in  
    //store swyftId for later usage
    self.swyftId = response.swyftId                      
}) { error in
    debugPrint(error)        
}
    
```

<a name="auth"/>

### User Authentication 

After you have enrolled a user you can authenticate the user. This creates a session for the user so they can interact with the Swyft Vision Cabinet by scanning a returned QR Code. You can either supply an authentication string for the SDK to display as a QR Code or have the SDK generate a dynamic one for you. You can set the qrCode foreground color by passing in the UIColor value you wish to use.

- Auto Generated QR Code
```swift
SwyftSdk.authenticateUser(swyftId: self.swyftId, qrCodeColor: self.view.tintColor, customAuth: customAuth, success: { response in
  DispatchQueue.main.async {
      //display the returned qrCode UIImage
      self.qrCodeImage.image = response.qrCode
  }

}) { error in
  debugPrint(error)
}
``` 
- Custom Generated QR Code
```swift
SwyftSdk.authenticateUser(swyftId: self.swyftId, qrCodeColor: self.view.tintColor, success: { response in                        
    DispatchQueue.main.async {
        //display the returned qrCode UIImage
        self.qrCodeImage.image = response.qrCode
    }

}) { error in
    debugPrint(error)
}
```

<a name="userUpdate"/>

### Update User 

After enrolling or authenticating a user you can update their profile. You will get an error if the new email is already in use.
```swift
let user = SwyftUser(
    email: "user@swyftstore.com",
    firstName: "John",
    lastName: "Smith",
    phoneNumber: "+11234567890")
        
SwyftSdk.updateUser(user: swyftUser, success: { (response) in  
    debugPrint(response.message)
}) { (error) in
    debugPrint(error)
}
    
```  

<a name="orders"/>

### Get User Orders

After athenticating a user you can retrieve their past orders. Orders will be returned in a paginated list. 
```swift
let start = 1
let pageSize = 20

SwyftSdk.getOrders(start: start, pageSize: pageSize, success: { response in
  //store/display orders
   self.orders = response.orders

}) { error in
  print(error)
}
```
<a name="pMethods"/>

### Get User Payment Methods

If Swyft handling the payment processing for your integration, after you authenticate the user you can retreive a list of their previously enrolled payment methods
```swift
SwyftSdk.getPaymentMethods(success: { response in
  //store/display payment methods orders
  self.paymentMethods = response.paymentMethods
}) { error in
   print(error)
}
```

<a name="addPMethod"/>

### Add User Payment Methods

If Swyft handling the payment processing for your integration, after you authenticate the user you can add additional payment methods for the user
```swift
//build method
let fullMethod = FullPaymentMethod(
cardNumber: cardNumber,
cardExpiry: cardExpiry,
cardType: cardType,
cardHolderName: cardHolderName,
cvv: cvv)

SwyftSdk.addPaymentMethod(method: fullMethod,
                       isDefault: isDefault, //This is used to set the payment method as the 'default' method. If this  
                                             //is the first/only method for the user it is ALWAYS treated as true, 
                       success: { response in
  //store new method
  let method = response.paymentMethod
}) { error in
  print(error)
}
```
<a name="updatePMethod"/>

### Update User Payment Methods

If Swyft handling the payment processing for your integration, after you authenticate the user you can update payment methods for the user
```swift
//load previously stored payment method you would like to change
let method = paymentMethods[0]
//map SwyftPaymentMethod to FullPaymentMethod
let fullMethod = FullPaymentMethod(from: method)
//change the values you wish to change
fullMethod.cvv = "111"
fullMethod.cardNumber = "4111111111112222"
SwyftSdk.editPaymentMethod(method: fullMethod, success: { response in
    //update methods stored in memory

}) { error in
    print(error)
}
```
<a name="setDefaultPMethod"/>

### Set User Default Payment Method

If Swyft is handling the payment processing for your integration, after you authenticate the user you can set the default payment method
```java
//load previously stored payment method you wish to set as the default method
let method = self.paymentMethods[indexPath.row]
SwyftSdk.setDefaultPaymentMethod(defaultMethod: method, success: { response in
    //update ui to show new default method
}) { error in
    print(error)
}
```
<a name="removePMethod"/>

### Delete User Payment Method

If Swyft is handling the payment processing for your integration, after you authenticate the user you can delete a payment method
```java
//load previously stored payment method you wish to set as the default method
let method = self.paymentMethods[indexPath.row]

SwyftSdk.deletePaymentMethod(deleteMethod: method, success: { response in
    //update list of payment methods

}) { error in
    print(error)
}
```

<a name="webhooks"/>

## Webhooks

Swyft offers a pair of webhooks that you can choose to integrate with along with the SDK. The webhooks can alert you eachtime one of your users begin a session with a Swyft Vision Cabinet, as well as once the session ends and its transaction details. If you wish to intgrate with the Swyft Vision Cabinet webhooks below are the payloads you can expect once you supply Swyft with your end points. 

<a name="webhookBegins"/>

### Session Begins

Session Begins WebHook Payload
```javascript
headers: {
   Content-Type: "application/json",
   ts: String, //epoch timestamp
   signature: String//hmac security token
}
payload: {
  email_address:String,
  preauth_amount: String,
  storeId: String,
  customerId: String
}

```

<a name="webhookEnd"/>

### Session Ends

Session Ends WebHook Payload
```javascript
headers: {
   Content-Type: "application/json",
   ts: String, //epoch timestamp
   signature: String //hmac security token
}

payload: {
  email_address:String,
  products: Array,
  store_id: String,
  customer_id: String,
  subtotal: String,
  tax: String,
  total: String
}
```
