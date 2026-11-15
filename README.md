# HiTechCloud eKYC iOS SDK

> **English** | [Tiếng Việt](README.vi.md)

---

## Features

- Camera Preview
- Liveness
- Front and back recognition of 4 types (9-digit People's Identity Card, 12-digit People's Identity Card, Citizen Identity Card, Chip-embedded Citizen Identity Card, Electronic Citizen Identity Card, VNeID QR)
- Automatic detection and capture
- Image alignment

## Integrating the SDK

### Requirements

- iOS 12.1 or later
- Swift 5.0 or later

### Add the bundled libraries

Add the following libraries to the project:

- GoogleMLKit/FaceDetection (version 2.1.0 or later)
- TensorFlowLiteSwift (version 2.5.0 or later)
- OpenCV2 (version 4.3.0 or later)

You can manually add the bundled *.framework library files to the project, or use CocoaPods:

```
- pod 'GoogleMLKit/FaceDetection', '~> 2.1.0'
- pod 'TensorFlowLiteSwift', '~> 2.5.0'
- pod 'OpenCV2', '~> 4.3.0'
```

### Add the SDK

(Project -> Target -> General)

- Add the HITECHCLOUD_EKYC.framework file and set it to Embed & Sign

### Usage

- Import HITECHCLOUD_EKYC into the project
- Import AVFoundation into the project
- Request the Privacy - Camera Usage Description permission in Info.plist
- Conform to HITECHCLOUDEKYCCameraDelegate, HITECHCLOUDEKYCIdCardDetectorDelegate, HITECHCLOUDEKYCFaceDetectorDelegate
- Initialize the HITECHCLOUDEKYC view and configure its properties

#### Face detection

```
import UIKit
import HITECHCLOUD_EKYC
import AVFoundation
```

```
class FaceDetectorVC: UIViewController {

  @IBOutlet weak var cameraView: UIView!
  @IBOutlet weak var correctView: UIImageView!

  var camera: HITECHCLOUDEKYC?

  override func viewDidLoad() {
    super.viewDidLoad()
    camera = HITECHCLOUDEKYC.init(frameCameraPreview: self.cameraView.bounds, frameCorrect: self.correctView.frame)
    if let camera = self.camera {
      self.cameraView.addSubview(camera.cameraPreview)
    }
    self.cameraView.bringSubviewToFront(self.correctView)
    self.cameraView.bringSubviewToFront(self.imageCropView)

    // Set Delegate
    camera?.cameraDelegate = self
    camera?.faceDetectorDelegate = self

    //Set Type
    camera?.isType = .face

    //Show debug view
    camera?.showDebug = true

    //Position
    camera?.cameraPosition = .front

    // Start camera session
    camera?.startSession()
  }

  override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    // Restart session - restart the camera if it was previously paused
    self.camera?.restartSession()
  }

  override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    // Pause camera session - pause the camera if needed
    self.camera?.stopSession()
  }
}
```

Data is processed and results are returned through the following methods:

```
extension FaceDetectorVC: HITECHCLOUDEKYCCameraDelegate {

  /// User denied camera permission
  func cameraPermissionCameraDenied(status: AVAuthorizationStatus) {
  }

  // Camera started successfully
  func cameraDidStart() {
  }

  /// Camera session failed to start
  func cameraDidFail(error: LocalizedError) {
  }

  /// Camera position changed
  func cameraChangePosition(newPosision: AVCaptureDevice.Position) {
  }
}
```

```
extension FaceDetectorVC: HITECHCLOUDEKYCFaceDetectorDelegate {
  /// Result: valid reports whether a face was detected and
  /// the conditions (looking straight ahead, no head tilt) are met
  func faceDetectionResult(valid: Bool, image: UIImage?) {
    // This method is still called every time a face is received
    // Set a flag when an image is received to stop running this block as needed
    DispatchQueue.main.async {
      self.correctView.image = self.correctView.image?.withRenderingMode(.alwaysTemplate)
      self.correctView.tintColor = valid ? UIColor.green : UIColor.red
    }

    if let imageResult = image, valid {
      // Image received
      // flag
    }
  }
}
```

#### ID Card detection

```
import UIKit
import HITECHCLOUD_EKYC
import AVFoundation
class IdCardDetectorVC: UIViewController {

  @IBOutlet weak var cameraView: UIView!
  @IBOutlet weak var correctView: UIImageView!

  var camera: HITECHCLOUDEKYC?

  override func viewDidLoad() {
    super.viewDidLoad()
    camera = HITECHCLOUDEKYC.init(frameCameraPreview: self.cameraView.bounds, frameCorrect: self.correctView.frame)
    if let camera = self.camera {
      self.cameraView.addSubview(camera.cameraPreview)
    }

    self.cameraView.bringSubviewToFront(self.correctView)

    // Set Delegate
    camera?.cameraDelegate = self
    camera?.idCardDetectorDelegate = self

    // Set Type
    camera?.isType = .idCard

    // Show debug view
    camera?.showDebug = true

    // Position
    camera?.cameraPosition = .back

    // Start camera session
    camera?.startSession()
  }

  override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    // Restart session - restart the camera if it was previously paused
    self.camera?.restartSession()
  }

  override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    // Pause camera session - pause the camera if needed
    self.camera?.stopSession()
  }
}
```

Data is processed and results are returned through the following methods:

```
extension IdCardDetectorVC: HITECHCLOUDEKYCIdCardDetectorDelegate {
  func idCardDetectionResult(valid: Bool, result: IdCardResult?) {
    // This method is still called every time an ID card is received
    // Set a flag when an image is received to stop running this block as needed
    DispatchQueue.main.async {
      self.correctView.image = self.correctView.image?.withRenderingMode(.alwaysTemplate)
      self.correctView.tintColor = valid ? UIColor.green : UIColor.red
    }
    if let result = result, valid {
      let type: IdCardType = result.type
      let image: UIImage = result.image
      // Data received
      //flag
    }
  }
}
```
