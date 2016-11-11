# Adxmi OpenRTB Integration Guide 1.0

## Overview
#### Descripition
  Adxmi provides a way for DSPs and advertisers to bid on each ad impression. This follows the OpenRTB 2.4 specification. Note that the Adxmi adaptation may require certain parameters that are optional in specification. Please read through our On-boarding Instructions and contact your account team or *support@adxmi.com* if you have any questions.

#### Limitations imposed on the OpenRTB specs
   + Campaign formats: Banner only.
   + Currency: USD only.
   + Impression: one per request.
   + CPM,CPC campaigns only.
   + Ad markup should be sent by the DSP directly within the bid reponse.
   + SSL is not support.
   + Each of multiple Exchange servers will attempt to establish persistent HTTP connections to each bidder.  HTTP Keep-Alive must be supported.

#### Basic Auction on Adxmi Exchange
  + Adxmi receives an ad request from a mobile device.
  + Adxmi sends bid request via http post(each bidder must respond within 200 ms total roundtrip).
  + Adxmi runs a first price or second price auction based on all the valid responses.
  + Adxmi pings the winning bidder's nurl(s) to notify the DSP of win.
  + Adxmi sends down the winning bid's HTML and imptrackers to the client.
  + The mobile device pings the imprackers url(s) after the ad markup is impressed.

##### Notes:
  We will never pass null values nor empty strings. If noted as not always passed, the parameter may not always be present in the request. For example, if the request doesn't contain latitude, we will omit "lat" from the request.

## Protocol
  Protocol pertains to the actual auction conversation between the Exchange and bidders as well as the mechanism for serving the winning ad.

  Please ensure you have the OpenRTB 2.4 Specification Guide available while consulting below. Our section naming will follow OpenRTB 2.4. This document will contain information and nuances specific to Adxmi's implementation. It also expects you to follow the guidelines presented in OpenRTB .

#### Bid Request

Any objects and sttributes from OpenRTB not supported by Adxmi will be noted by ~~strikethrough~~.

| Object    | Section | Description|
| ----------|:-------:| -----------|
| BidRequest| 3.2.1   |Top-level object  |
| Imp       | 3.2.2   |Container for the description of a specific impression; at least 1 per request.|
| Banner    | 3.2.3   |Details for a banner impression (incl. in-banner video) or video companion ad. |
| ~~Video~~ |~~3.2.4~~|~~Details for a video impression.~~|
|~~Audio~~  |~~3.2.5~~|~~Container for an audio impression.~~|
|~~Native~~ |~~3.2.6~~|~~Container for a native impression conforming to the Dynamic Native Ads API.~~
| Format    | 3.2.7   |An allowed size of a banner|
| ~~site~~  |~~3.2.8~~|~~Details of the website calling for the impression.~~|
| App       | 3.2.9   |Details of the application calling for the impression.|
| Publisher | 3.2.10  |Entity that controls the content of and distributes the site or app.|
|~~Content~~|~~3.2.11~~|~~Details about the published content itself, within which the ad will be shown.~~|
|~~Producer~~|~~3.2.12~~|~~Producer of the content; not necessarily the publisher (e.g., syndication).~~|
| Device    | 3.2.13  |Details of the device on which the content and impressions are displayed.|
| Geo       |3.2.14   |Location of the device or user’s home base depending on the parent object.|
| User      |3.2.15   |Human user of the device; audience for advertising.|
| ~~Data~~  |~~3.2.16~~|~~Collection of additional user targeting data from a specific data source.~~|
|~~Segment~~|~~3.2.17~~|~~Specific data point about a user from a specific data source.~~|
| Regs      |3.2.18   |Regulatory conditions in effect for all impressions in this bid request.|
| ~~Pmp~~   |~~3.2.19~~|~~Collection of private marketplace (PMP) deals applicable to this impression.~~|
| ~~Deal~~  |~~3.2.20~~|~~Deal terms pertaining to this impression between a seller and buyer.~~|

##### Object:BidRequest

| attribute | Tybe         | Description| Always passed|
| ----------|:------------:| -----------|--------------|
| id        | string       | Unique ID of the bid request, provided by the exchange. |  Yes     |
| Imp       | object array |Array of  Imp objects (Section 3.2.2) representing the impressions offered. At least 1  Imp object is required.| Yes|
| ~~site~~  |~~object~~     |~~Details via a  Site object (Section 3.2.8) about the publisher’swebsite. Only applicable and recommended for websites.~~|~~n.a~~|
| app      | object         |Details via an  App object (Section 3.2.9) about the publisher’s app (i.e., non-browser applications). Only applicable and recommended for apps. |Yes|
| device   | object         |Details via a  Device object (Section 3.2.13) about the user’s device to which the impression will be delivered.|Yes|
| user     | object         | Details via a  User object (Section 3.2.15) about the human user of the device; the advertising audience.|Yes|
| test |integer     |Indicator of test mode in which auctions are not billable, where 0 = live mode, 1 = test mode.|n.a|
|at        | integer |Auction type, where 1 = First Price, 2 = Second Price Plus.Exchange-specific auction types can be defined using values greater than 500.|Yes|
| tmax     | integer | Maximum time in milliseconds to submit a bid to avoid timeout. This value is commonly communicated offline.|Yes|
| ~~wseat~~|~~string array~~| ~~Whitelist of buyer seats (e.g., advertisers, agencies) allowed to bid on this impression. IDs of seats and knowledge of the buyer’s customers to which they refer must be coordinated between bidders and the exchange a priori. Omission implies no seat restrictions.~~|~~n.a~~|
| ~~allimps~~|~~integer~~  |~~Flag to indicate if Exchange can verify that the impressions offered represent all of the impressions available in context (e.g., all on the web page, all video spots such as pre/mid/post roll) to support road-blocking. 0 = no or unknown, 1 = yes, the impressions offered represent all that are available.~~| ~~n.a~~|
| cur       | string array | Always pass USD only  | Yes |
| bcat   | string   array   |Blocked advertiser categories using the IAB content categories. Refer to List 5.1. |No|
| badv       | string arra      |Block list of advertisers by their domains (e.g., “ford.com”).| NO|
|  bapp      | string array     |  Block list of applications by their platform-specific exchange- independent application identifiers. On Android, these should be bundle or package names (e.g., com.foo.mygame). On iOS,these are numeric IDs.| No |
| regs       |  object          |  A Regs object (Section 3.2.18) that specifies any industry, legal,or governmental regulations in force for this request. |No |
| ~~ext~~    | ~~object~~       |  ~~Placeholder for exchange-specific extensions to OpenRTB.~~| ~~n.a~~|

##### Object:imp
| attribute | Tybe         | Description| Always passed|
| ----------|:------------:| -----------|--------------|
| id        | string       | A unique identifier for this impression within the context of the bid request (typically, starts with 1 and increments.|  Yes     |
| banner    | object       | A Banner object (Section 3.2.3); required if this impression is offered as a banner ad opportunity. |Yes|
| ~~video~~ | ~~object~~   | ~~A  Video object (Section 3.2.4); required if this impression is offered as a video ad opportunity.~~|~~n.a~~|
| ~~audio~~ | ~~object~~   | ~~An Au dio object (Section 3.2.5); required if this impression is offered as an audio ad opportunity.~~|~~n.a~~|
| ~~native~~| ~~object~~   | ~~A  Native object (Section 3.2.6); required if this impression is offered as a native ad opportunity.~~|~~n.a~~|
| ~~pmp~~   | ~~object~~   |  ~~A  Pmp object (Section 3.2.19) containing any private marketplace deals in effect for this impression.~~|~~n.a~~|
| displaymanager|  string  | Will pass "Adxmi" when the sdk is present.|No|
| displaymanagerver| string| Adxmi SDK version passed from SDK, otherwise not passed|No|
| instl          | integer |1 = the ad is interstitial or full screen, 0 = not interstitial.|Yes|
| tagid          | string  | Identifier for specific ad placement or ad tag that was used to initiate the auction. This can be useful for debugging of any  issues, or for optimization by the buyer.This is known as slot id by Adxmi publishers| Yes|
| bidfloor       | float    |Minimum bid for this impression expressed in CPM.|No|
| bidfloorcur  | string  | Always pass USD only|Yes|
| ~~clickbrowser~~|~~integer~~|~~Indicates the type of browser opened upon clicking the creative in an app, where 0 = embedded, 1 = native. Note that the Safari View Controller in iOS 9.x devices is considered a native browser for purposes of this attribute.~~|~~n.a~~|
| secure    | integer |Flag to indicate if the impression requires secure HTTPS URL creative assets and markup, where 0 = non-secure, 1 = secure. If omitted, the secure state is unknown, but non-secure HTTP support can be assumed.| Yes|
| ~~iframebuster~~| ~~string array~~ | ~~Array of exchange-specific names of supported iframe busters.~~|~~n.a~~|
| ~~exp~~  | ~~integer~~ | ~~Advisory as to the number of seconds that may elapse between the auction and the actual impression.~~|~~n.a~~|
| ~~ext~~  | ~~object~~  | ~~Placeholder for exchange-specific extensions to OpenRTB.~~|~~n.a~~|

##### Object:banner
| attribute | Tybe         | Description| Always passed|
| ----------|:------------:| -----------|--------------|
| w          | integer      | Width in device independent pixels (DIPS). If no  format objects are specified, this is an exact width requirement. Otherwise it is a preferred width. | Yes|
| h          |integer       |Height in device independent pixels (DIPS).If no  format objects are specified, this is an exact height requirement. Otherwise it is a preferred height.| Yes |
| format     | object array  | Array of format objects (Section 3.2.7) representing the banner sizes permitted. If none are specified, then use of the h and  w attributes is highly recommended.| Yes |
| ~~wmax~~   | ~~integer~~ | ~~NOTE: Deprecated in favor of the  format array.Maximum width in device independent pixels (DIPS).~~|~~n.a~~|
| ~~hmax~~   | ~~integer~~ | ~~NOTE: Deprecated in favor of the  format array.Maximum height in device independent pixels (DIPS).~~|~~n.a~~|
| ~~wmin~~   | ~~integer~~ | ~~NOTE: Deprecated in favor of the  format array.Minimum width in device independent pixels (DIPS).~~|~~n.a~~|
| ~~hmin~~   | ~~integer~~ | ~~NOTE: Deprecated in favor of the  format array.Minimum height in device independent pixels (DIPS).~~|~~n.a~~|
| ~~id~~     | ~~string~~  | ~~Unique identifier for this banner object. Recommended when Banner objects are used with a Video object (Section  3.2.4) to represent an array of companion ads. Values usually start at 1 and increase with each object; should be unique within an impression.~~|~~n.a~~|
| btype  | integer array|  Blocked banner ad types. Refer to List 5.2.|Yes|
| battr  | integer array|  Blocked creative attributes. Refer to List 5.3. |Yes|
| pos    | integer      |  Ad position on screen. Refer to List 5.4.|
| ~~mimes~~  | ~~string array~~ | ~~Content MIME types supported. Popular MIME types may include “ application/x-shockwave-flash ”,“ image/jpg ”, and “ image/gif ”.~~|~~n.a~~|
| ~~topframe~~  |  ~~integer~~  | ~~Indicates if the banner is in the top frame as opposed to an iframe, where 0 = no, 1 = yes.~~|~~n.a~~|
| ~~expdir~~| ~~integer array~~ | ~~Directions in which the banner may expand. Refer to List 5.5.~~|~~n.a~~|
| api       |integer array      | Currently Adxmi support MRAID-2 Only |Yes|
| ~~ext~~   | ~~object~~        | ~~Placeholder for exchange-specific extensions to OpenRTB.~~|~~n.a~~|

##### Object:Format
| attribute | Tybe         | Description| Always passed|
| ----------|:------------:| -----------|--------------|
| w         | integer      | Width in device independent pixels (DIPS).|Yes|
| h         | integer      | Height in device independent pixels (DIPS). |Yes|
| ~~ext~~   | ~~object~~   | ~~Placeholder for exchange-specific extensions to OpenRTB.~~|~~n.a~~|

##### Object:App
| attribute | Tybe         | Description| Always passed|
| ----------|:------------:| -----------|--------------|
| id        | string       | Exchange-specific app ID.|Yes|
| name      | string       | App name (may be aliased at the publisher’s request).|Yes|
| bundle    | string       | A platform-specific application identifier intended to be unique to the app and independent of the exchange. On Android, this should be a bundle or package name (e.g., com.foo.mygame). On iOS, it is a numeric ID.|Yes|
| ~~domain~~| ~~string~~   | ~~Domain of the app (e.g., “mygame.foo.com”).~~|~~n.a~~|
| storeurl  |  string      | App store URL for an installed app; for IQG 2.1 compliance.| No|
| cat       | string array | Array of IAB content categories of the app. Refer to List 5.1.|Yes|
|~~sectioncat~~ | ~~string array~~ | ~~Array of IAB content categories that describe the current section of the app. Refer to List 5.1.~~|~~n.a~~|
| ~~pagecat~~   | ~~string array~~ | ~~Array of IAB content categories that describe the current page or view of the app. Refer to List 5.1.~~|~~n.a~~|
| ver           | string          | Application version.|No|
| ~~privacypolicy~~ | ~~integer~~ | ~~Indicates if the app has a privacy policy, where 0 = no, 1 = yes.~~|~~n.a~~|
| ~~paid~~      | ~~integer~~     | ~~0 = app is free, 1 = the app is a paid version.~~|~~n.a~~|
| publisher  |object | Details about the Publisher (Section 3.2.10) of the app.|yes|
| ~~content~~| ~~object~~| ~~Details about the Content (Section 3.2.11) within the app.~~|~~n.a~~|
| ~~keywords~~| ~~string~~| ~~Comma separated list of keywords about the app.~~|~~n.a~~|
| ~~ext~~     | ~~object~~| ~~Placeholder for exchange-specific extensions to OpenRTB.~~|~~n.a~~|

##### Object:Publisher
| attribute | Tybe         | Description| Always passed|
| ----------|:------------:| -----------|--------------|
| id        | string       | Exchange-specific publisher ID.|Yes|
| ~~name~~  | ~~string~~   | ~~Publisher name (may be aliased at the publisher’s request).~~|~~n.a~~|
| ~~cat~~   |~~string array~~|~~Array of IAB content categories that describe the publisher.Refer to List 5.1.~~|~~n.a~~|
| ~~domain~~| ~~string~~     | ~~Highest level domain of the publisher (e.g., “publisher.com”).~~|~~n.a~~|
| ~~ext~~  | ~~object~~     | ~~Placeholder for exchange-specific extensions to OpenRTB.~~|~~n.a~~|

##### Object:Device
| attribute | Tybe         | Description| Always passed|
| ----------|:------------:| -----------|--------------|
|  ua       | string       |Browser user agent string.|Yes|
|  geo      | object       |Location of the device assumed to be the user’s current location defined by a  Geo object (Section 3.2.14).|Yes|
| dnt       | integer      | Standard “Do Not Track” flag as set in the header by the browser, where 0 = tracking is unrestricted, 1 = do not track.|Yes|
| lmt       | integer      |“Limit Ad Tracking” signal commercially endorsed (e.g., iOS,Android), where 0 = tracking is unrestricted, 1 = tracking must be limited per commercial guidelines.|Yes|
| ip        | string       | IPv4 address closest to device.|Yes|
| ~~ipv6~~  | ~~string~~   | ~~IP address closest to device as IPv6.~~|~~n.a~~|
| devicetype| integer      | The general type of device. Refer to List 5.19.make  string  Device make (e.g., “Apple”).|No|
| model     | string       | Device model (e.g., “iPhone”).os  string  Device operating system (e.g., “iOS”).osv  string  Device operating system version (e.g., “3.1.2”).|No|
| hwv       | string       | Hardware version of the device (e.g., “5S” for iPhone 5S).|No|
| h          | integer      | Physical height of the screen in pixels.|No|
| w          | integer      | Physical width of the screen in pixels. |No|
| ppi        | integer      | Screen size as pixels per linear inch.  |No|
| pxratio    | float        | The ratio of physical pixels to device independent pixels.|No|
| js         | integer      | Support for JavaScript, where 0 = no, 1 = yes.|Yes|
| ~~geofetch~~| ~~integer~~|  ~~Indicates if the geolocation API will be available to JavaScript code running in the banner, where 0 = no, 1 = yes.~~|~~n.a~~|
| ~~flashver~~|~~string~~  | ~~Version of Flash supported by the browser.~~|~~n.a~~|
| language    | string     | Browser language using ISO-639-1-alpha-2~|No|
| carrier     | string     | Carrier or ISP (e.g., “VERIZON”). “WIFI” is often used in mobile to indicate high bandwidth (e.g., video friendly vs. cellular).|No|
| connectiontype | integer | Network connection type. Refer to List 5.20.|No|
| ifa            |string   | ID sanctioned for advertiser use in the clear (i.e., not hashed). |No|
| didsha1        | string  | Hardware device ID (e.g., IMEI); hashed via SHA1.|No|
| didmd5         | string  | Hardware device ID (e.g., IMEI); hashed via MD5.|No|
| dpidsha1       | string  | Platform device ID (e.g., Android ID); hashed via SHA1.|No|
| dpidmd5        | string  | Platform device ID (e.g., Android ID); hashed via MD5.|No|
| macsha1        | string  | MAC address of the device; hashed via SHA1.|No|
| macmd5         | string  | MAC address of the device; hashed via MD5.|No|
| ~~ext~~        | ~~object~~| ~~laceholder for exchange-specific extensions to OpenRTB.~~|~~n.a~~|

##### Object:Geo
| attribute | Tybe         | Description| Always passed|
| ----------|:------------:| -----------|--------------|
| lat       | float        | Latitude from -90.0 to +90.0, where negative is south.|No|
| lon       | float        | Longitude from -180.0 to +180.0, where negative is west.|No|
| type      | integer       | Source of location data; recommended when passing lat / lon . Refer to List 5.18.|No|
|~~accuracy~~| ~~integer~~ | ~~Estimated location accuracy in meters; recommended when lat/lon are specified and derived from a device’s location services (i.e., type = 1). Note that this is the accuracy as reported from the device. Consult OS specific documentation (e.g., Android, iOS) for exact interpretation.~~ | ~~n.a~~|
|~~lastfix~~ | ~~integer~~ | ~~Number of seconds since this geolocation fix was established.Note that devices may cache location data across multiple fetches. Ideally, this value should be from the time the actual fix was taken.~~|~~n.a~~|
|ipservice |integer  | Service or provider used to determine geolocation from IP address if applicable (i.e., type = 2). Refer to List 5.21.| n.a |
| country   | string        | Country code using ISO-3166-1-alpha-3.|Yes|
| ~~region~~| ~~string~~    | ~~Region code using ISO-3166-2; 2-letter state code if USA.~~| ~~n.a~~|
| ~~regionfips104~~| ~~string~~| ~~Region of a country using FIPS 10-4 notation. While OpenRTB supports this attribute, it has been withdrawn by NIST in 2008.~~|~~n.a~~|
| metro     | string        | Google metro code; similar to but not exactly Nielsen DMAs.See Appendix A for a link to the codes.|No|
| city      | string        | City using United Nations Code for Trade & Transport Locations. See Appendix A for a link to the codes.|No|
| ~~zip~~   | ~~string~~    | ~~Zip or postal code.~~|~~n.a~~|
| utcoffset | integer       | Local time as the number +/- of minutes from UTC.|No|
|~~ext~~    | ~~object~~    | ~~Placeholder for exchange-specific extensions to OpenRTB.~~|~~n.a~~|

##### Object:User
| attribute | Tybe         | Description| Always passed|
| ----------|:------------:| -----------|--------------|
| id        | string       | Exchange-specific ID for the user. At least one of  id or buyeruid is recommended.|Yes|
|~~buyeruid~~| ~~string~~  | ~~Buyer-specific ID for the user as mapped by the exchange for the buyer. At least one of  buyeruid or  id is recommended.~~|~~n.a~~|
| yob       | integer      | Year of birth as a 4-digit integer.|No|
| gender    | string       | Gender, where “M” = male, “F” = female, “O” = known to be other (i.e., omitted is unknown).|No|
| keywords  | string       | Comma separated list of keywords, interests, or intent.|No|
| ~~customdata~~| ~~string~~  | ~~Optional feature to pass bidder data that was set in the exchange’s cookie. The string must be in base85 cookie safe characters and be in any format. Proper JSON encoding must be used to include “escaped” quotation marks.~~|~~n.a~~|
| geo       | object       | Location of the user’s home base defined by a  Geo object (Section 3.2.14). This is not necessarily their current location.|Yes|
| ~~data~~  | ~~object array~~ | ~~Additional user data. Each  Data object (Section 3.2.16) represents a different data source.~~|~~n.a~~|
| ~~ext~~   | ~~object~~   | ~~Placeholder for exchange-specific extensions to OpenRTB.~~|~~n.a~~|

##### Object:Regs
| attribute | Tybe         | Description| Always passed|
| ----------|:------------:| -----------|--------------|
| coppa     | integer      | Flag indicating if this request is subject to the COPPA regulations established by the USA FTC, where 0 = no, 1 = yes. Field will only be passed when coppa=1.|No|
| ~~ext~~   | ~~object~~   | ~~Placeholder for exchange-specific extensions to OpenRTB.~~|~~n.a~~|



#### Bid Response Specification
Note that:
+ Below section naming will follow OpenRTB for simplicity.
+ The types are unchanged from OpenRTB specification.
+ Any object and attributes not supported by Adxmi will be noted by a ~~strikethrough~~

| Object    | Section | Description|
| ----------|:-------:| -----------|
|BidResponse|4.2.1    | Top-level object.|
| SeatBid   |4.2.2    |Collection of bids made by the bidder on behalf of a specific seat.|
| Bid       | 4.2.3   | An offer to buy a specific impression under certain business terms.|

##### Object:BidResponse
| Object    | Type; Requirement   | Description|
| ----------|:-------------------:| -----------|
| id        | string; required    | ID of the bid request to which this is a response.|
| seatbid   | object array        | Array of seatbid objects; 1+ required if a bid is to be made.|
| bidid     | string              |Bidder generated response ID to assist with logging/tracking.|
| cur       | string;default “USD”|**Currently only accepts and defaults to "USD"**
|~~customdata~~| ~~string~~       | ~~Optionalfeature to allow a bidder to set data in the exchange’s cookie. The string must be in base85 cookie safe characters and be in any format. Proper JSON encoding must be used to include “escaped” quotation marks.~~|
| nbr        | integer          | Reason for not bidding. Refer to List 5.22.|
| ~~ext~~    | ~~object~~       | ~~Placeholder for bidder-specific extensions to OpenRTB.~~|

##### Object:SeatBid
| Object    | Type; Requirement   | Description|
| ----------|:-------------------:| -----------|
| bid       |object array;required|Array of 1+  Bid objects (Section 4.2.3) each related to an impression. Multiple bids can relate to the same impression.|
| seat      | string;recommended (required in some case)|  ID of the buyer seat (e.g., advertiser, agency) on whose behalf this bid is made.Required by DSPs who hace multiple buyers seats using their platform|
| ~~group~~ | ~~integer;default 0~~|~~0 = impressions can be won individually; 1 = impressions must be won or lost as a group.~~|
| ~~ext~~   | ~~object~~| ~~Placeholder for bidder-specific extensions to OpenRTB.~~|

##### Object:Bid
| Object    | Type; Requirement   | Description|
| ----------|:-------------------:| -----------|
| id        | string; required    | Bidder generated bid ID to assist with logging/tracking.|
| impid     | string; required    | ID of the  Imp object in the related bid request.|
| price     | float; required     | Bid price expressed as CPM although the actual transaction isfor a unit impression only. Note that while the type indicates float, integer math is highly recommended when handling currencies (e.g., BigDecimal in Java).|
| adid      | string              |ID of a preloaded ad to be served if the bid wins.|
| nurl      |string               | Win notice URL called by the exchange if the bid wins (notnecessarily indicative of a delivered, viewed, or billable ad);optional means of serving ad markup.
| adm       |string; required     | **Slot of the ad markup directly in the bid response is the only method suppoted by Adxmi**|
|~~adomain~~|~~string array~~     | ~~Advertiser domain for block list checking (e.g., “ford.com”).This can be an array of for the case of rotating creatives.Exchanges can mandate that only one domain is allowed.~~|
| bundle   | string               | A platform-specific application identifier intended to beunique to the app and independent of the exchange. On Android, this should be a bundle or package name (e.g., com.foo.mygame). On iOS, it is a numeric ID.|
| iurl     | string; recommended  | URL without cache-busting to an image that is representative of the content of the campaign for ad quality/safety checking.|
| cid      | string; recommended  | Campaign ID to assist with ad quality checking; the collectionof creatives for which  iurl should be representative.|
| crid     |string; recommended   | Creative ID to assist with ad quality checking.|
| cat      |string array;recommended| IAB content categories of the creative. Refer to List 5.1. 
| attr     | integer array        |Set of attributes describing the creative. Refer to List 5.3.|
| api      |integer                  |API required by the markup if applicable. Refer to List 5.6.|
|~~protocol~~|~~integer~~            | ~~Video response protocol of the markup if applicable. Refer to List 5.8.~~|
| ~~qagmediarating~~| ~~integer~~   | ~~Creative media rating per IQG guidelines. Refer to List 5.17.~~|
| dealid     | string; requried for pmp | Reference to the  deal.id from the bid request if this bid pertains to a private marketplace direct deal.
| w          | integer               | Width of the creative in device independent pixels (DIPS).|
| h          | integer               |Height of the creative in device independent pixels (DIPS).|
| ~~exp~~    | ~~integer~~           | ~~Advisory as to the number of seconds the bidder is willing to wait between the auction and the actual impression.~~|
| ~~ext~~    | ~~object~~            | ~~Placeholder for bidder-specific extensions to OpenRTB.~~|


##### Substitution Macros
| Object           | Type; Requirement   |
| -----------------|:-------------------:|
|${AUCTION_ID}     | ID of the bid request; from  BidRequest.id attribute.
| ${AUCTION_BID_ID}| ID of the bid; from  BidResponse.bidid attribute.
|${AUCTION_IMP_ID} | ID of the impression just won; from  imp.id attribute.
|${AUCTION_SEAT_ID} | ID of the bidder seat for whom the bid was made.
|${AUCTION_AD_ID}   |ID of the ad markup the bidder wishes to serve; from  bid.adid attribute.|
|${AUCTION_PRICE}   |Settlement price using the same currency and units as the bid.|
|${AUCTION_CURRENCY}|  The currency used in the bid (explicit or implied); for confirmation only.|

Notice that:
+ All marcos must be formatted as \${MACRO_NAME}  or  ${macro_name}


##### Sending nbr when not biding
  Preferred for bidder to send a bare minimum bid response object when not bidding but providing "nbr".
  In this case the bidder would send back an HTTP response code 200 with this barebones response object.



## Bid Response Examples
#### Banner Bid Response
```
  {
   “id”:”1234567890″,
   “bidid”:”abc123″,
   “cur”:”USD”,
     “seatbid”:[
      {
         “seat”:”512″,
         “bid”:[
               “id”:”12345678″,
               “impid”:”102″,
               “price”:”9.43″,
               “nurl”:”http://adserver.com/winnotice?impid=102″,
               “iurl”:”"http://adserver.com/pathtosampleimage″,
               “cid”:”"campaign111″,
               “crid”:”"creative112″,
               “adid”:”12345″,
               “cat”:[
                  “IAB3”
               ],
               “adm”:”%3Cdiv%3Eexample+adm%3C%2Fdiv%3E”,
               “attr”:[
                  4,
                  7
                       ],
               “api”:”5″,
               “h”:480,
               “w”:320,
               “dealid”:”xyz123″,
         ]
      }
   ]
}
```