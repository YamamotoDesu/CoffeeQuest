# CoffeeQuest
<img src="https://user-images.githubusercontent.com/47273077/129432462-12add16c-d8d3-4210-ae2c-4b095150d56c.png" width="300" height="600">

## Run app
### 1. Refister(※Use Safari) And Get yelp Api Key
https://www.yelp.com/developers/v3/manage_app

### 2. Set api key  
https://github.com/YamamotoDesu/CoffeeQuest/blob/main/CoffeeQuest/Resources/APIKeys.swift

### 3. Delete Podfile.lock  
```
rm Podfile.lock 
```

### 4. Update cocoapods
```
pod deintegrate && pod install
```

### 5. Set Location on iOS Simulator  
Under the Debug menu in the Simulator, Choose "Custom Location".  
Set latitude and longitude.  
location: Shinjuku  
longitude: 139.710007  
latitude: 35.685001  

## Code Example  
https://github.com/YamamotoDesu/CoffeeQuest/blob/main/CoffeeQuest/ViewModels/BusinessMapViewModel.swift  
```swift  
import UIKit
import MapKit

public class BusinessMapViewModel: NSObject {
  
  // MARK: - Properties
  public let coordinate: CLLocationCoordinate2D
  public let name: String
  public let rating: Double
  
  public let image: UIImage
  public let ratingDescription: String
  
  // MARK: - Object Lifecycle
  public init(coordinate: CLLocationCoordinate2D,
              name: String,
              rating: Double,
              image: UIImage) {
    self.coordinate = coordinate
    self.name = name
    self.rating = rating
    self.image = image
    self.ratingDescription = "\(rating) stars"
  }
}

// MARK: - MKAnnotation
extension BusinessMapViewModel: MKAnnotation {
  
  public var title: String? {
    return name
  }
  
  public var subtitle: String? {
    return ratingDescription
  }
}

```  
