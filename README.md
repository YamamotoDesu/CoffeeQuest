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
### **[View Model](https://github.com/YamamotoDesu/MVVMPattern)**  
**[BusinessMapViewModel](https://github.com/YamamotoDesu/CoffeeQuest/blob/main/CoffeeQuest/ViewModels/BusinessMapViewModel.swift)**    

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
  
### **[Factory](https://github.com/YamamotoDesu/FactoryPattern)**  
**[AnnotationFactory](https://github.com/YamamotoDesu/CoffeeQuest/blob/main/CoffeeQuest/Factory/AnnotationFactory.swift)**  
```swift  
import MapKit
import YelpAPI

public class AnnotationFactory {
  public func createBusinessMapViewModel(for business: Business) -> BusinessMapViewModel {
    let coordinate = business.location
    let name = business.name
    let rating = business.rating
    let image: UIImage
    switch rating {
    case 0.0..<3.0: image = UIImage(named: "terrible")!
    case 3.0..<3.5:  image = UIImage(named: "bad")!
    case 3.5..<4.0: image = UIImage(named: "meh")!
    case 4.0..<4.75: image = UIImage(named: "good")!
    case 4.75...5.0: image = UIImage(named: "great")!
    default: image = UIImage(named: "bad")!
    }
    return BusinessMapViewModel(coordinate: coordinate,
                                name: name,
                                rating: rating,
                                image: image)
  }
}

```  

### Protocol  
**[BusinessSearchClient](https://github.com/YamamotoDesu/CoffeeQuest/blob/main/CoffeeQuest/Protocols/BusinessSearchClient.swift)**  
```swift  
public protocol BusinessSearchClient {
  func search(with coordinate: CLLocationCoordinate2D,
              term: String,
              limit: UInt,
              offset: UInt,
              success: @escaping (([Business]) -> Void),
              failure: @escaping ((Error?) -> Void))
}
```  

### Model  
**[Business](https://github.com/YamamotoDesu/CoffeeQuest/blob/main/CoffeeQuest/Models/Business.swift)**  
```swift  
import MapKit

public struct Business {
  var name: String
  var rating: Double
  var location: CLLocationCoordinate2D
}
```  

### **[Adapter](https://github.com/YamamotoDesu/AdapterPattern)**  
**[YLPClient+2BBusinessSerachClient](https://github.com/YamamotoDesu/CoffeeQuest/blob/main/CoffeeQuest/Adapters/YLPClient%2BBusinessSerachClient.swift)**    
```swift  
import MapKit
import YelpAPI

extension YLPClient: BusinessSearchClient {
  public func search(with coordinate: CLLocationCoordinate2D,
                     term: String,
                     limit: UInt,
                     offset: UInt,
                     success: @escaping (([Business]) -> Void),
                     failure: @escaping ((Error?) -> Void)) {
    let yelpCoordinate = YLPCoordinate(latitude: coordinate.latitude,
                                       longitude: coordinate.longitude)
    search(with: yelpCoordinate,
           term: term,
           limit: limit,
           offset: offset,
           sort: .bestMatched) { (searchResult, error) in
            guard let searchResult = searchResult, error == nil else {
              failure(error)
              return
            }
            let businesses = searchResult.businesses.adaptToBusinesses()
            success(businesses)
    }
  }
}

extension Array where Element: YLPBusiness {
  func adaptToBusinesses() -> [Business] {
    return compactMap { yelpBusiness in
      guard let yelpCoordinate = yelpBusiness.location.coordinate else {
        return nil
      }
      let coordinate = CLLocationCoordinate2D(latitude: yelpCoordinate.latitude,
                                              longitude: yelpCoordinate.longitude)
      return Business(name: yelpBusiness.name,
                      rating: yelpBusiness.rating,
                      location: coordinate)
    }
  }
}

```  

### Controller  
**[ViewController](https://github.com/YamamotoDesu/CoffeeQuest/blob/main/CoffeeQuest/Controllers/ViewController.swift)**   
```swift  
import MapKit
import YelpAPI

public class ViewController: UIViewController {
  
  // MARK: - Properties
  public let annotationFactory = AnnotationFactory()
  private var businesses: [Business] = []
  public var client: BusinessSearchClient = YLPClient(apiKey: YelpAPIKey)
  private let locationManager = CLLocationManager()
  
  // MARK: - Outlets
  @IBOutlet public var mapView: MKMapView! {
    didSet {
      mapView.showsUserLocation = true
    }
  }
  
  // MARK: - View Lifecycle
  public override func viewDidLoad() {
    super.viewDidLoad()
    DispatchQueue.main.async { [weak self] in
      self?.locationManager.requestWhenInUseAuthorization()
    }
  }
  
  // MARK: - Actions
  @IBAction func businessFilterToggleChanged(_ sender: UISwitch) {
    
  }
}

// MARK: - MKMapViewDelegate
extension ViewController: MKMapViewDelegate {
  
  public func mapView(_ mapView: MKMapView, didUpdate userLocation: MKUserLocation) {
    centerMap(on: userLocation.coordinate)
  }
  
  private func centerMap(on coordinate: CLLocationCoordinate2D) {
    let regionRadius: CLLocationDistance = 3000
    let coordinateRegion = MKCoordinateRegion(center: coordinate,
                                              latitudinalMeters: regionRadius,
                                              longitudinalMeters: regionRadius)
    mapView.setRegion(coordinateRegion, animated: true)
    searchForBusinesses()
  }
  
  private func searchForBusinesses() {
    client.search(with: mapView.userLocation.coordinate,
                  term: "coffee",
                  limit: 35,
                  offset: 0,
                  success: { [weak self] (businesses) in
                    guard let self = self else { return }
                    self.businesses = businesses
                    DispatchQueue.main.async {
                      self.addAnnotations()
                    }
    }, failure: { error in
      print("Search failed: \(String(describing: error))")
    })
  }
  
  private func addAnnotations() {
    for business in businesses {
      let viewModel = annotationFactory.createBusinessMapViewModel(for: business)
      mapView.addAnnotation(viewModel)
    }
  }
  
  public func mapView(_ mapView: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
    guard let viewModel = annotation as? BusinessMapViewModel else { return nil }
    let identifier = "business"
    let annotationView: MKAnnotationView
    if let existingView = mapView.dequeueReusableAnnotationView(withIdentifier: identifier) {
      annotationView = existingView
    } else {
      annotationView = MKAnnotationView(annotation: viewModel, reuseIdentifier: identifier)
    }
    annotationView.image = viewModel.image
    annotationView.canShowCallout = true
    return annotationView
  }
}

```  
