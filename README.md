# CoffeeQuest
<img src="https://user-images.githubusercontent.com/47273077/129431955-49da704a-e1f8-440a-9442-8422c1fd24a6.png" width="300" height="600">

## Run app(※Use Safari)  
### 1. Get yelp Api Key
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
