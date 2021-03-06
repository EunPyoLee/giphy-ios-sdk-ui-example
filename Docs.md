# GIPHY SDK for iOS
 

## _Table of contents_
**Setup**
- [Requirements](#requirements) 
- [CocoaPods](#cocoapods) 
- [Carthage](#carthage) 
- [API Key](#configure-your-api-key) <br>
- [Customization](#custom-ui)  

**Templates**
- [GiphyViewController](#giphyviewcontroller)
- [Settings](#template-settings)
    - [Layout](#layout)
    - [Media Types](#media-types)  
    - [Theme](#theme) 
- [GiphyDelegate](#events)

**GPHMedia**
- [GPHMediaView](#gphmediaview)
- [Media IDs](#media-ids) 

**Caching & Dependencies** 
- [Caching](#caching)
- [Dependencies](#dependencies)

**The Grid**
- [GiphyGridController](#grid-only-and-the-giphygridcontroller-setup)
- [Presentation](#giphygridcontroller-presentation) 
- [GPHContent](#giphygridcontroller-gphcontent)
- [GPHGridDelegate](#giphygridcontroller-gphgriddelegate)

### Requirements 
- iOS 10 or later  
- Cocoapods or Carthage 
- A Giphy API key from the [Giphy Developer Portal](https://developers.giphy.com/dashboard/?create=true). 
- Xcode 11.3 and later  

### Github Example Repo 
- Run the example app to see the GIPHY SDK in action with all of its configurations. Run `pod install` and set your API key [here](https://github.com/Giphy/giphy-ios-sdk-ui-example/blob/master/Swift/Example/ViewController.swift#L113) before building the example app. 
- Open [issues or feature requests](https://github.com/Giphy/giphy-ios-sdk-ui-example/issues)  
- View [releases](https://github.com/Giphy/giphy-ios-sdk-ui-example/releases)

### CocoaPods

Add the GiphyUISDK to your Podfile like so: 

```swift 
use_frameworks!

target "YourAppTarget" do 
pod 'Giphy' 
end
```
**Note**: for pure Objective-C projects, add an empty swift file to your project and choose `Create the Bridging Header` when prompted by Xcode. This allows static libraries to be linked.

### Carthage

1. Add GiphySDK to your Cartfile like so:
```
binary "https://s3.amazonaws.com/sdk.mobile.giphy.com/SDK/GiphySDK.json"
```
2. Add `-ObjC` to the `Other Linker Flags` Build Setting

3. Add `PINCache.framework`, `DeepDiff.framework`, `GiphyCoreSDK.framework`, `GiphyUISDK.framework`, and `WebP.framework` from `Carthage/Build/iOS` into your project. 

4. Create a new Run Script under Build Phases (See step 8 of the directions [here](https://github.com/Carthage/Carthage#quick-start)). Only 2 of the frameworks need to be added to the run script (The rest can be excluded from this step as they are static frameworks):   
- `PINCache.framework`  
- `DeepDiff.framework` 

### Configure your API key 

First things first, be sure to import: 
```swift
import GiphyUISDK
import GiphyCoreSDK
```

Configure your API key. Apply for an API key [here](https://developers.giphy.com/dashboard/).
```swift
Giphy.configure(apiKey: "yOuR_kEy_HeRe")
```

## Custom UI

We offer two solutions for the SDK user interface - pre-built templates which handle the entirety of the GIPHY experience, and a [Grid-Only implementation](https://developers.giphy.com/docs/sdk#grid) which allows for endless customization.  

See [customization](https://developers.giphy.com/docs/sdk#grid) to determine what's best for you.
 
_Skip ahead to [Grid-Only section](#grid-only-and-the-giphygridcontroller-setup)_

### GiphyViewController 

Create a new `GiphyViewController`, which takes care of most of the magic.  
```swift
let giphy = GiphyViewController()
```

Create a new `GiphyViewController` every time you want to show GIPHY (maintaining a reference to the same `GiphyViewController` object isn't necesssary and can impact performance and lead to unexpected results) 

## Template Settings
 
### _Media Types_

Set the content type(s) you'd like to show by setting the `mediaTypeConfig` property, which is an array of `GPHContentType`s.

**Note**: Emoji-only is not available for the carousel layout option. 
```swift
giphy.mediaTypeConfig = [.gifs, .stickers, .text, .emoji]
```
Objective-C: 
```Objective-C
[giphy setMediaConfigWithTypes: [[ NSMutableArray alloc] initWithObjects: 
@(GPHContentTypeGifs), @(GPHContentTypeStickers), @(GPHContentTypeText), @(GPHContentTypeEmoji), nil ] ]; 
```
### _Recently Picked_

As of `v1.2.5`, you can add an additional `GPHContentType` to the `mediaTypeConfig` array: `.recents`
<img src="images/recents.PNG" width="281.109445275" height="500">

```swift
giphy.mediaTypeConfig = [.gifs, .stickers, .recents]
``` 

GIFs that are selected by the user are automatically added to the recents tab, which is only displayed if the user has previously picked a gif. 

Users can remove gifs from recents with a long-press on the GIF in the recents grid. 

### _Theme_

Set the theme type (`GPHThemeType`) to be `.dark`, `.light`, `.lightBlur`, `.darkBlur` or `.automatic`. The `automatic` option follows the current Dark Mode setting of the device. 
 
```swift
giphy.theme = GPHTheme(type: .lightBlur) 
```
For video editing apps, we recommend trying out `.lightBlur` or `.darkBlur` themes. <br>
<img src="images/light-blur.PNG" width="281.109445275" height="500">
<img src="images/dark-blur.PNG" width="281.109445275" height="500">

### _Extending GPHTheme_

As of version 1.2.8, you can also subclass `GPHTheme` to override visual properties like font and colors, so as to apply the visual language of your app. 

```swift 
public class CustomTheme: GPHTheme {
    public override init() {
        super.init()
        self.type = .light
    }
    
    public override var textFieldFont: UIFont? {
        return UIFont.italicSystemFont(ofSize: 15.0)
    }

    public override var textColor: UIColor {
        return .black
    }  
}

```
### _Additional Settings_ 
- **Sticker Column Count**: For carousel layouts, we provide the option to set the number of columns for stickers and text. Possible `GPHStickerColumnCount`values are `.two`, `.three`. and `.four`. We recommend going for 3 or 4 columns when leveraging the blur `GPHThemeType`. 

```
giphy.stickerColumnCount = GPHStickerColumnCount.three 
```


- **Confirmation screen**:  we provide the option to show a secondary confirmation screen when the user taps a GIF, which shows a larger rendition of the asset.
This confirmation screen is only available for `.waterfall` mode - this property will be ignored if the `layout` is `.carousel`. 
```swift
giphy.showConfirmationScreen = true 
```

- **Rating**: set a specific content rating for the search results. Default `ratedPG13`.
```swift
giphy.rating = .ratedPG13
```

- **Rendition**:  option to select the rendition type for the grid.  Default `fixedWidth`.
```swift
giphy.renditionType = .fixedWidth 
```

- **Localization**:  option to choose whether or not to localize the search results based on phone settings. Default `false` will set the language to `en`.
```swift
giphy.shouldLocalizeSearch = false
```

- **Tray Height**: height for the tray's "snap point" as a ratio of the `GiphyViewController`'s height. Default `0.7`
```swift
GiphyViewController.trayHeightMultiplier = 0.7 
```

### _Presentation_ 

Present the `GiphyViewController` and watch as the GIFs start flowin'.
```swift
present(giphy, animated: true, completion: nil)
```

### _Events_
Set the delegate and conform to the `GiphyDelegate` protocol to handle GIF selection.
```swift
giphy.delegate = self
```

```swift
extension YourController: GiphyDelegate { 
   func didSelectMedia(giphyViewController: GiphyViewController, media: GPHMedia)   {
   
        // your user tapped a GIF!   
        giphyViewController.dismiss(animated: true, completion: nil) 
   }
   
   func didDismiss(controller: GiphyViewController?) {
        // your user dismissed the controller without selecting a GIF.  
   }
}
```

From there, it's up to you to decide what to do with the GIF!  


### _GPHMediaView_

Create a `GPHMediaView` to display the media: 

```swift
let mediaView = GPHMediaView() 
mediaView.media = media  
```

Use the media's `aspectRatio` property to size the view: 
```swift
let aspectRatio = media.aspectRatio 
```

If you want to build your own view to display a GIF, grab a URL to the asset like so:  
```swift
let webpURL = media.url(rendition: .original, fileType: .webp) 
let gifURL = media.url(rendition: .fixedWidth, fileType: .gif) 
let vidURL = media.url(rendition: .fixedWidth, fileType: .mp4) 

let url = URL(string: gifURL) 
```

### _Media IDs_

In a messaging app context, you may want to send media `id`s rather than `GPHMedia` objects or image assets. 

Obtain a `GPHMedia`'s `id` property via `media.id` 

On the receiving end, obtain a `GPHMedia` from the `id` like so: 
 
```swift
GiphyCore.shared.gifByID(id) { (response, error) in
    if let media = response?.data {
        DispatchQueue.main.sync { [weak self] in 
            self?.mediaView.media = media
        }
    }
}
``` 

### _Caching_ 
We use [PINCache](https://github.com/pinterest/PINCache) to cache media assets, which reduces unnecessary image requests and loading times.

By default, we use both PINCache’s memory cache and disk cache. The disk cache is limited to 300 mb by default, but you can set it to any value you’d like: 

```swift
// set to 300 mb 
GPHCache.shared.diskCacheByteLimit = 300 * 1000 * 1000  
```
If you only want to cache GIFs in memory, set 
`GPHCache.shared.setting` to `memoryOnly` like so 
```swift
GPHCache.shared.setting = .memoryOnly 
``` 
Similarly, you can cache only to disk with: 
```swift
GPHCache.shared.setting = .diskOnly 
```
Note: We *don't* automatically clear the cache when the `GiphyViewController` is dismissed. 
Manually clear the cache on your convenience by calling `GPHCache.shared.clear()` to clear the cache based on your setting, or `GPHCache.shared.clear(.memoryOnly)` to choose a specific cache you want to clear.

The cache stores NSData objects for the images (the SDK displays .webp files by default). You can get the raw image data yourself via: 

```swift

guard let url = media.url(rendition: .fixedWidth, fileType: .webp) else { return } 
GPHCache.shared.downloadAssetData(url) { (data, error) in
}
```
#### *Dependencies*  
[PINCache](https://github.com/pinterest/PINCache): image caching <br> 
[YYImage](https://github.com/ibireme/YYImage): GIF playback <br>
[libwebp](https://github.com/webmproject/libwebp): webp playback <br>
[DeepDiff](https://github.com/onmyway133/DeepDiff): Collection view diffing algorithm <br> 


#### *Buttons*

Download the Sketch file [here](https://s3.amazonaws.com/sdk.mobile.giphy.com/design/GIPHY-SDK-UI-Kit.sketch) if you're looking for a great button icon to prompt the GIPHY SDK experience.  

## Grid-Only and the GiphyGridController Setup

The following section refers to the Grid-Only solution of the SDK. Learn more [here](https://developers.giphy.com/docs/sdk#grid)

_See the [Template section](#templates-via-giphyviewcontroller) for template setup instructions._

The `GiphyGridController` takes care of requesting content from the GIPHY API, loading, caching, and rendering images in a customizable grid, and leaves the rest of the experience up to you. 

The `GiphyGridController` offers more customization of the grid than the `GiphyViewController`, via the `numberOfTracks`, `cellPadding`, and `direction` properties, all of which apply only to the `waterfall` layout. 

Create a new grid controller: 
```
let gridController = GiphyGridController()
```

Customize the grid design: 
```
// space between cells 
gridController.cellPadding = 2.0

// the scroll direction of the grid 
gridController.direction = .vertical 

// the number of "tracks" is the span count. it represents num columns for vertical grids & num rows for horizontal grids 
gridController.numberOfTracks = 3

// hide the checkered background for stickers if you'd like (true by default) 
gridController.showCheckeredBackground = false 
gridController.view.backgroundColor = .lightGray

// by default, the waterfall layout sizes cells according to the aspect ratio of the media 
// the fixedSizeCells setting makes it so each cell is square
// this setting only applies to Stickers (not GIFs) 
gridController.fixedSizeCells = true 


``` 

### GiphyGridController: Presentation

Unlike the `GiphyViewController`, the `GiphyGridController` is not by itself a fully functional component, and must exist alongside other UI in order to offer a meaningful user experience. We recommend embedding the `GiphyGridController` inside of another `UIViewController` by adding it as a child view controller, adding its subview, and constraining it according to your design. 

*Important*
For performance reasons, a new `GiphyGridController` should be created _every time_ the Giphy search experience is presented, and the instance should be always set to nil when it is dismissed. Ensure that there is only ever one instance of a `GiphyGridController` allocated at a given screen - multiple instances of `GiphyGridController`s may not be added to the same screen. 

```swift
addChild(gridController)
view.addSubview(gridController.view)  

gridController.view.translatesAutoresizingMaskIntoConstraints = false

gridController.view.leftAnchor.constraint(equalTo: view.safeLeftAnchor).isActive = true
gridController.view.rightAnchor.constraint(equalTo: view.safeRightAnchor).isActive = true
gridController.view.topAnchor.constraint(equalTo: view.safeTopAnchor).isActive = true
gridController.view.bottomAnchor.constraint(equalTo: view.safeBottomAnchor).isActive = true
```

### GiphyGridController: GPHContent 

The `GiphyGridController` comes with a new class `GPHContent`. A `GPHContent` describes a content request to the Giphy API. 

Create content objects with `GPHContent` class methods, like so: 

#### Trending 
```
let trendingGIFs = GPHContent.trending(mediaType: .gif) 
let trendingStickers = GPHContent.trending(mediaType: .sticker) 
let trendingText = GPHContent.trending(mediaType: .text)
```

#### Emoji 
```
let emoji = GPHContent.emoji
```

#### Search 
``` 
let search = GPHContent.search(withQuery: "Hello", mediaType: .gif, language: .english)
```

#### Recents

Show GIFs that the user has previously picked. 
``` 
let recentlyPicked = GPHContent.recents 
```

Only show a "recents" tab if there are any recents. Get the number of recents via: 
``` 
let numberOfRecents = GPHRecents.count 
```

Optionally, we also provide the option to clear the set of recents: 

```
GPHRecents.clear() 
```

### Updating the content

Set the grid controller's `content` property and call update: 
```
gridController.content = GPHContent.search(withQuery: "Sup", mediaType: .text, language: .english)
gridController.update()


```
### GiphyGridController: GPHGridDelegate

Similar to the `GiphyDelegate`, the `GPHGridDelegate` is the mechanism for responding to gif selection events in the grid. 

Conform to the `GPHGridDelegate` and set the delegate. 

```
gridController.delegate = self
```

```
extension ViewController: GPHGridDelegate {
    func contentDidUpdate(resultCount: Int) {
        print("content did update")
    }
    
    func didSelectMedia(media: GPHMedia, cell: UICollectionViewCell) {
        print("did select media")
    } 
}
``` 
